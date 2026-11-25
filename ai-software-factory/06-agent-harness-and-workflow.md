# 6. Agent harness and operating model

## 6.1. The model: Agent = Model + Harness

An **Agent** is a model plus a harness. The **Model** provides reasoning capability. The **Harness** provides:

- Task contract and state.
- Context discovery and memory boundaries.
- Tools and permission enforcement.
- Workspace isolation.
- Task Workflow Specification (playbook) and skills.
- Pre-action guidance.
- Post-action sensors and verification.
- Human approval gate.
- Traces, evidence and cost accounting.
- Recovery behavior, timeouts and circuit breakers.

1. The Factory SHOULD keep the model/provider replaceable.
2. The Factory SHOULD NOT hard-code provider-specific concepts into the Factory's canonical data model.
3. The Factory MAY embed open, reusable agent harnesses instead of rebuilding them.

**Rationale.** The model provides reasoning capability; everything else that lets an agent work safely and repeatably (contract, context, tools, permissions, workspace, workflow, sensors, gates, traces, recovery) lives in the harness. The durable investment of the Factory is therefore the harness, not the choice of model. Models and providers change quickly, while task contracts, evidence and policy need to outlive them. For open, reusable agent harnesses, see [R02].

## 6.2. Separating Role, Skill, Playbook and Tool

| Concept | Meaning |
|---|---|
| Role | Responsibilities and permission profile |
| Skill | Reusable expertise or procedure |
| Playbook | An ordered workflow for a task type, written per the Task Workflow Specification (§6.7) |
| Tool | An executable capability |

1. A role does not require its own long-running agent process; the same agent runtime MAY take on different roles with different tool and context scopes.

**Rationale.** Separating the four concepts makes it possible to grant permissions by role, reuse skills across playbooks, and replace tools without rewriting workflows.

**Example.**

| Concept | Example |
|---|---|
| Role | Implementer, verifier, security reviewer |
| Skill | Reproducing a bug, checking the dependency graph, verifying a flow in a browser |
| Playbook | Bug Fix, Feature Development, Database Migration |
| Tool | Git, shell, browser driver, CI API, catalog queries |

## 6.3. Initial agent roles

When activated, each role below MUST have the responsibilities and default access given in the following table.

| Role | Responsibilities | Default access |
|---|---|---|
| Intake / Requirement Analyst | Clarify the outcome, scope, assumptions and acceptance criteria | Read; draft change/spec |
| Context / Investigator | Explore code, documentation, metadata, logs and dependencies | Read-only by default |
| Architect / Planner | Analyze impact, propose the design and implementation plan | Read-only; create proposals |
| Implementer | Change code and tests | Write in an isolated workspace/branch |
| Verifier | Run approved checks and collect evidence | Read source; execute allowlisted commands |
| Reviewer | Evaluate the diff and evidence independently | Read-only |
| Security Reviewer | Analyze security and data exposure risks | Read-only; security tools with scope-limited permissions |
| Release Operator | Prepare the release and monitor the rollout | Scope-limited release permissions; privileged actions MUST have approval |
| Improvement Agent | Propose improvements to rules, skills, documentation, evals or runbooks; propose reconciliation decisions for affected knowledge artifacts (§5.10) | Create proposals/PRs; MUST NOT self-approve changes to its own policy; MUST NOT move an artifact's state other than to `NEEDS_REVALIDATION` |
| Bug Hunter | Find defects in a diff, in parallel with other hunters; every finding cites file and line | Read-only |
| Finding Verifier | Independently confirm or refute each candidate finding before it is posted | Read-only |
| Change Shepherd | Follow an approved change through the whole rollout: flag, success/failure signals, monitoring, health reporting, pause/rollback recommendations (§12.2.1) | Read telemetry; flag/rollout actions only through approved, audited tools |

**Role activation**

1. The roles needed for a task MUST be decided by the playbook and the risk policy; the Factory MUST NOT activate every role for every task.

**Requester accountability**

2. Every agent task MUST have a specific person who directed it (the requester).
3. The requester MUST be accountable for the outcome and MUST sign off the PR's Testing / Risk / How-to-revert statements.
4. Whether that sign-off also counts as an approval MUST be determined solely by the separation of duties (SoD) table (§9.10): yes for tier Low, no for tier Medium, High or a control file change.
5. The principle "you land it, you own it" MUST apply to agent-written changes exactly as to human-written changes.

**Specialized reviewers**

6. The value of domain-specialized review agents MUST be demonstrated by evaluation; the Factory MUST NOT assume by default that they are valuable.

**Rationale.** The Standard is deliberately stricter than the cited practice, where the requester counts as one of the required reviewers [R29]: here the requester's sign-off counts as an approval only when the separation of duties table in §9.10 allows it (tier Low). Domain-specialized review agents (for example, one agent per infrastructure area) create value only when each agent has a narrow focus and the right context [R28].

**Example.** A tier Low bug fix may need only an implementer, a verifier and a reviewer, while a tier High data migration additionally needs an architect, a security reviewer, a release operator and a Change Shepherd.

## 6.4. Task contract

The **Task contract** is the standardized contract of a task, per the `Task` schema (Appendix K). The `spec` fields:

| Field | Content |
|---|---|
| `metadata.requestedBy` | Requester: the authenticated trigger identity (§9.10), not free text |
| `goal`, `why` | The desired outcome; the problem or opportunity, linked to the issue |
| `taskType`, `workflow` | The task type; the `ACTIVE` Task Workflow Specification the task follows (§6.7), in the form `<workflow-id>@<version>` |
| `repositories` | Affected repositories |
| `scope.in`, `scope.out` | In and out of scope |
| `doneWhen` | Acceptance criteria: `id`, `statement`, `oracle`, `critical` (§6.4.1) |
| `assumptions`, `risksAndUnknowns` | Assumptions (with `status`), risks and unknowns |
| `howToVerify`, `howToRevert` | How to verify; how to revert |
| `expectedEvidence` | Expected evidence |
| `basis` | The knowledge artifacts (by `id@version`), contract versions and confirmed assumptions the task relies on; recorded at `CONTEXT_READY`/`PLANNED` |
| `policy.verificationProfile`, `policy.budgets` | Verification profile; loop budgets and `cost` ceiling |

**Contract**

1. Every Factory task MUST have a standardized Task contract; for human-written PRs, the Task contract MAY be represented by the PR template (item 6).
2. The Task contract SHOULD capture the user's intent and acceptance criteria.
3. The Task contract SHOULD NOT prescribe every implementation detail.
4. Important missing information MUST be raised explicitly.
5. The agent MUST NOT invent high-impact assumptions on its own.

**Human-written PRs**

6. For a work item performed by a person without an agent, the Task contract MAY be represented by the Intent, Risk, Testing and How to revert sections of the PR template (Appendix G); in that case the checkpoint record is derived from the platform's review and approval events, and the Factory MUST NOT require additional artifacts.

**Workflow and basis**

7. `workflow` MUST point only to the Task Workflow Specification the task follows (§6.7).
8. `basis` MUST list the knowledge artifacts (by `id@version`), contract versions and confirmed assumptions the task relies on; only confirmed assumptions belong to the basis. `basis` is the ground for detecting when a task is relying on knowledge that has changed (§6.5).

