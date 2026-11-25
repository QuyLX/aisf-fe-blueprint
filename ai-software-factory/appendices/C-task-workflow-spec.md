# Appendix C – Task Workflow Specification

1. Every workflow spec (playbook) that an agent executes MUST follow the format in this appendix and MUST validate against the `TaskWorkflowSpec` schema (Appendix K).
2. The workflow spec contract is defined in §6.7.1; when this appendix and §6.7.1 differ, §6.7.1 prevails.

## C.1. Structure

```yaml
apiVersion: aisf/4.0
kind: TaskWorkflowSpec
metadata:
  id: <workflow-id>
  version: <semver>
  owner: <team>
  status: DRAFT | PROPOSED | ACTIVE | NEEDS_REVALIDATION | SUPERSEDED | DEPRECATED | ARCHIVED   # §5.8
spec:
  taskClasses: []            # task classes served by this workflow
  riskCeiling: Low | Medium | High
  trigger: []                # events or requests that start the workflow
  inputs: { required: [], optional: [] }
  preconditions: []
  subStates: {}              # parent task state → [sub-states]; first entry is the initial sub-state
  checkpoints: []            # review checkpoints (§6.12): id, kind, when, after step
  steps: []                  # see C.2
  handoffs: []               # see §6.13
  budgets: {}                # ≤ policy ceilings (§6.4)
  recoveryDefaults: {}       # applied to steps that do not override them (§6.14)
  completion: { requiredOutputs: [] }
  metrics: []                # §13.7
```

## C.2. Step attributes

Each step MUST declare the fields marked "Yes" in the second column.

| Field | Required | Meaning |
|---|---|---|
| `id` | Yes | Unique identifier within the workflow |
| `skeletonStep` | Yes | Corresponding step in the §6.6 skeleton (1–16) |
| `kind` | Yes | `mandatory` \| `conditional` \| `optional` (§6.7.3) |
| `when` | For `conditional` | Condition on declared inputs; evaluated by policy |
| `subState` | No | Sub-state the task is in while the step runs |
| `role` | Yes | Executing role (§6.3), or `policy` / `human` |
| `skills`, `tools` | No | Skills and tools allowed |
| `inputs`, `outputs` | Yes | Input and output artifacts; this step's output is the next step's input |
| `verification` | Yes | Check ID or completion criteria of the step |
| `onFailure` | Yes | Behavior when the step fails: target state, step to return to, or escalation |
| `timeout`, `retry`, `idempotent`, `sideEffect`, `rollback`, `compensation` | Yes (may be inherited from `recoveryDefaults`) | Recovery attributes (§6.14) |

## C.3. Reference spec: Bug Fix

**Example.** Full spec of the Bug Fix workflow; the other workflows are outlined in [Appendix D](D-catalog-workflow.md).

