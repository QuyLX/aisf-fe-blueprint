[Contents](README.md) · [← 8. Verification](08-verification.md) · [10. Autonomy Policy →](10-autonomy-policy.md)

<a id="s9"></a>

# 9. Agent Operating Model

<a id="s9-1"></a>

## 9.1. Role · Skill · Playbook · Tool

Role (responsibility), Skill (reusable, versioned capability), Playbook (a versioned file that implements a Standard **Task Workflow Specification**, §6.7), Tool (an action outside the model). One agent may hold several roles; use subagents only when context isolation or real parallelism is needed. The reviewer is always independent and read-only.

<a id="s9-2"></a>

## 9.2. Playbook (Task Workflow Specification)

Each playbook is a workflow spec under the contract of Standard §6.7.1, validated by schema. A playbook uses the ID of the Standard's `coreWorkflow` when the purpose matches; a profile-specific playbook states which core workflow it elaborates.

| Playbook | Core workflow (Standard §6.7.2) | Purpose |
|---|---|---|
| `investigation` | `investigation` | Understand code, routes, root cause |
| `bug-fix` | `bug-fix` | Reproduce → failing test → fix → verify |
| `feature-development` | `feature-development` | From a spec with approved acceptance scenarios |
| `console-module` | elaborates `feature-development` | Create/extend a module under the Console Module Contract |
| `refactoring` | `refactoring` | Preserve behavior, evidence parity |
| `design-system-change` | elaborates `api-contract-change` | Component/token + consumer impact + codemod |
| `api-contract-change` | `api-contract-change` | Regenerate client/mock + consumers + docs |
| `content-model-change` | elaborates `database-migration` | Model + migration + consuming pages |
| `dependency-change` | `dependency-change` | Patch/minor/major handled separately |
| `codemod-migration` | elaborates `refactoring` | Write/review a codemod, apply it in bulk |
| `test-design` | elaborates `feature-development` (Specifying step) | From requirements and ACs → propose scenarios, personas and test cases; an approver signs off before they become oracles ([§9.10](#s9-10)) |
| `regression-from-incident` | elaborates `bug-fix` | Bug or incident → reproduction test that fails on the faulty commit → new scenario in the Journey Registry → seeded defect candidate and eval case ([§9.10](#s9-10)) |
| `release-deployment` | `release-deployment` | Promote a release candidate through the `release` channel ([§11.5](11-delivery-change-management.md#s11-5)) |
| `codebase-gardening` | `codebase-gardening` | Gardening ([§12](12-evaluation.md#s12)) |
| `knowledge-reconciliation` | `knowledge-reconciliation` | Scheduled or batch reconciliation ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| `performance-hill-climb` | `performance-hill-climb` | Optimize against the performance budget ([§4.7](04-technical-foundation.md#s4-7)) |
| `docs-update` · `marketing-page` · `accessibility` · `security-review` | elaborate `feature-development` or `bug-fix`, declared in the spec | Specialized topics |

**Required components of every playbook** (Standard §6.7.1): `id`, `version`, `owner`, `status` per the knowledge artifact lifecycle; the task classes served and the ceiling tier; trigger; input; precondition; list of steps, each with `kind` (`mandatory` | `conditional` | `optional`), `when`, the corresponding framework step of Standard §6.6, `role`, `skills`, `tools`, `inputs`, `outputs`, `verification` (check IDs, never a separate verifier command), `onFailure` and recovery properties ([§9.12](#s9-12)); sub-states; checkpoints ([§9.11](#s9-11)); handoff; budget (time, `cost` ceiling, and the Standard counters `verifyRepairIterations` 3, `reviewRounds` 3, `mergeQueueAttempts` 2, which may only be lowered; Standard §6.4); completion; metric; eval cases.

Rules:

1. **Only an `ACTIVE` playbook may be executed by an agent.** A task class without an `ACTIVE` playbook runs only under the Standard §6.6 framework, with autonomy no higher than `implement-and-request-review`.
2. **No framework step is skipped.** Each step maps to a step of the Standard §6.6 framework; a playbook MUST NOT omit any step marked as non-omittable, including step 12 (knowledge synchronization, `knowledge.impact`).
3. **A sub-state belongs to exactly one parent state.** A playbook uses the `coreWorkflow` sub-states in the vocabulary when elaborating a core workflow, and registers its own sub-states in its own namespace (for example `console-module`: `PLANNED` → `MANIFEST_DESIGNED`; `VERIFYING` → `JOURNEY_VERIFIED`). Policy, approval, budget and conformance reference parent states only ([§18.2](18-aisf-conformance.md#s18-2)).
4. **Adaptation is additive only** (Standard §6.7.3). An agent may add steps or branches with a stated reason; it MUST NOT drop, reorder past, or merge away a `mandatory` step, a `conditional` step whose condition holds, a checkpoint or a gate. `when` conditions are evaluated by policy from declared inputs (sub-tier, path, surface, output of an earlier step), not from the agent's assertion; a condition that cannot be evaluated means the step runs.
5. **A playbook is a control file** ([§8.7](08-verification.md#s8-7)). Dropping or lowering a `mandatory` step, a checkpoint or a gate, or raising a budget, is loosening (R3). A new version runs against the task class's eval set before it moves to `ACTIVE` ([§12](12-evaluation.md#s12)).

<a id="s9-3"></a>

## 9.3. Task contract & lifecycle

Every work item uses the **Standard Task contract** (Standard §6.4, §6.4.1). Frontend adds the surface and journey fields:

```yaml
kind: Task
metadata:
  id: TASK-2026-01431
  requestedBy: user:dev-compute            # recorded by the task runner from the triggering event (§10.5)
spec:
  goal: Creating an instance with insufficient quota shows the quota error and never leaves a pending row
  why: issue CONSOLE-2210
  taskType: bug-fix
  workflow: bug-fix@1.0.0                  # ACTIVE playbook (§9.2)
  repositories: [console]
  surfaces: [console/compute]              # x-fe
  journeys: [console/compute.create-instance]   # x-fe, Journey Registry (§7.7)
  scope:
    in: [error handling of the create-instance flow]
    out: [quota API contract changes]
  doneWhen:
    - { id: AC-1, statement: "The quota-exceeded scenario is reproduced on the base commit", oracle: new-independent-test }
    - { id: AC-2, statement: "The quota error is shown and no optimistic row remains after reload", oracle: new-independent-test, ref: AC-CI-4 }   # journey AC (§7.7)
    - { id: AC-3, statement: "The operational and x:a11y profile checks pass", oracle: existing-test }
  assumptions: []
  howToVerify: "factory verify:runtime --surface console/compute, plus stress mode"
  howToRevert: Revert commit; no data change
  basis:                                   # captured at CONTEXT_READY/PLANNED (Standard §6.4, §6.5)
    artifacts:
      - ADR-FE-0007@2                      # Knowledge Records, id@version (§7.3)
      - journey:console/compute.create-instance@4
      - contract:openapi:compute@1.4.0
    assumptions: []
  policy:
    budgets:
      verifyRepairIterations: 3
      reviewRounds: 3
      mergeQueueAttempts: 2
      cost: { limit: 20, unit: usd }       # AI + CI spend attributable to this task (Standard §6.4)
```

The done checks (`doneWhen`) are mandatory. For R2 and above, they are approved before implementation (FE-TEST-003).

**Cost ceiling (Standard §6.4).** The effective ceiling is `min(policy, playbook, task)`; warning, stop and raising the ceiling follow the Standard. Profile defaults in policy (*suggested*, decided in Phase 0 together with the cost estimate, [§15.1](15-roadmap.md#s15-1)): R0 USD 5, R1 USD 10, R2 USD 20, R3 USD 50; in the pilot (Slice 0), USD 20 per `bug-fix` task and USD 5 for `docs.link-fix`. Ceilings are tuned per task class once a Cost per Verified Accepted Work Item baseline exists ([§13](13-metrics.md#s13)); raising a default is policy loosening.

**Orchestration records through the end of Slice 1.** The factory does not build its own workflow engine in Slice 0 and Slice 1. The Task, `CheckpointRecord` ([§9.11](#s9-11)), `Handoff` and `RecoveryEvent` ([§9.12](#s9-12)), `basis`, `oracleRefs` with hashes ([§9.10](#s9-10)) and the verifier's separate identity are **derived from issue tracker, PR and CI events**, and stored as CI artifacts and PR metadata (check runs, labels, structured comments following the schema). The "task runner" at this stage is the CI jobs triggered by events; the verifier runs under a separate CI or bot identity, distinct from the implementer's, and is recorded in the audit log. Whether to build or buy a task runner or orchestration platform is decided before Phase 3 ([§17](17-decisions-to-make.md#s17)).

**Basis changes mid-task (Standard §6.5).** The task runner tracks `basis`. When a record in the basis changes version or moves to `NEEDS_REVALIDATION`, `SUPERSEDED` or `DEPRECATED`, when a pinned contract changes version, or when an assumption is refuted, the task MUST NOT pass the next gate (plan approval, merge approval, merge queue, release) until a `continue`, `replan`, `reapprove` or `stop` decision is made. The requester may decide `continue` only when the task is R0/R1 and no AC or scope is touched; in every other case the authority that approved the affected checkpoint decides. The agent proposes with analysis and does not decide on its own. A task already `MERGED` is not reopened; a completed task keeps its evidence and basis snapshot (`id@version` and digest).

The lifecycle uses the Standard states (Standard §6.5). The frontend step at each state:

| Standard state | Frontend step |
|---|---|
| `REQUESTED` → `TRIAGED` | Intake; task class and `ACTIVE` playbook; brief fields; `requirement-review` ([§9.11](#s9-11)) |
| `CONTEXT_READY` | Surface resolve; `factory context --diff`; governing records; `factory search` for cross-repo knowledge, with citations ([§7.8](07-factory-metadata-registry.md#s7-8)); record `basis` |
| `PLANNED` (→ `AWAITING_PLAN_APPROVAL` at A3) | Risk classifier ([§10.1](10-autonomy-policy.md#s10-1)); playbook; plan; test plan ([§8.10](08-verification.md#s8-10)); for R2+, the verifier's ACs and tests are approved and locked ([§9.10](#s9-10)) |
| `EXECUTING` | Worktree or sandbox; open a draft PR |
| `VERIFYING` ⇄ `REPAIRING` | `verify:fast` → `verify` → `verify:runtime`; attested `VerificationRun` ([§8.7](08-verification.md#s8-7)) |
| `REVIEWING` | AI review ([§9.9](#s9-9)) and independent review; findings resolved; `knowledge.impact` and the reconciliation decision for affected records ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| `AWAITING_MERGE_APPROVAL` | `PolicyDecision`; approval slots filled for the reviewed SHA ([§10.5](10-autonomy-policy.md#s10-5)) |
| `IN_MERGE_QUEUE` → `MERGED` | Merge queue ([§11.6](11-delivery-change-management.md#s11-6)) |
| `RELEASING` → `OBSERVING` (→ `PAUSED` / `ROLLED_BACK`) | Staged rollout; Change Shepherd ([§11.7](11-delivery-change-management.md#s11-7)) |
| `COMPLETED` | Learning: findings, metrics, gardening |

Content publishing uses the Standard's abbreviated channel lifecycle ([§11.4](11-delivery-change-management.md#s11-4)).

<a id="s9-4"></a>

## 9.4. Judgment boundary

| Decision | Who | How |
|---|---|---|
| Verifiable behavior (logic, a11y, layout invariants, contracts, performance) | Agent | By code, tests, browser evidence |
| Public API, token/theme contract, ownership, compatibility, subjective visuals, product scope | **Human** | The agent escalates, one question at a time, with options and evidence |
| New policy/record | Record owner | Look for an `ACTIVE` record and open PRs first; extend the existing owner |

<a id="s9-5"></a>

## 9.5. Agent security

Least privilege per role/task · write only in the worktree/branch · no production secrets · ephemeral sandbox with an egress allowlist · tool/MCP allowlist with scoped credentials · issues, PR comments, docs, CMS content, web pages and tool output are **untrusted data** · agents cannot write to the evidence store or the policy repo · every action is audited · periodic evals: prompt injection, tool misuse, exfiltration, verifier tampering.

**Agents on developer machines (Standard §6.15).** A PR created by a developer with a local agent goes through exactly the same gates as any other PR; the developer is the requester and author in `policy.sod`. Results of `factory verify`, `factory preflight` or tests run locally carry `executor: local` ([§8.7](08-verification.md#s8-7) item 6), are shown in the PR for reference and never satisfy a required check. Interactive sessions do not accumulate trust and never go through A0 ([§10.4](10-autonomy-policy.md#s10-4)).

<a id="s9-6"></a>

## 9.6. Factory CLI

```bash
factory doctor [--level|--coverage|--control-files|--owners]       # conformance, drift, coverage
factory context <path> | --diff | --query <q>   # governing records + profile + surface
factory search <query> [--kind|--repo|--version|--include-draft]   # Knowledge Index, access-aware, with citations (§7.8)
factory plan [--diff|--surface <id>] [--include <test>]   # test plan with a reason per selection (§8.10)
factory trace <req|journey|AC>            # requirement → journey → AC → test → evidence (§7.7)
factory env <status|up|reset> [--tenant]  # test environment and test tenant lease (§8.8)
factory impact <package|token|contract>   # consumers from the portfolio index + sentinel
factory upgrade [--to <version>]          # upgrade @platform/* / Next.js + run codemods
factory verify[:fast|:runtime] [--surface <id>]
factory preflight [--quick]               # pre-review from a fresh session + in-scope checks
factory generate <app|module|package|page>   # golden path → L2 immediately
factory eval [--suite regression|legibility|seeded]
factory manifest --json                   # self-describes every command, option, response, exit code
```

Conventions: every command supports `--json` with a typed envelope and **stable error codes**; output is token-efficient (`--detail brief|full`, large topics return a table of contents first); docs reflect the version in the repo's lockfile; **CLI-first, MCP is an adapter** (MCP is really needed only for remote services: CI status, evidence store, broker).

<a id="s9-7"></a>

## 9.7. Repository agent-legible

Lint/check errors come with a remediation message (the best channel for teaching agents) · one PR, one intent, with a template per intent · comments explain *why*, never narrate the code.

<a id="s9-8"></a>

## 9.8. Golden paths (generators)

`app --profile authenticated|public-static|docs` (creates a new repo at L2), `console-module`, `admin-module`, `desk-view`, `site-page`, `docs-page`, `cms-block` (schema + component), `chat-surface` (with a mock streaming server), `ds-component` (with contract spec, story, a11y binding), `package`. A deviation from a golden path MUST be declared with an owner. Creating a new repo, app, Console module or public package with a generator still needs `design-review` ([§9.11](#s9-11)); the generator guarantees the quality of what is created and does not replace the decision on whether to create it.

<a id="s9-9"></a>

## 9.9. AI code review (Standard §7.9–7.12)

- **Mode per surface**, set in `factory.yaml` (`review.aiMode`). Every surface starts at `shadow`, moves to `advisory`, then to `blocking`. Raising the mode requires thresholds agreed with the surface owner, met on measured data: precision computed from finding dispositions, the rate of harmful suggestions on PRs known to be clean, and recall on the incident-derived set ([§12](12-evaluation.md#s12)). Raising and lowering the mode are control file changes; lowering the mode counts as loosening.
- **Depth by sub-tier:** R0 and R1 `quick`; R2 `standard`; R3 `deep`. For R2 and R3, review is never downgraded because of cost or incidents; it waits or escalates ([§11.8](11-delivery-change-management.md#s11-8)).
- **Separate security pass** for `x:security-sensitive`, `x:privileged-admin` and `x:ai-surface`. An open security finding raises the PR to at least R2 until it is fixed, or dismissed by the security owner or an independent person.
- **Add-only.** AI review posts findings, never posts approvals, and never fills an approval slot. Every posted finding MUST be fixed, or dismissed by an authorized person with a reason ([§10.5](10-autonomy-policy.md#s10-5)). The implementing agent never dismisses a finding.
- **Injection pre-scan** (`review.injection-prescan`, a Standard core check). A deterministic scan, outside any LLM, looking for text aimed at the reviewer or agent, role markers, and hidden or zero-width characters, in changed files that are neither control files nor declared `promptAssets`. The Chat AI repo declares its prompt templates and injection test fixtures as `promptAssets`; those files are covered instead by code owner approval.
- **Convergence** (Standard §7.11): at most 3 review/fix rounds, after the first pass re-review only on the diff, flag the whole bug class rather than a single case, do not repeat findings.
- **Input:** the diff, the whole repo, the Task contract, the `REVIEW.md` of each affected module, and the resolved check results. When feasible, the reviewer model differs from the author model.
- **Pre-PR self-review:** `factory preflight` runs a review pass with fresh context plus the in-scope checks before the PR is opened (Standard §7.10).

<a id="s9-10"></a>

## 9.10. Agents in testing: test design, regression and oracle protection

Agents take part in testing in several roles, but **no agent both defines the acceptance oracle and writes the code to pass it**.

| Role | Does | Must not |
|---|---|---|
| **Test designer** (playbook `test-design`) | From requirements, journey ACs and the archetype/pack risks, proposes scenarios, personas, missing ACs and a list of test cases; points out the failure branches that `REVIEW.md` describes | Approve its own proposal. For R2+, ACs and scenarios are approved by the product owner or the journey owner before implementation (FE-TEST-003) |
| **Verifier** | Writes the reproduction test and acceptance tests in a separate context and session, from approved ACs; proves the test fails on the base commit | Read or modify the implementer's code before the tests are locked |
| **Implementer** | Writes code; may add supplementary tests | Modify locked oracle files, modify existing scenarios or fixtures, modify ACs in `journeys.yaml`, quarantine tests, accept visual baselines (FE-TEST-002, FE-TEST-004) |
| **Test maintainer** (gardening) | Proposes fixes for flaky tests, orphaned tests, unused fixtures ([§8.11](08-verification.md#s8-11)) | Quarantine (owner only, [§8.8](08-verification.md#s8-8)); delete a test without referencing the deprecation of a journey or AC |

**Oracle protection.**

1. When the task enters `PLANNED`, the oracle files (the verifier's tests, scenarios and referenced ACs) are recorded in the Task as `oracleRefs` with hashes (through the end of Slice 1, in PR metadata, [§9.3](#s9-3)). `integrity.oracle-lock` fails when the implementer changes them ([§8.6](08-verification.md#s8-6)).
2. The verifier and implementer run in two separate sessions, with separate identities in the audit log ([§9.5](#s9-5)). The two roles are never assigned to the same session.
3. If the implementer believes the oracle is wrong, it opens an **oracle change request**: the task returns to `PLANNED`, and the person who approved the ACs (not the implementer, and not the requester for R2+) decides. An oracle change is never in the same PR as the behavior change it checks ([§8.6](08-verification.md#s8-6)).
4. The AI reviewer receives the locked ACs as input, and flags any change that weakens an assertion ([§9.9](#s9-9)).

**Regression tests from bugs and incidents** (playbook `regression-from-incident`):

1. The verifier builds a reproduction scenario from the bug report or postmortem, and proves the test fails on the faulty commit (or on the base commit when the bug is still present).
2. The test is attached to the corresponding journey and scenario; if no suitable scenario exists, a new scenario is proposed and the journey owner approves it.
3. The fix follows the `bug-fix` playbook with that test as the locked oracle.
4. A qualifying incident ([§11.5](11-delivery-change-management.md#s11-5)) produces an additional seeded defect candidate and reviewer eval case ([§12](12-evaluation.md#s12)); the Agent/Eval engineer decides whether it goes into the dev set or the hold-out set.

<a id="s9-11"></a>

## 9.11. Review checkpoints (Standard §6.12)

The three review activities, approval and verification, the checkpoint table, pass/fail criteria and the rules for recording `CheckpointRecord` belong to the Standard. The table below lists only the conditions frontend adds and the specific approvers in this profile; a cell reading "as Standard" means the profile adds nothing.

| Checkpoint | Additional frontend conditions | Approver in the profile |
|---|---|---|
| `requirement-review` | As Standard (R0–R2 correspond to the expected Low and Medium tiers) | Requester; product owner for a new user-facing capability or R3 |
| `specification-review` | Any new or modified AC in `journeys.yaml` ([§7.7](07-factory-metadata-registry.md#s7-7)) | Product owner or journey owner |
| `design-review` | Changes to the Console Module Contract or the public API of the DS and `@platform/*`; content model migration; **creating a new Console module**, including via `factory generate` ([§9.8](#s9-8)) | DS owner for the DS; architect or the area's code owner for the rest |
| `plan-review` | As Standard (R2 with reversibility LOW corresponds to the Medium tier with LOW) | Independent code owner |
| `implementation-review` | As Standard | Approval slots per [§10.4–10.5](10-autonomy-policy.md#s10-4) |
| `independent-verification` | The verifier runs in a separate session and identity ([§9.10](#s9-10)) | No approver |
| `release-readiness` | **Stricter:** promotion through the `release` channel at R2+ (not only the High tier); the first production deployment of an app (`ops.readiness-review`, [§19.11](19-nfr-enterprise.md#s19-11)) | Release owner of the vertical; security owner when applicable |

Through the end of Slice 1, `CheckpointRecord` is derived from issue and PR events and summarized in the PR template ([§9.3](#s9-3)).

<a id="s9-12"></a>

## 9.12. Handoff and recovery (Standard §6.13–6.14)

The `Handoff` record, the recipient's rules, the recovery properties of a step and the timeout, retry, rollback and compensation mechanisms belong to the Standard. Through the end of Slice 1, `Handoff` and `RecoveryEvent` are derived from PR and CI events and stored as CI artifacts ([§9.3](#s9-3)). The profile adds:

**Handoff.**

- The profile's handoff chain: test designer → verifier → implementer → reviewer → approver. Example of a valid state: the implementer hands off to the verifier only at `VERIFYING`, with the head SHA pinned.
- The recipient escalates after 2 rejections (the profile sets the Standard's *suggested* default).
- Context excluded when entering the verifier and reviewer: the implementer's notes, reasoning and plan ([§9.10](#s9-10)).

**Recovery.**

- Examples of side effects in frontend: opening a PR, publishing a DS prerelease, revalidating a CDN cache are `reversible`; sending email or notifications from a real test environment, publishing a public package, removing a URL from the search index are `compensable` when a compensation is declared; deleting data and content model migration are `irreversible`.
- The only production mitigation actions the agent may take are flag-off and pause within the limits of the Change Shepherd ([§11.7](11-delivery-change-management.md#s11-7)).
- `RecoveryEvent` feeds the workflow metrics per playbook ([§13](13-metrics.md#s13)).

---

[Contents](README.md) · [← 8. Verification](08-verification.md) · [10. Autonomy Policy →](10-autonomy-policy.md)