**Loop budget**

9. Policy MUST define the ceilings of the loop budgets; a playbook or task MAY only lower them and MUST NOT raise them.
10. The effective value MUST be `min(policy, playbook, task)`.
11. Budgets MUST be counted separately per the following table.

| Counter | Consumed by | Not consumed by |
|---|---|---|
| `verifyRepairIterations` | Required check failures caused by this change itself | Failures reproducible on main without this change |
| `reviewRounds` | Review/fix rounds (maximum 3, §7.11) | Re-review triggered only by a merge queue rebase |
| `mergeQueueAttempts` | Failures attributable to this change, **including semantic conflicts** with changes already on main: the PR must adapt itself | Failures not attributable to this change |

12. Exhausting any counter MUST move the task to `ESCALATED`.
13. An `ERROR` result (§7.5) MUST NOT consume budget.

**Cost ceiling**

14. Every task MUST have a cost ceiling (`cost`, in `usd` or `tokens`) for the AI and CI cost attributable to the task, under the same `min(policy, playbook, task)` rule.
15. The task runner MUST track cost while running.
16. On reaching the *suggested* 80% of the ceiling, the task runner MUST warn the requester.
17. On reaching the ceiling, the task MUST stop at the nearest checkpoint and move to `ESCALATED`.
18. Loosening the ceiling MUST be a recorded decision by the requester (tier Low) or by the task's approver (Medium/High).

**Merge queue accountability**

19. Merge queue accountability MUST be decided by an operational action, not by judgment: rerun the failing check on the latest main (a) without the change and (b) with the change, then apply the following table.

| (a) main without the change | (b) main with the change | Accountability |
|---|---|---|
| Fail | — | Not caused by this change: main is broken. Pause the queue and notify the owner of the change that broke it; do not consume budget |
| Pass | Fail | Caused by this change, including semantic conflicts with work just merged; consumes `mergeQueueAttempts` |
| Pass | Pass (failure not reproducible) | Flake candidate; handle per §7.14, do not consume budget. If it recurs, hand it to the check owner |

**Rationale.** The contract captures *what* must be true and *how to know* it is true, and does not prescribe how to implement it. A high-impact assumption invented by the agent is the hardest source of defects to detect, because every later step is correct relative to that wrong assumption.

The three loop budget counters are separated because they measure three different kinds of failure: the change breaking checks by itself, review that does not converge, and a change that cannot integrate with main. The `min(policy, playbook, task)` rule ensures that playbooks and tasks can only be stricter than policy. An `ERROR` result is an infrastructure failure of the check, not a failure of the change, so it does not consume budget.

The cost ceiling blocks the case of a task that iterates expensively while still staying within the round limits. In addition to the per-task ceiling, the Pilot Charter sets a cost ceiling for the whole pilot (Appendix H).

The merge queue accountability table turns the question "whose failure is this" into a repeatable check rerun, instead of a debate.

**Example.** A complete Task contract for a bug fix (standard format: `Task` schema, Appendix K):

```yaml
apiVersion: factory.example/v1
kind: Task
metadata:
  id: TASK-2026-00124
  requestedBy: user:developer   # authenticated trigger identity (§9.10), not free text
spec:
  goal: A timed-out worker job is retried exactly once more, then fails visibly
  why: issue CICD-4821
  taskType: bug-fix
  workflow: bug-fix@1.0.0          # ACTIVE Task Workflow Specification (§6.7)
  repositories: [pipeline-service]
  scope:
    in: [retry policy for worker timeouts]
    out:
      - public API behavior changes (need approval)
      - new dependencies (need dependency review)
  doneWhen:
    - id: AC-1
      statement: The timeout failure is reproduced on the base commit
      oracle: new-independent-test
      critical: true
    - id: AC-2
      statement: Exactly one retry after a worker timeout; no duplicate execution
      oracle: new-independent-test
      critical: true
    - id: AC-3
      statement: Service and contract tests pass
      oracle: existing-test
      critical: false
  assumptions: []
  risksAndUnknowns: []
  howToVerify: backend-service-v1 profile plus the AC-1/AC-2 tests in stress mode
  howToRevert: Revert commit; no data change
  expectedEvidence: [reproduction, regression-test, test-report, change-summary, knowledge-impact]
  basis:                           # what the task relies on (§6.5); captured at CONTEXT_READY/PLANNED
    artifacts:
      - SPEC-PIPE-RETRY@3          # knowledge artifacts, id@version (§5.8)
      - ADR-0012@1
      - contract:pipeline-api-v2@2.3.0
    assumptions: []                # only confirmed assumptions belong to the basis
  policy:
    verificationProfile: backend-service-v1
    budgets:                      # may only lower the policy ceilings
      verifyRepairIterations: 3
      reviewRounds: 3
      mergeQueueAttempts: 2
      cost: { limit: 40, unit: usd }   # AI + CI spend attributable to this task (§6.4)
```

### 6.4.1. Change Brief and the specification threshold

The **Change Brief** is the minimum set of fields of the Task contract (§6.4), not a separate artifact. The Change Brief consists of:

| Field | Content |
|---|---|
| `metadata.requestedBy` | The accountable person (requester) |
| `goal` | What must be true after the change, in user/system language |
| `why` | The problem or opportunity; link to the issue |
| `scope.in`, `scope.out` | In and out of scope (replacing free-text constraints) |
| `doneWhen` | Acceptance criteria, 1–5 testable statements; each criterion has `id`, `statement`, `oracle` (`existing-test` \| `new-independent-test` \| `contract` \| `runtime-check` \| `human-judgment`), `critical` |
| `assumptions` | Each assumption has a `statement` and a `status` (`confirmed` \| `unconfirmed`) |
| `risksAndUnknowns` | Risks and unknowns |
| `howToVerify` | Commands, scenarios or expected evidence |
| `howToRevert` | Flag, revert or data recovery path; list what is not reversible |

**Thresholds**

1. The Change Brief fields MUST be fully filled before `PLANNED`.
2. The Factory MUST determine the required artifact per the following table; a full specification is required only when a threshold is exceeded.

| Condition | Required artifact |
|---|---|
| Low risk, single module, reversible | Change Brief (the Intake agent MAY draft; the requester confirms) |
| New user-facing capability, public API/contract change, multi-repository change, data migration, or tier High | Full specification with reviewed acceptance criteria in `ACTIVE` state (`specification-review`, §6.12) |
| Unconfirmed assumption that affects correctness or risk | Stop at `NEEDS_CLARIFICATION` |

**Critical acceptance criterion**

3. Every acceptance criterion MUST have a `critical` field.
4. The approver of `requirement-review` (§6.12) MUST confirm the `critical` value.
5. The agent MUST NOT set `critical: false` on its own.
6. At tier Medium and High, every criterion MUST default to `critical: true` unless the approver marks otherwise with a reason.
7. At every tier, the criterion proving that the reported defect has been reproduced and has been fixed MUST always be `critical: true`.
8. Independent verification (§6.6 step 10, §7.7) MUST be performed for every `critical` criterion.

**Who writes the independent oracle**

9. For tier Low, the verifier role MUST derive scenarios from the Change Brief's acceptance criteria in a context separate from the implementer.
10. For tier Medium and High, critical acceptance scenarios MUST be confirmed by the requester or a code owner before implementation starts.
11. For tier Medium and High, the implementer MUST NOT modify confirmed acceptance scenarios without approval.