```yaml
apiVersion: aisf/4.0
kind: TaskWorkflowSpec
metadata:
  id: bug-fix
  version: 1.0.0
  owner: engineering-enablement
  status: ACTIVE
spec:
  taskClasses: [bug-fix]
  riskCeiling: High
  trigger:
    - issue labeled type:bug with a factory task command
    - forward-fix task created by incident-response
    - production signal above declared threshold, after de-duplication (§12.4); requester = automation owner
  inputs:
    required: [goal, why, repository, reproductionReference]   # issue, log, report or failing scenario
    optional: [suspectedArea, relatedIncident]
  preconditions:
    - repository has golden commands and a resolved verification profile
    - no unresolved knowledge conflict blocks the affected area (§4.7)

  subStates:
    CONTEXT_READY: [REPRODUCING, REPRODUCED]
    PLANNED:       [ROOT_CAUSE_IDENTIFIED]
    EXECUTING:     [FIXING]

  checkpoints:
    - { id: requirement-review,       kind: mandatory,   after: classify }   # approver depends on the provisional tier
    - { id: design-review,            kind: conditional, after: diagnose,
        when: "fix touches public-api OR module-boundary OR schema" }
    - { id: plan-review,              kind: conditional, after: plan,
        when: "tier == High OR reversibility == LOW" }
    - { id: independent-verification, kind: mandatory,   after: verify }
    - { id: implementation-review,    kind: mandatory,   after: review }
    - { id: release-readiness,        kind: conditional, after: approve-merge,
        when: "deployable AND (tier == High OR reversibility == LOW)" }   # Medium: confirmed by the implementation-review approver

  recoveryDefaults:
    timeout: 30m
    retry: { max: 2, backoff: exponential, on: [ERROR, timeout] }
    idempotent: true
    sideEffect: none

  steps:
    - id: intake
      skeletonStep: 1
      kind: mandatory
      role: intake
      inputs: [goal, why, reproductionReference]
      outputs: [change-brief]
      verification: change-brief fields complete; every AC has an oracle
      onFailure: NEEDS_CLARIFICATION

    - id: deduplicate
      skeletonStep: 1
      kind: conditional
      when: "trigger is a production signal"
      role: policy
      inputs: [signal-fingerprint]
      outputs: [dedup-decision]                  # new task | attached to open task/incident
      verification: no open task or incident has the same fingerprint
      onFailure: { attachToExisting: CANCELLED }

    - id: classify
      skeletonStep: 2
      kind: mandatory
      role: policy
      inputs: [change-brief, changed-paths-estimate]
      outputs: [policy-decision]                 # provisional tier, reversibility, autonomy, slots; official tier recomputed from the diff at every VERIFYING
      verification: policy decision recorded
      onFailure: escalate

    - id: resolve-context
      skeletonStep: 3
      kind: mandatory
      role: investigator
      skills: [context-resolve]
      inputs: [change-brief]
      outputs: [context-pack, basis]             # basis: ACTIVE artifacts id@version (§6.4)
      verification: every artifact in basis is ACTIVE or handled per §6.5
      onFailure: NEEDS_CLARIFICATION

    - id: reproduce
      skeletonStep: 10                           # independent oracle written before the fix
      kind: mandatory
      subState: REPRODUCING
      role: verifier                             # separate context from implementer (§7.7)
      skills: [repro]
      tools: [test-runner, browser-driver]
      inputs: [change-brief, context-pack]
      outputs: [reproduction-test, reproduction-evidence]
      verification: reproduction-test FAILS on base SHA for the reported reason
      onFailure:
        notReproducible: { addStep: investigate-further, then: escalate-to-requester }
      timeout: 60m

    - id: diagnose
      skeletonStep: 4
      kind: mandatory
      subState: ROOT_CAUSE_IDENTIFIED
      role: investigator
      inputs: [reproduction-evidence, context-pack]
      outputs: [root-cause, supporting-evidence, knowledge-conflicts]
      verification: root cause explains the reproduction; conflicts recorded (§4.7)
      onFailure:
        rootCauseUnconfirmed: escalate
        implementationDiffersFromSpec: { record: knowledge-conflict, then: continue-as-bug }
        fixNeedsDataMigration: { switchWorkflow: database-migration, decision: replan }

    - id: plan
      skeletonStep: 5
      kind: mandatory
      role: planner
      inputs: [root-cause, policy-decision]
      outputs: [plan]                            # lightweight for tier Low
      verification: plan covers every AC and states how to revert
      onFailure: escalate

    - id: prepare-workspace
      skeletonStep: 7
      kind: mandatory
      role: implementer
      outputs: [workspace]                       # worktree pinned to base SHA
      verification: base SHA recorded in task record
      onFailure: escalate

    - id: implement
      skeletonStep: 8
      kind: mandatory
      subState: FIXING
      role: implementer
      inputs: [plan, reproduction-test]          # reproduction-test is read-only for the implementer
      outputs: [code-change, additional-tests]
      verification: reproduction-test now passes locally
      onFailure: { stayIn: EXECUTING, onTimeout: escalate }   # local iteration; REPAIRING exists only after VERIFYING
      sideEffect: reversible                     # branch only
      rollback: discard branch

    - id: self-check
      skeletonStep: 9
      kind: mandatory
      role: implementer
      outputs: [fast-check-results]
      verification: format, lint, type, focused tests, build
      onFailure: { stayIn: EXECUTING }

    - id: verify
      skeletonStep: 10
      kind: mandatory
      role: verifier
      inputs: [code-change, reproduction-test, resolved-check-plan]
      outputs: [verification-run]                # via verification adapters (§7.16)
      verification: all required checks PASS or carry a valid disposition
      onFailure: REPAIRING                       # consumes verifyRepairIterations

    - id: stress
      skeletonStep: 10
      kind: conditional
      when: "changed paths tagged timing-sensitive"
      role: verifier
      outputs: [stress-run]
      verification: affected tests pass N repeated runs (§7.14)
      onFailure: REPAIRING
      timeout: 60m

    - id: review
      skeletonStep: 11
      kind: mandatory
      role: reviewer
      inputs: [diff, verification-run, change-brief]
      outputs: [findings-and-dispositions]
      verification: every posted finding has a disposition (§7.9)
      onFailure: REPAIRING                       # consumes reviewRounds

    - id: knowledge-reconcile
      skeletonStep: 12
      kind: mandatory
      role: improvement
      inputs: [diff, basis]
      outputs: [knowledge-impact, reconciliation-decisions]
      verification: check knowledge.impact passes (§5.9)
      onFailure: escalate

    - id: package
      skeletonStep: 13
      kind: mandatory
      role: implementer
      outputs: [pr-description, evidence-manifest]
      verification: PR template sections complete (Appendix G)
      onFailure: { retryStep: package }

    - id: approve-merge
      skeletonStep: 14
      kind: mandatory
      role: human-or-policy                      # policy merges only when every A0 condition holds (§9.5)
      inputs: [pr-description, evidence-manifest]
      outputs: [approval-slots, merge-result]
      verification: slots filled for current SHA; merge queue passes
      onFailure: REPAIRING                       # consumes mergeQueueAttempts on attributable failures
      sideEffect: reversible
      rollback: revert commit on main

    - id: observe
      skeletonStep: 15
      kind: conditional
      when: deployable
      role: change-shepherd
      outputs: [rollout-health]
      verification: observation window passes thresholds
      onFailure: { PAUSED: flag-off, ROLLED_BACK: human-authorized }

    - id: learn
      skeletonStep: 16
      kind: mandatory
      role: improvement
      outputs: [workflow-metrics, gardening-findings]
      verification: metrics recorded (§13.7)
      onFailure: record-and-continue

  handoffs:
    - from: verifier/reproduce
      to: implementer/implement
      validIn: [CONTEXT_READY, PLANNED]
      requiredArtifacts: [reproduction-test, reproduction-evidence]
    - from: implementer/self-check
      to: verifier/verify
      validIn: [VERIFYING]
      requiredArtifacts: [code-change, head-sha]
      excludeContext: [implementer-notes, implementer-plan-rationale]   # independence (§7.7)
    - from: implementer/package
      to: human/approve-merge
      validIn: [AWAITING_MERGE_APPROVAL]
      requiredArtifacts: [pr-description, evidence-manifest, knowledge-impact]
  maxHandoffRejections: 2

  budgets:                    # ceilings; tasks may only lower them (§6.4)
    verifyRepairIterations: 3
    reviewRounds: 3
    mergeQueueAttempts: 2
    cost: { limit: 60, unit: usd }

  completion:
    requiredOutputs:
      - reproduction-evidence
      - reproduction-test
      - verification-run
      - findings-and-dispositions
      - reconciliation-decisions
      - change-summary

  metrics: [success, failureByStep, retries, approvalWaitByCheckpoint, escalationsByCause, handoffRejections, leadTimeBySubState, cost]
```

## C.4. Minimal version for the pilot

**Example.** Pilot mode (§20.8, roadmap in §15.0) may use the following subset of C.3. The omitted parts are targets for later phases, not permanent options.

| Kept in the pilot | Omitted in the pilot |
|---|---|
| Steps `intake`, `classify`, `resolve-context`, `reproduce`, `diagnose`, `plan` (lightweight), `implement`, `verify`, `review`, `knowledge-reconcile`, `package`, `approve-merge`; `learn` in lightweight form: the "Review findings" and "Knowledge impact" sections of the PR (Appendix G) plus metrics collected from events | `stress`, `observe` |
| Checkpoints `requirement-review`, `independent-verification`, `implementation-review`; `plan-review` when tier is High | Separate `release-readiness` variant for tier Medium |
| Reproduction test written by the verifier in a separate context, read-only for the implementer | Handoff records as artifacts (independence is still kept: the verifier does not receive the implementer's notes) |
| Loop budget | Sub-states (inferred from events or skipped); recovery records as artifacts (`ERROR` still does not consume budget) |
| Records inferred from labels, PR events and CI runs (§6.5) | Manually filled records |
| Tier from a deterministic path → tier mapping table (`riskMap` in `factory.yaml`); undeclared paths are High | Automated tier classifier |

## C.5. The `generic` workflow

**The `generic` workflow** is the core workflow owned by the Standard, used for every task whose type has no `ACTIVE` workflow spec yet (§6.7.1 rule 1). It has the same version as the Standard (for example `generic@4.0.0`) and is always `ACTIVE`.

1. The `generic` workflow MUST have exactly one step for each step of the §6.6 skeleton, with step kinds as follows: steps 1–3, 5, 7–9, 11–14 and 16 are `mandatory` (step 5 MAY be a lightweight plan at tier Low); steps 4, 6, 10 and 15 are `conditional`, with the condition being the negation of the omission condition in the last column of §6.6 (step 4: tier is not Low; step 6: the §6.12 condition holds; step 10: there is a `critical` acceptance criterion or tier is Medium/High; step 15: the change is deployed).
2. The checkpoints of `generic` MUST be `requirement-review`, `implementation-review` and `independent-verification` (when the §6.12 condition holds) as `mandatory`, plus every checkpoint required by the general conditions of §6.12 (`specification-review`, `design-review`, `plan-review`, `release-readiness`) as `conditional`.
3. `generic` MUST NOT define sub-states.
4. The loop budgets and cost ceiling of `generic` MUST equal the policy ceilings (§6.4); a task MAY lower them.
5. The default recovery attributes MUST be: `timeout` per policy, `retry` only for `ERROR` and timeout, `idempotent: false`, `sideEffect` as declared by the step; a step that does not declare `sideEffect` MUST be treated as `irreversible` (§6.14).
6. The required outputs on completion MUST include: a Change Brief that has passed `requirement-review`, `basis`, the `VerificationRun` of the final commit, findings and dispositions, reconciliation decisions (§5.10) when artifacts are affected, and the PR description.
7. The `generic` task class MUST NOT exceed trust T0 (§9.4); `allowedAutonomy` is no higher than `implement-and-request-review`.
8. A profile MAY add steps or checkpoints to `generic` per §20.3 and MUST NOT remove any part of items 1–7.