**Rationale.** Principle P01 needs a minimal artifact that is usable for ordinary tickets, which rarely contain reviewed acceptance criteria. The Change Brief meets that need without creating additional documents. A full specification is needed only when a threshold is exceeded, so that the cost of specification is proportionate to risk.

The `critical` field decides which criteria must have an independent oracle. If the agent were allowed to set `critical: false` itself, it could exempt itself from the hardest part of verification.

An oracle written by the implementer itself usually tests exactly what the implementer understood, even when that understanding is wrong. Separating the author of the scenarios from the implementer, and locking confirmed scenarios at tier Medium/High, is how oracle independence is maintained.

*The opposing view was considered:* some practitioners argue that "the best spec is the code" and skip the planning step [R06]. The Standard accepts this for exploration — a throwaway prototype may come first and inform the Change Brief — but production changes still need a clearly stated outcome and acceptance criteria, because verification needs an oracle.

**Example.** The Change Brief fields:

```yaml
kind: Task
metadata:
  requestedBy: accountable human          # the "requester"
spec:
  goal: What should be true after the change, in user/system terms
  why: Problem or opportunity; link to issue
  scope:                                  # replaces free-text constraints
    in: []
    out: []
  doneWhen:                               # acceptance criteria, 1–5 testable statements
    - id: AC-1
      statement: ...
      oracle: existing-test | new-independent-test | contract | runtime-check | human-judgment
      critical: true | false              # confirmed at requirement-review (§6.4.1)
  assumptions:
    - statement: ...
      status: unconfirmed                 # confirmed | unconfirmed
  risksAndUnknowns: []
  howToVerify: Command, scenario, or expected evidence
  howToRevert: Flag, revert, or data-recovery path; irreversible parts listed
```

## 6.5. Task lifecycle and state machine

The task runner MUST apply the following state machine for the `code` change channel.

```text
REQUESTED
   ↓
TRIAGED ──────────────→ REJECTED | NEEDS_CLARIFICATION ──→ (back to TRIAGED)
   ↓
CONTEXT_READY ─────────→ NEEDS_CLARIFICATION                (failed requirement-review, §4.7 conflict)
   ↓
PLANNED ───────────────→ NEEDS_CLARIFICATION | REJECTED     (§4.7 conflict | failed design-review)
   ↓
AWAITING_PLAN_APPROVAL ─→ REJECTED | PLANNED (revise) | NEEDS_CLARIFICATION   (only when policy requires)
   ↓
EXECUTING                                                   (draft PR opened here)
   ↓
VERIFYING ────fail────→ REPAIRING ──→ VERIFYING              (budget: verifyRepairIterations)
   ↓ pass
REVIEWING ──changes───→ REPAIRING ──→ VERIFYING              (budget: reviewRounds)
   ↓ all findings resolved
AWAITING_MERGE_APPROVAL                                      (per §9.10; bound to commit SHA)
   ↓
IN_MERGE_QUEUE ──fail─→ REPAIRING ──→ VERIFYING              (budget: mergeQueueAttempts)
   ↓ pass on latest main
MERGED ──not deployed (docs, unpublished SDK, internal tooling)──→ COMPLETED
   ↓ deployable                                               (after post-merge checks)
RELEASING (staged) ──→ PAUSED ──resume──→ state it came from (RELEASING | OBSERVING)
   │                     └──────────────→ ROLLED_BACK
   ├──────────────────→ ROLLED_BACK
   ↓
OBSERVING ─regression─→ PAUSED (flag-off by Change Shepherd) | ROLLED_BACK
   ↓ observation window passed
COMPLETED

ROLLED_BACK → REVERTED_ON_MAIN (revert PR merged, or deploy of the commit blocked)
            → INCIDENT_ANALYSIS → FORWARD_FIX (new task) | COMPLETED

Any non-terminal state → BLOCKED | ESCALATED (with reason and owner).
Any state before MERGED → CANCELLED (with reason and owner).
Any state before MERGED → PLANNED, on a basis-change decision replan or reapprove.
BLOCKED → the state it came from, once the blocker is resolved.
ESCALATED → the state it came from | PLANNED | CANCELLED, by a recorded human decision.
Terminal states: COMPLETED, REJECTED, CANCELLED, FORWARD_FIX.
A new commit after approval → VERIFYING; approval is invalidated unless the
commit was produced by a policy-approved commit mechanism (§9.8).
```

**Rule**

1. Every fix path MUST return to `VERIFYING`.
2. A task MUST NOT advance to `REVIEWING`, `AWAITING_MERGE_APPROVAL` or `MERGED` based on the evidence of an earlier commit.
3. Exceeding a loop budget MUST move the task to `ESCALATED` and MUST NOT start a new loop.
4. The full list of valid transitions is in the vocabulary (`taskState.transitions`). A transition not in the list is invalid, and the task runner MUST reject it.
5. Leaving `ESCALATED` MUST require a recorded decision by the accountable person: continue (MAY include a one-time budget loosening, which MUST be recorded explicitly as a loosening decision, not a reset), replan (`PLANNED`), or cancel.
6. A loop budget MUST NOT ever be reset.
7. A task that is canceled and then recreated with the same goal MUST link to the old task and inherit the budget already used.
8. A rollback MUST NOT be considered complete until the commit is reverted on main or deployment of that commit is blocked.
9. In pilot mode (§20.8), the Factory MAY represent the states before `EXECUTING` with labels on the issue and, from `EXECUTING` onward, with labels and CI check runs on the draft PR, without a separate task store (for example: Appendix H, I).
10. **Records are derived, not hand-written.** State transition records, checkpoints, approvals and handoffs SHOULD be derived automatically from platform events (labels, PR review and approval events, CI runs, structured comments).
11. The Factory MUST NOT require humans to fill in records merely to serve the Factory; fields that cannot be derived from events MUST be filled by the task runner or an agent, not by a reviewer.
12. This state machine is the only task-level lifecycle of the `code` change channel. A workflow MUST NOT define a competing lifecycle; a workflow MAY only refine it with sub-states (below).
13. A change channel that does not use PRs MUST apply the reduced lifecycle in §12.6, which is a mapping of the same state machine, not a second lifecycle: `VERIFYING` → `VERIFYING`; `AWAITING_APPROVAL` → `AWAITING_MERGE_APPROVAL`; `PUBLISHED` → `MERGED` and `RELEASING`; `OBSERVING`, `ROLLED_BACK`, `COMPLETED`, `BLOCKED` unchanged; the corresponding gate is `publish` (§7.4).
14. The `PAUSED` state MUST be used only for rollouts. Pausing the execution of a step (waiting for a human gate, time budget exhausted, degraded mode) MUST NOT change the task state; it MUST be recorded as a recovery event (§6.14), or the task MUST move to `BLOCKED` when waiting on an external dependency.

**State transition record**

15. Every state or sub-state transition MUST record:
    - The actor (the identity of the person or agent).
    - The time.
    - The state and sub-state before and after.
    - References to input/output artifacts.
    - The related policy decision and checkpoint, if any.
    - The current commit/workspace.
    - The failure or escalation reason.

**Sub-states per workflow**

A Task Workflow Specification (§6.7) MAY define sub-states to describe the progress specific to a task type within a task state.

1. Every sub-state MUST belong to exactly one parent state in the state machine above, and MUST be recorded as a `state` + `subState` pair (for example `EXECUTING` / `FIXING`).
2. Transitions between sub-states MUST occur only within the parent state. Leaving a parent state MUST occur only through the transitions of the task-level state machine; a sub-state MUST NOT create a shortcut past any state, gate or checkpoint.
3. When a task enters a parent state, the sub-state MUST be set to the initial sub-state that the workflow declares for that state, including when entering through the fix path `REPAIRING → VERIFYING`.
4. Policy, approval, loop budgets and conformance MUST reference only the parent state and MUST NOT reference sub-states. Sub-states are used for progress, checkpoint positioning and workflow metrics (§13.7).
5. Sub-state names MUST belong to the workflow's namespace and MUST be registered in the workflow spec; a profile MUST map its own sub-states to parent states per PC-2 (§20.2).

The Bug Fix and Database Migration workflows MUST use the following sub-state mapping, except in pilot mode, where sub-states MAY be derived from platform events or skipped (§20.8 item 4); the full catalog is in Appendix D.

| Workflow | Parent state → sub-state |
|---|---|
| Bug Fix | `CONTEXT_READY` → `REPRODUCING`, `REPRODUCED` · `PLANNED` → `ROOT_CAUSE_IDENTIFIED` · `EXECUTING` → `FIXING` |
| Database Migration | `PLANNED` → `MIGRATION_DESIGNED` · `VERIFYING` → `MIGRATION_VALIDATED` · `RELEASING` → `MIGRATING` · `OBSERVING` → `POST_MIGRATION_VERIFICATION` |

**When a task's basis changes**

1. **Detection.** The task runner MUST monitor `basis` (§6.4). The basis is considered changed when an artifact in the basis changes version or moves to `NEEDS_REVALIDATION`, `SUPERSEDED` or `DEPRECATED` (§5.8); when an assumption in the basis is refuted; when a finding during execution goes beyond the approved scope or assumptions (§6.7); or when the official tier or reversibility crosses the threshold of a pre-implementation checkpoint that has not yet run (§6.7.3 rule 4).
2. **Block the next gate.** The task MAY continue independent work, but MUST NOT pass the next gate (plan approval, merge approval, merge queue, release, or `publish` for a change channel that does not use PRs) until a decision is made.
3. **The decision** MUST be recorded as one of the following values, with the corresponding consequences.

| Decision | When | Consequence |
|---|---|---|
| `continue` | The change does not touch the task's scope, acceptance criteria, assumptions or design | The basis is updated to the new version; the reason is recorded; approvals remain in effect |
| `replan` | The implementation plan or design is affected, but the approved scope and acceptance criteria remain correct | The task returns to `PLANNED`; evidence of the old commit is kept but not used to advance |
| `reapprove` | The acceptance criteria, scope, high-impact assumptions, or the approved design change | All approvals whose basis is affected lose validity; the corresponding checkpoints (§6.12) run again |
| `stop` | The task's goal is no longer valid or has been superseded | The task moves to `CANCELLED`, or to `BLOCKED` pending the owner; unfinished work is saved together with the reason |

4. **Who decides.** The requester MAY decide `continue` only when the task is tier Low and no acceptance criterion or scope is touched. All other cases MUST be decided by the authority that approved the affected checkpoint. The agent MAY propose and record the analysis, but MUST NOT decide on its own.
5. **After merge.** A task that is already `MERGED` MUST NOT be reopened when its basis is superseded. If needed, a new task (a forward fix or a follow-up change) MUST be created, and reconciliation (§5.10) handles the knowledge.
6. **The history of a completed task is immutable.** The task record, evidence, approvals and basis snapshot (`id@version` and the digest of each artifact) MUST be kept as they were at the time of execution. When an artifact is later modified or superseded, the old task MUST NOT be rewritten; the new artifact MUST link back through `supersedes`.

**Rationale.** Every fix path returns to `VERIFYING` so that no evidence from an old commit is used for a new commit. A loop budget is never reset because a reset would make the budget unlimited; loosening a budget is permitted, but it must leave a trace as a human decision. A rollback is complete only when the commit is reverted on main or deployment of that commit is blocked, because otherwise the next deployment would bring the defect back.

If humans have to fill in records just so the Factory has data, the records will be missing or wrong, and the Factory's path becomes heavier than the ordinary path. Labels, PR review and approval events, CI runs and structured comments already contain most of the necessary information.

Sub-states let each workflow describe its own progress without creating a competing lifecycle. Because policy, approval and budgets reference only the parent state, shared tooling works with every workflow.

A task in progress may rely on a spec, design, ADR, contract or assumption that is changed or superseded during execution. Blocking at the next gate (instead of stopping immediately) lets the task still do independent work, but it cannot be approved or merged on a stale basis. The history of a completed task is kept immutable so that the question "why was this change made" is always answered by the version of the knowledge at that time, not by the current version.

**Example.** Bug Fix has `REPRODUCING` and `REPRODUCED` within `CONTEXT_READY`. The fix path `REPAIRING → VERIFYING` resets the sub-state of `VERIFYING` to its initial sub-state.

## 6.6. Standard execution workflow

The following table is the skeleton that every Task Workflow Specification (§6.7) MUST implement. A specific workflow MAY detail, split or merge steps, but MUST map each of its steps to a step of this skeleton.

| # | Step | Content | Omittable |
|---|---|---|---|
| 1 | **Intake** | Validate the task contract and references; Requirement Review (§6.12) | No |
| 2 | **Classification** | Determine the task type and workflow, affected products/repositories, risk, data class (§10.6) and reversibility | No |
| 3 | **Context discovery** | Retrieve the specs, architecture, ownership, rules, contracts, dependencies and relevant source symbols; record `basis` | No |
| 4 | **Impact analysis** | Check the changed paths and downstream consumers; extend to multi-repo scope when needed | Merged into step 3 for tier Low |
| 5 | **Planning** | A plan for complex/high-risk work; small changes use a lightweight plan | Lightweight plan for tier Low |
| 6 | **Approval gate** | Specification, Design and Plan Review when the conditions of §6.12 hold | No, when the conditions hold |
| 7 | **Workspace preparation** | A clean branch/worktree/sandbox with the base commit pinned | No, for the `code` change channel |
| 8 | **Implementation** | In small batches; update the plan/checkpoints as appropriate | — |
| 9 | **Self-verification** | Fast local checks and the profile's initial tests | — |
| 10 | **Independent verification** | The mandatory tests/evals through the verification adapter (§7.16), with an independent oracle for `critical` acceptance criteria (§6.4.1); Independent Verification (§6.12) | No, when the task has a `critical` acceptance criterion or is at tier Medium/High |
| 11 | **Review** | Diff, evidence, risks and unresolved findings, through the AI review pipeline (§7.9) with convergence control (§7.11); Implementation Review (§6.12) | No |
| 12 | **Knowledge synchronization** | Run `knowledge.impact`, record reconciliation decisions for affected artifacts (§5.9–5.10) | No |
| 13 | **Packaging** | Evidence manifest and PR description | No |
| 14 | **Approval and merge** | Branch protection and risk/trust rules; approval bound to the commit SHA (§9.8); land through the merge queue | No |
| 15 | **Release and observation** | Release Readiness Review when required (§6.12); monitor the required signals | When the change is not deployed |
| 16 | **Learning** | Classify findings, record workflow metrics (§13.7) and propose improvements | No |

1. A workflow MUST NOT omit any step marked "No" in the last column.
2. A change channel that does not use PRs (§12.6) MUST apply steps 1–3, 10, 12 and 14–16 per its reduced lifecycle.
3. In pilot mode, step 16 MAY be satisfied per §20.8 item 4 (the findings and knowledge impact part of the PR description, together with metrics collected from events); it MUST NOT be dropped entirely.

## 6.7. Task Workflow Specification and playbooks

A **Task Workflow Specification** is a versioned contract that defines the workflow of a task type. A **Playbook** is the file that implements a Task Workflow Specification; in this Standard, every playbook is a workflow spec.

1. Every task executed by an agent MUST run under an `ACTIVE` Task Workflow Specification.
2. A workflow spec MUST be a structure validated by a schema (Appendix C, Appendix K), not a prose list of steps.

**Rationale.** Not every task follows the same process: a bug fix needs reproduction first, a data migration needs a design and a rollback plan, a refactoring needs a behavior baseline. The Task Workflow Specification packages that difference into a versioned, validatable contract, instead of prose that an agent can interpret arbitrarily.

### 6.7.1. Contract

Every Task Workflow Specification MUST declare the following components.

| Component | Requirement |
|---|---|
| Identity | `id`, `version` (semver), `owner`, `status` per the knowledge artifact lifecycle (§5.8) |
| Task class | The task classes the workflow serves, and the ceiling tier (risk ceiling) up to which the workflow may be used |
| Trigger | The event or request that starts the workflow (a labeled issue, a command, a schedule, an incident, another workflow handing over) |
| Input | Required and optional inputs, each with a type and a source |
| Precondition | Conditions that must hold before the first step (for example Requirement Review has passed, the repository has a golden command) |
| Step | An ordered list. Each step declares: `id`, `kind` (`mandatory` \| `conditional` \| `optional`), `when` for a `conditional` step, the corresponding skeleton step (§6.6), `role`, `skills`, `tools`, `inputs`, `outputs`, `verification` (a check ID or completion criterion), `onFailure`, and the recovery properties of §6.14 (`timeout`, `retry`, `idempotent`, `sideEffect`, `rollback`/`compensation`) |
| Sub-state | Sub-states per parent state and the initial sub-state (§6.5) |
| Checkpoint | The review checkpoints that apply (§6.12), each `mandatory` or `conditional` with its condition, and its position in the step sequence |
| Handoff | The handoffs between roles: required artifacts, valid states, the list of excluded context (§6.13) |
| Gate and budget | Additional approval gates; loop budgets equal to or lower than the policy ceiling (§6.4) |
| Recovery | Default behavior on timeout, `ERROR`, budget exhaustion, and the escalation path (§6.14) |
| Completion | The required outputs for the task to be considered complete |
| Metric | The workflow metrics recorded (§13.7) |

Rules:

1. **Only `ACTIVE` workflows may be executed by an agent.** An agent MUST NOT execute a workflow that is not in `ACTIVE` state. A task of a type that has no `ACTIVE` workflow spec (including types that only have an outline in Appendix D) MUST run under the **`generic`** workflow: the core workflow defined by the Standard (Appendix C.5), which implements exactly the §6.6 skeleton with every step marked "No" being `mandatory`. The `generic` task class MUST NOT exceed trust T0, so `allowedAutonomy` is no higher than `implement-and-request-review` (§9.4).
2. **A workflow references checks by ID.** A workflow MUST reference checks by ID and MUST NOT define its own verifier commands. Check execution MUST go through the verification adapter (§7.16).
3. **A workflow does not loosen.** A workflow MUST NOT drop a mandatory skeleton step (§6.6), a mandatory checkpoint (§6.12) or a gate of §9, and MUST NOT raise budgets above the policy ceiling.
4. **A workflow spec is a control file** (§7.13) and is subject to evaluation (§9.1). A change that removes or lowers a `mandatory` step, a checkpoint, a gate, or raises a budget MUST be classified as loosening.

### 6.7.2. Workflow catalog

1. The Factory MUST support at least the workflow types in the following table.
2. A workflow MAY remain in `DRAFT` or `PROPOSED` (with an outline in Appendix D) until a repository needs it; when promoted to `ACTIVE`, the workflow MUST have a full spec, an owner and evals.

| Workflow | Main sub-states | Workflow-specific mandatory checkpoints (in addition to `requirement-review`, `implementation-review`, `independent-verification` and the general conditions of §6.12, for example `plan-review` for every `irreversible` step) | Reference status |
|---|---|---|---|
| Bug Fix | `REPRODUCING`, `REPRODUCED`, `ROOT_CAUSE_IDENTIFIED`, `FIXING` | Plan Review at tier High | Full spec in Appendix C |
| Feature Development | `SPECIFYING`, `IMPACT_ANALYZED`, `DESIGNED`, `IMPLEMENTING_SLICE`, `ACCEPTANCE_VERIFYING` | Specification Review when the §6.4.1 threshold is exceeded; Design Review when architecture is touched | Outline in Appendix D |
| Refactoring | `INVARIANTS_DEFINED`, `BASELINE_CAPTURED`, `REFACTORING_BATCH`, `EQUIVALENCE_VERIFIED` | Design Review when module boundaries change | Outline |
| Dependency Change | `CAPABILITY_SEARCHED`, `CANDIDATE_EVALUATED`, `UPDATING`, `CONSUMERS_VERIFIED` | Design Review for a new dependency or a major upgrade | Outline |
| API Contract Change | `CONSUMERS_IDENTIFIED`, `CONTRACT_PROPOSED`, `CONTRACT_APPROVED`, `PROVIDER_IMPLEMENTING`, `COMPATIBILITY_VERIFIED` | Specification Review, Design Review; Plan Review when breaking | Outline |
| Database Migration | `MIGRATION_DESIGNED`, `MIGRATION_VALIDATED`, `MIGRATING`, `POST_MIGRATION_VERIFICATION` | Design Review, Plan Review, Release Readiness Review | Outline |
| Incident Response | `TRIAGING`, `MITIGATING`, `STABILIZING`, `ROOT_CAUSE_ANALYSIS` | `release-readiness` per the incident variant (incident commander); verification MAY be `deferred` through break-glass (§12.7) | Outline |
| Release / Deployment | `CANDIDATE_VERIFIED`, `STAGED`, `CANARY`, `PROMOTING`, `HEALTH_CONFIRMED` | Release Readiness Review | Outline |

The `generic` workflow is defined as mandatory in Appendix C.5. The other workflows in the catalog (Investigation, Codebase Gardening, Lookback, SDK Release, AI Application Change, Knowledge Reconciliation, Performance Hill-climb, PR Babysit, Session Pickup) are described in Appendix D.

### 6.7.3. Workflow adaptation

A workflow MUST be flexible enough to handle problems newly discovered during execution without opening a path to bypass controls. Every step MUST be of one of the following kinds, and whether a step runs MUST be decided per the table.

| Step kind | Meaning | Who decides whether the step runs |
|---|---|---|
| `mandatory` | Always runs | No one; it cannot be skipped |
| `conditional` | Runs when the `when` condition holds | Policy. The condition is evaluated from declared inputs (tier, path, metadata, output of a prior step), not from the agent's assertion. If the condition cannot be evaluated, the step runs |
| `optional` | May run at discretion | The agent or the requester |

Rules:

1. **Additive only.** The agent MAY add steps or handling branches (extra investigation, extra tests, specialized review) with a reason. An added step MUST share the workflow's budget and MUST NOT replace a mandatory step.
2. **No bypassing controls.** The agent MUST NOT skip, reorder past, or merge away a `mandatory` step, a `conditional` step whose condition holds, a checkpoint or an approval gate. A mandatory step MUST be permitted not to run only when the task is `CANCELLED`, or when the workflow spec is changed through the control file process.
3. **A finding that changes the basis.** If a new finding changes an approved assumption, scope or design, the task MUST apply the basis change process (§6.5): stop at the next gate, re-evaluate the plan, and invalidate the related approvals when needed.
4. **Risk only increases, and the official tier comes from the diff.** The tier computed at the Classification step is only the **expected tier**, based on estimated paths. The **official tier** MUST be recomputed from the diff on every entry into `VERIFYING` (P19). The tier within a task MUST NOT decrease, and new approval slots MUST be added by union (§9.10). If the official tier or reversibility crosses the threshold of a pre-implementation checkpoint that has not run (`specification-review`, `design-review`, `plan-review`), that is a basis change with a mandatory `reapprove` decision: the task MUST return to `PLANNED` (or `AWAITING_PLAN_APPROVAL`) and that checkpoint MUST run before the task proceeds. Adding a slot after implementation MUST NOT be used as a substitute for pre-implementation review.
5. **No implicit scope expansion.** A problem outside the scope MUST become a new linked task, unless the requester expands the scope through Requirement Review.
6. **Switching workflows.** If the task turns out to belong to a different type, the task MUST be moved to the appropriate workflow as a `replan` or `reapprove` decision, and MUST NOT continue under the old workflow.
7. Added steps, the results of condition evaluation and skipped optional steps MUST be recorded in the task record.

**Rationale.** The agent may add work, but may not remove controls. The condition of a `conditional` step is evaluated from declared inputs rather than from the agent's assertion, because the agent has an incentive (even if unintentional) to conclude that an expensive step does not need to run.

The tier only increases within a task because the tier computed at the Classification step is based on estimated paths, while the actual diff may touch more. Adding an approval slot after implementation cannot replace pre-implementation review: a wrong design found after the code is written costs far more than one found before.

**Example.** A bug fix that turns out to require a data migration moves to the Database Migration workflow as a `replan` or `reapprove` decision.

## 6.8. Workflows and autonomous agents

1. The Factory SHOULD use deterministic workflows when the sequence is known and the decision scope is limited.
2. The Factory SHOULD use the reasoning capability of autonomous agents for exploration, diagnosing ambiguity or planning.
3. The Factory MAY combine both when needed.
4. The Factory SHOULD use multi-agent execution only when independent context, distinct expertise or parallelism is genuinely greater than the cost of coordination, merging and verification.
5. An agent MAY perform long-running, goal-oriented tasks (from a few hours to a few days, possibly spawning child agent sessions, for example subagents).
6. A long-running task MUST checkpoint its state into the task record, MUST respect time and cost budgets, and MUST resume from the record rather than from conversation memory.
7. While waiting at a human gate, the agent SHOULD continue unblocked work (investigation, tests, documentation) instead of sitting idle.
8. The agent MUST NOT pass a human gate that is pending.

**Rationale.** Multi-agent execution has coordination, merging and verification costs. Long-running, goal-oriented tasks (which may spawn sub-agents) are described in practice in [R28]; continuing unblocked work while waiting at a human gate, without passing the gate, follows [R06].

**Example.** Suggested classification:

- **Workflow-first:** releases, dependency updates, standard bug fixes, documentation checks, periodic code gardening.
- **Agent-led:** complex investigations, unfamiliar code, root cause analysis, architecture exploration.
- **Combined:** feature development, cross-repository contract changes, incident remediation.

## 6.9. Workspace isolation and parallelization

1. Every task executed by an agent MUST have an isolated workspace (its own worktree, branch or sandbox).
2. The Factory MUST pin the base commit and record it in the task metadata.
3. The Factory MUST NOT let concurrently running agents modify the same file without an explicit coordination strategy.
4. The Factory MUST define ownership for shared files, lockfiles, generated files and migrations.
5. Rebase/merge MUST occur only through an explicit integration step with fresh verification.
6. The Factory MUST use a merge queue so that two changes that each pass individually cannot combine into a broken main branch.
7. The Factory MUST limit concurrency based on CI capacity, model/provider limits and the observed conflict rate.
8. The Factory MUST retain resumable task state, and MUST NOT rely on transient conversation context as the sole task record.

**Rationale.** Two changes that each pass individually can still combine into a broken main branch [R29]. When many agents run in parallel, this probability grows with the number of concurrent changes, so both a merge queue and a concurrency limit are needed.

## 6.10. Contract for tools and MCP

1. Every tool integration MUST declare:
   - The purpose and supported actions.
   - The input/output schema.
   - The permission scope and identity.
   - Data sensitivity and retention period.
   - Timeout, retry, rate limit and error handling behavior.
   - Network and file system access.
   - The version and owner.
   - The level of eval/security test coverage.
2. The Factory MUST NOT grant general-purpose shell, browser, database or production API access by default.
3. The Factory SHOULD prefer narrow tools with structured input and output.
4. Repeated agent actions SHOULD be packaged as a small CLI or driver that ships with a skill.
5. The Factory SHOULD prefer codemods and scripts that a reviewer can rerun over large manual edits.

**Rationale.** Narrow tools with structured input and output are easier to grant permissions to, easier to audit and harder to misuse than a general-purpose shell or browser. Packaging repeated actions into a small CLI or driver that ships with a skill (for example, an application driver over the browser DevTools protocol) brings three benefits: reusable across runs, fewer tokens than ad-hoc exploration, and testable like other code [R05]. Similarly, codemods and scripts that a reviewer can rerun are better than large manual edits ("build the lever") [R06].

## 6.11. Hooks and automation

1. The Factory SHOULD use hooks or equivalent lifecycle callbacks for deterministic actions such as:
   - Loading local context when entering a module.
   - Running format/type/lint checks after a change.
   - Blocking prohibited commands or access paths.
   - Collecting task/agent metadata.
   - Validating output before creating a PR.
   - Recording lessons from a work session as a proposal.
2. A hook SHOULD enforce deterministic policy.
3. A hook SHOULD NOT become a second orchestration framework or create hidden changes that cannot be traced.

## 6.12. Review checkpoints, approval and verification

**Three different activities.** The Factory MUST distinguish the following three activities. A checkpoint MAY require all three, but each activity MUST be recorded separately and MUST NOT be used to substitute for another.

| Activity | Question | Who performs it | Output | Nature |
|---|---|---|---|---|
| **Review** | Is there any problem with the artifact? | A person or an agent (AI review) | Findings (`Important`, `Nit`, `Pre-existing`) with disposition | Advisory; does not by itself permit proceeding |
| **Approval** | Is proceeding permitted? | Only a person with authority per §9.10 | An approval slot bound to the artifact version or commit SHA, with the basis | An accountable decision; an agent never approves (P20) |
| **Verification** | Does the behavior meet objective criteria? | Deterministic checks, independent oracles, through the verification adapter | `CheckResult` (`PASS`, `FAIL`, `ERROR`, `NOT_ASSESSED`) | Evidence; does not replace approval |

Consequence: a clean review pass MUST NOT be treated as an approval; an approval MUST NOT turn a `FAIL` check into a pass (only a valid disposition under §7.5 can do that); a green set of checks MUST NOT replace a mandatory approval.

**Checkpoints.** The Factory MUST apply the following checkpoint table.

| Checkpoint | Subject | Mandatory when | Approver (§9.10) | Pass when | On fail |
|---|---|---|---|---|---|
| `requirement-review` | The Change Brief fields of the Task (§6.4.1) | Every task, before `PLANNED` | The requester (expected tier Low and Medium); the product/domain owner for a new user-facing capability or expected tier High | The fields are complete; every acceptance criterion has an oracle; there is no `unconfirmed` assumption that affects correctness or risk | `NEEDS_CLARIFICATION` |
| `specification-review` | The full spec | The §6.4.1 threshold is exceeded | The product/domain owner; the contract owner when the spec defines a contract | The acceptance criteria are reviewed; critical scenarios are confirmed (Medium/High); the spec moves to `ACTIVE` | The spec stays `PROPOSED`; the task does not enter `PLANNED` |
| `design-review` | Design document, ADR, contract proposal | Changes to module boundaries or architecture; public API or contract; data migration; multiple repositories; a new dependency of platform or runtime kind; creating a new public repository, service or package; tier High | The architect or the code owner of the area; the contract owner when a contract is touched | The options and trade-offs are recorded; the ADR is `ACTIVE` when it is an architecture decision; the impact analysis lists consumers | `PLANNED` (revise) or `REJECTED` |
| `plan-review` | The implementation plan | Tier High (the `plan-approval` slot of A3); tier Medium with reversibility LOW; every task with an `irreversible` step (§6.14); when the workflow requires it | An independent code owner | The plan covers every acceptance criterion; there is a rollback or compensation path for every step with a side effect; every `irreversible` step has a go/no-go condition; checkpoints and budgets are explicit | `AWAITING_PLAN_APPROVAL` → `PLANNED` (revise) or `REJECTED` |
| `implementation-review` | The diff and evidence | Every change | AI review posts findings; approval slot per §9.5 and §9.10 | Every finding has a disposition; no `blocking` integrity sensor hit remains without an accepted justification; the approval slot is filled for the current commit SHA | `REPAIRING` |
| `independent-verification` | Behavior against the acceptance criteria | Every task with an acceptance criterion `critical: true` (§6.4.1); every tier Medium/High task | No approver: this is verification, performed by a verifier with a separate context (§7.7) | Every required check is `PASS` or has a valid disposition; every critical acceptance criterion has evidence from an independent oracle | `REPAIRING` |
| `release-readiness` | The operational readiness record (§12.2), artifacts, the rollout plan | A deployable change to production at tier High; every change with reversibility LOW; publishing a public artifact (SDK, package). At tier Medium, the readiness record lives in the PR and is confirmed by the person who fills the `implementation-review` approval slot, with no need for a second person | The service owner or SRE; the security owner where applicable. Variant when the change is a mitigation in an open incident: the incident commander (slot `incident-commander`), with missing verification `deferred` per §12.7 | The signals and thresholds are confirmed; the rollback or compensation path has been tested; on-call has been notified | `RELEASING` does not start |

Rules:

1. **A workflow only adds checkpoints.** A workflow spec MUST declare which checkpoints apply and where. A workflow spec MAY add checkpoints or stricter conditions, and MUST NOT drop the mandatory conditions in the table above.
2. **Approval is recorded with an approval slot.** An approval MUST be recorded with an approval slot carrying the corresponding `checkpoint` and a kind by approver: `requester-signoff`, `product-owner`, `architect`, `code-owner`, `plan-approval`, `service-owner`, `security-owner` (Appendix J, K).
3. **Approval is bound to the exact subject.** An approval for `requirement-review`, `specification-review`, `design-review` and `plan-review` MUST be bound to the `id@version` (and digest) of the artifact; for `implementation-review` it MUST be bound to the commit SHA (§9.8); for `release-readiness` it MUST be bound to the digest of the artifact being released.
4. **Approval records the basis.** Every approval MUST record the basis it relies on (assumptions, scope, artifact versions). When the basis changes, the corresponding approval loses validity per the decision in §6.5.
5. **Creating a new component is a maintenance commitment.** A `design-review` for a new public repository, service or package MUST pass only when it has: an owner and a backup owner (§9.10), the reason the existing component cannot be extended, a verification profile, and the maintenance burden on the owning team after the addition (§13.2). An agent MUST NOT create a new public repository, service or package on its own outside a task that has passed this checkpoint.
6. **Recorded separately, interactions may be combined.** For a small task, multiple checkpoints MAY take place in the same interaction, but each checkpoint MUST be recorded as a separate `CheckpointRecord` (Appendix K) and MUST be summarized in the PR (Appendix G).
7. **A checkpoint that does not apply** MUST be recorded by policy as `not-applicable` with the condition that was evaluated, and MUST NOT be left blank.

**Rationale.** Review, approval and verification answer three different questions, so merging them loses information: a clean AI review pass says nothing about who permits proceeding, and a green set of checks says nothing about whether the change matches the intent.

Generating code is cheap; maintaining it is not [R35]. Because an agent can create a repository, service or package very quickly, the `design-review` for a new component requires an owner, the reason the existing component cannot be extended, and an assessment of the maintenance burden.

**Example.** For a small task, the requester confirms the Change Brief directly on the issue: one interaction, but it still produces a separate `CheckpointRecord` for `requirement-review`.

## 6.13. Handoff protocol between roles

A **Handoff** is the transfer of responsibility for performing the next step from one role to another: agent to agent, agent to person, or person to agent. The coordination protocol between roles MUST be defined, in addition to the role descriptions (§6.3).

The **handoff record** (`Handoff`, Appendix K) MUST include:

- The task, the source step and the target step; the role and identity of the giving and receiving parties (or the queue).
- The state and sub-state of the task at the time of handoff.
- **Required artifacts:** the output of the source step matching the input of the target step per the workflow spec, each artifact with a reference and a digest.
- The `basis` reference, the decisions already made, open questions and assumptions.
- Evidence references; the workspace (branch, base SHA, head SHA); the budget used; the timeout of the target step.
- The party responsible for the next step.

Rules:

1. **Valid state.** A workflow spec MUST declare in which state each handoff is valid. A handoff in any other state MUST be rejected.
2. **The receiver checks before starting.** The receiver MUST verify that every required artifact is present and valid (correct schema, artifacts in the basis are `ACTIVE`, SHA matches). If anything is missing, the receiver MUST record `rejected` with the list of missing items and return to the source step; after the number of rejections the workflow declares (*suggested default* 2), the handoff MUST be escalated. The receiver MUST NOT fill in missing context by guessing.
3. **Context passes through the task record**, MUST NOT pass through conversation memory. The receiver MUST be able to work solely from the handoff record and the referenced artifacts.
4. **Preserve independence.** For a handoff into a verifier or reviewer role, the workflow spec MUST declare a list of excluded context (for example the implementer's notes, reasoning and plan), so that the verifier derives expectations from the acceptance criteria and contracts, not from the implementer's explanation (§7.7).
5. **Accountability is not transferred.** The requester MUST remain accountable for the task (§6.3). A handoff MUST transfer only responsibility for performing the step. At every moment, the task MUST have exactly one party responsible for the current step.
6. A handoff to a person for approval MUST use the evidence bundle in the reviewer's reading order (§7.6).

**Rationale.** The role descriptions (§6.3) say what each role does, but not what the next role receives and when. Without a handoff protocol, the receiver tends to fill in context by guessing or depends on the giver's conversation memory. The list of excluded context preserves verifier independence: if the verifier reads the implementer's reasoning, it tends to check exactly what the implementer believes to be correct.

**Example.** The implementer hands off to the verifier only at `VERIFYING`, with the head SHA pinned.

## 6.14. Recovery from failure

Every step in a workflow spec MUST declare the following recovery properties; the Factory MUST apply them uniformly to every workflow.

| Property | Value |
|---|---|
| `timeout` | The maximum duration of the step |
| `retry` | The maximum number of retries, the backoff, and the kinds of result that are retried (`ERROR`, timeout) |
| `idempotent` | Whether rerunning the step creates additional side effects |
| `sideEffect` | `none` \| `reversible` \| `compensable` \| `irreversible` |
| `rollback` | How to undo, for a `reversible` step |
| `compensation` | The reviewed compensating action, for a `compensable` step |

**Mechanisms.** The Factory MUST apply the following recovery mechanisms.

| Mechanism | Definition | Rule |
|---|---|---|
| Timeout | A step exceeds its maximum duration | The result is an `ERROR` of the step, not a `FAIL`; retry per `retry`, then escalate |
| Retry | Rerun the same step with the same input | Only for `ERROR` and timeout. A `FAIL` MUST NOT be retried blindly; it enters the repair loop and consumes loop budget (§6.4). Automatic retry applies only to `idempotent` steps or steps with an idempotency key. The number of retries is recorded but does not consume loop budget |
| Idempotency | Rerunning does not create additional side effects | A step with external side effects MUST be idempotent (the idempotency key consists of task, step and input) or MUST NEVER be retried automatically |
| Checkpoint | Save the step's state into the task record | After every `mandatory` step and before every gate: output, workspace SHA, budget used, sub-state |
| Pause / resume | Intentionally suspend execution (waiting for a human gate, time budget exhausted, degraded mode §12.7) | Resume from the nearest checkpoint, not from conversation memory. Before resuming, recheck the basis (§6.5), the base commit, the validity of approvals and credentials |
| Rollback | Return the system to its previous state by undoing the action itself | Only for `reversible` side effects (revert a commit, flag-off, redeploy the previous artifact). A rollback is complete only when the commit is reverted on main or deployment is blocked (§6.5) |
| Compensation | A new, forward-moving action that neutralizes the business consequence of a side effect that cannot be undone | For `compensable` side effects. A compensation MUST be declared and reviewed before the step runs, and is itself a change with risk, verification and approval |
| Escalation | Hand the decision to a person | When retries or the loop budget are exhausted, a compensation fails, a decision about the basis is needed, or a handoff is repeatedly rejected. The escalation record states who receives it, why, the current state and the options |

Rules:

1. **Rollback differs from compensation.** A rollback removes the action; a compensation adds a new action and leaves a trace of both. The Factory MUST NOT describe a compensation as a rollback.
2. **An `irreversible` step** (cannot be undone or compensated) makes the task reversibility LOW (§9.5), so the autonomy level follows the §9.5 matrix. Regardless of tier, that step MUST have `plan-review` and `release-readiness`, MUST run only after an explicit instruction from a person at that very step, and MUST NEVER be retried automatically.
3. **A `compensable` step with no declared compensation** MUST be treated as `irreversible`.
4. An agent MUST NOT perform a production rollback or data recovery on its own (rationale: §18.1). The Change Shepherd MAY flag-off and pause within policy limits (§12.2.1).
5. Every recovery event MUST be recorded (`RecoveryEvent`, Appendix K) and MUST be included in the workflow metrics (§13.7).

**Rationale.** A compensation does not erase the original action but adds a new one, so calling it a rollback would hide the fact that the system actually suffered the side effect. A `FAIL` is not retried blindly because rerunning a check that failed due to a real defect only costs money and may mask flaky behavior; the repair loop and the loop budget are where `FAIL` is handled.

**Example.** Compensation for a `compensable` side effect: an email already sent → a correction email; a fee already charged → a refund; a migration already run → a compensating migration. The consequence of an `irreversible` step under the §9.5 matrix: tier Low becomes A2, tier Medium becomes A3.

## 6.15. Interactive agent use on the developer machine

Gates MUST NOT depend on whether a PR used an agent.

Rules:

1. **The developer is the requester and author.** A PR created from an interactive session is the developer's PR and MUST pass through exactly the same gates as every other PR: centralized required checks, integrity sensors, separation of duties, approval by tier (P19). The Factory MUST NOT have a separate gate for "PRs that used an agent", and MUST NOT have a lighter gate.
2. **Gates apply uniformly.** Integrity sensors, control file protection and separation of duties checks MUST run on every PR within the governed scope, regardless of whether the author is a person, a Factory agent or a developer using a local agent.
3. **Trust and autonomy apply only to Factory-run agents.** Trust levels (§9.4) and the zero-human path (§9.5) MUST apply only to agent configurations run and recorded by the Factory. An interactive session MUST NOT accumulate trust and MUST NEVER be merged under A0.
4. **Local evidence does not satisfy required checks.** Results from the developer's machine MUST NOT be used to satisfy a required check at the `merge` or `publish` gate; only results from the centralized pipeline count (§7.15, §7.16).
5. **Same knowledge, same tools.** A local agent SHOULD use the same `AGENTS.md`, skills and workflow specs as the Factory's agents, and run self-verification (§7.10) before opening a PR.
6. **Declaration, not control.** The PR template has an optional field indicating whether an agent was used (Appendix G). This field MUST be used only for metrics and learning; it MUST NOT change gates and MUST NOT be used to assign individual blame.
7. The bypass rate (§13.2) MUST be measured to know whether the Factory's path is light enough.
8. The Factory agent path SHOULD NOT be heavier than the human-written PR path for the same tier.

**Rationale.** Most agent use happens interactively, in an IDE or CLI on the developer machine, not through the task runner. The Factory does not try to distinguish a commit the developer wrote by hand from a commit the developer made with an agent; an "official agent" path heavier than the human path only creates an incentive to use agents covertly. Results from the developer machine are useful for a fast loop, but are not counted at the gate because the local environment is not controlled. If the bypass rate is high, developers are avoiding the Factory's path, and that is a signal the path is too heavy.
