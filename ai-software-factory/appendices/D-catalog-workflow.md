# Appendix D – Workflow catalog by task type

This appendix describes the workflows the Factory supports (§6.7.2). Bug Fix has a full spec in Appendix C. The other workflows are in **outline** form: they define sub-states, checkpoints, specific steps and recovery characteristics, enough for a repository to write a full spec per Appendix C when needed. An outline is not executed by agents until its full spec is in `ACTIVE` status; until then, tasks of that type run under the `generic` workflow (§6.7.1 rule 1).

In every workflow, `requirement-review`, `implementation-review` and `independent-verification` apply per §6.12 and are not repeated below. Sub-states are written as `PARENT_STATE → SUB_STATE`.

## D.1. Bug Fix

Full spec: Appendix C.

## D.2. Feature Development

| Component | Content |
|---|---|
| Trigger | Product requirement or epic broken down into tasks with independent outcomes |
| Sub-state | `TRIAGED → SPECIFYING` · `CONTEXT_READY → IMPACT_ANALYZED` · `PLANNED → DESIGNED` · `EXECUTING → IMPLEMENTING_SLICE` · `VERIFYING → ACCEPTANCE_VERIFYING` |
| Checkpoint | `specification-review` when the §6.4.1 threshold is exceeded; `design-review` when touching module, architecture or contract boundaries; `plan-review` when tier is High; `release-readiness` when deployable at Medium/High |
| Specific steps | *mandatory:* clarify outcomes and acceptance criteria; impact analysis; implement in small slices behind a feature flag; trace each acceptance criterion to evidence. *conditional:* acceptance scenarios confirmed by the requester or code owner before implementation (Medium/High); runtime verification for critical user journeys |
| Recovery | Side effects are usually `reversible` (flag-off, revert). Accompanying data migrations are split into the Database Migration workflow |
| Output | `ACTIVE` spec when required, spec/AC traceability, PR evidence, updated `implementation-description` documentation |

## D.3. Refactoring

| Component | Content |
|---|---|
| Trigger | Gardening finding, prioritized technical debt, preparation for a larger change |
| Sub-state | `PLANNED → INVARIANTS_DEFINED`, `BASELINE_CAPTURED` · `EXECUTING → REFACTORING_BATCH` · `VERIFYING → EQUIVALENCE_VERIFIED` |
| Checkpoint | `design-review` when changing module boundaries or public interfaces |
| Specific steps | *mandatory:* state behavioral invariants; capture a test baseline and (when applicable) performance metrics before changing anything; refactor in small batches, one verification pass per batch. *conditional:* compare output on recorded traffic or fixtures when there is behavioral risk |
| Specific controls | No intentional behavior change. Integrity sensors `integrity.expected-value-edit` and `integrity.test-count-delta` are `blocking`; any change to an expected value moves the task to another workflow |
| Recovery | `reversible`; each batch can be reverted independently |
| Output | Behavior-preservation evidence; architecture documentation reconciled |

## D.4. Dependency Change

| Component | Content |
|---|---|
| Trigger | Need for a new capability, dependency update bot (requester is the automation owner, §9.10), vulnerability |
| Sub-state | `CONTEXT_READY → CAPABILITY_SEARCHED` · `PLANNED → CANDIDATE_EVALUATED` · `EXECUTING → UPDATING` · `VERIFYING → CONSUMERS_VERIFIED` |
| Checkpoint | `design-review` for a new dependency, a major version upgrade, or a platform/runtime dependency; record the decision per §11.3 |
| Specific steps | *mandatory:* search for already approved capabilities first (§11.2); hard gates on security, license and data policy (§11.3); lock the resolved version; update SBOM and inventory. *conditional:* check consumers when the package is shared |
| Recovery | `reversible` via lockfile; a dependency already published externally is `irreversible` |
| Output | Dependency decision record, compatibility evidence |

## D.5. API Contract Change

| Component | Content |
|---|---|
| Trigger | Request to change a public interface or an inter-service interface |
| Sub-state | `CONTEXT_READY → CONSUMERS_IDENTIFIED` · `PLANNED → CONTRACT_PROPOSED`, `CONTRACT_APPROVED` · `EXECUTING → PROVIDER_IMPLEMENTING` · `VERIFYING → COMPATIBILITY_VERIFIED` |
| Checkpoint | `specification-review` and `design-review` are mandatory (contract owner, with consumer consultation); `plan-review` when breaking (tier High); `release-readiness` |
| Specific steps | *mandatory:* identify every consumer from the catalog and contract tests; contract diff; provider-side consumer-driven contract tests; backward-compatibility plan or time-boxed deprecation. *conditional:* coordinated changes across multiple repositories |
| Knowledge | A new contract version triggers reconciliation for every artifact `basedOn` the old contract (§5.10) |
| Recovery | Additive changes are `reversible`; a breaking change already used by consumers is `compensable` (keep the old version in parallel) or `irreversible` |
| Output | Contract diff, consumer/provider evidence, migration notes for consumers |

## D.6. Database Migration

| Component | Content |
|---|---|
| Trigger | Schema change, backfill, or data move requested by another task |
| Sub-state | `PLANNED → MIGRATION_DESIGNED` · `VERIFYING → MIGRATION_VALIDATED` · `RELEASING → MIGRATING` · `OBSERVING → POST_MIGRATION_VERIFICATION` |
| Checkpoint | `design-review`, `plan-review` and `release-readiness` are mandatory |
| Specific steps | *mandatory:* design using expand/contract so that the running version stays compatible; dry-run on a production-like snapshot; analyze locks, run time and storage; verify backup and the recovery path before running; post-migration verification (record counts, constraints, sample queries). *conditional:* batched backfill with checkpoints and rate limiting |
| Recovery | The expand step is `compensable` (a compensating migration declared in advance). Steps that delete or transform data destructively are `irreversible`: split into a separate step, run after an explicit human authorization, no automatic retry (§6.14). Each backfill batch is idempotent |
| Output | Migration plan, dry-run evidence, run record, post-migration evidence |

## D.7. Incident Response

| Component | Content |
|---|---|
| Trigger | Production alert, user report, security event, production signal indicating an ongoing incident (§12.4) |
| Sub-state | `TRIAGED → TRIAGING` · `EXECUTING → MITIGATING` · `OBSERVING → STABILIZING` · `INCIDENT_ANALYSIS → ROOT_CAUSE_ANALYSIS` |
| Checkpoint | `release-readiness` uses the incident variant: the incident commander fills the `incident-commander` slot (§6.12). Verification of the mitigation may be `deferred` via break-glass (§12.7) and completed after recovery |
| Specific steps | *mandatory:* classify severity and scope; mitigate with actions that are never blocked (flag-off, rollback to a verified revision, unpublish, §12.6); correlate signals; root cause analysis answering "which layer should have caught this defect" (§12.4); create a forward-fix task (Bug Fix) and preventive actions. *conditional:* reconciliation when the incident reveals incorrect knowledge |
| Agent role | Propose only; execute mitigation when an engineer explicitly instructs it (§12.5) |
| Recovery | Rollback and data recovery require human authorization (§18.1) |
| Output | Incident report, forward-fix task, additional tests and eval cases |

## D.8. Release / Deployment

| Component | Content |
|---|---|
| Trigger | Release train, SDK/package publish, or the tail of a deployable task (§12.1) |
| Sub-state | `RELEASING → CANDIDATE_VERIFIED`, `STAGED`, `CANARY`, `PROMOTING` · `OBSERVING → HEALTH_CONFIRMED` |
| Checkpoint | `release-readiness` is mandatory |
| Specific steps | *mandatory:* verify the commit and the immutable artifact with provenance (§12.3); deploy to an internal/staging environment and smoke test; staged rollout with confirmed thresholds; decide to continue, pause or roll back based on evidence. *conditional:* install matrix from the artifact and consumer-side compatibility for SDKs |
| Recovery | Redeploying the previous artifact is `reversible`; a publicly published artifact is `irreversible` and can only be superseded by a new version |
| Output | Release manifest, system health evidence |

## D.9. Other workflows

| Workflow | Core stages | Required output |
|---|---|---|
| Generic | Normative definition in Appendix C.5: the §6.6 skeleton, every non-omittable step is `mandatory`, no sub-states; trust does not exceed T0 | Outputs in Appendix C.5 item 6 |
| Investigation | Understand → Search → Trace → Hypothesize → Confirm | Findings and evidence, no code changes unless requested |
| Codebase Gardening | Scan → Rank → Propose → Fix bounded items → Verify | Small maintenance PRs, technical debt trends, and a code freshness report (deprecated APIs, outdated polyfills and dependencies, conformance with current platform standards) [R38] |
| Lookback | Select fixes released within the window → Match signal fingerprints and rerun reproduction tests → Classify: not recurring / recurring → Reopen root cause analysis on recurrence (§12.4) | Recurrence report by task class; escaped defects recorded for trust (§9.4) and eval (§9.2) |
| Knowledge Reconciliation | Receive trigger → Find artifacts via the index → Analyze impact → Decide per authority → Synchronize → Revalidate (§5.10) | `ReconciliationRecord`, artifacts updated or transitioned, work items for owners |
| SDK Release | Public API diff → Runtime matrix → Package build/install → Examples → Compatibility → Publish approval | Package artifact and compatibility report |
| AI Application Change | Select dataset → Change prompt/model/tool → Offline eval → Safety checks → Shadow/limited rollout | Eval report, traces, cost/latency measurements |
| Performance Hill-climb | Baseline → Hypothesis → Change → Measure over repeated runs → Explain the numbers → Keep or revert | Benchmark report with notes on variance, limiting factors and errors [R06] |
| PR Babysit | Monitor CI and review → Separate flakes from real failures → Fix within limits → Resolve findings | PR reaches a mergeable state or is escalated [R06] |
| Session Pickup / Pause Safely | Checkpoint state → Record next step → Resume from the task record (§6.14) | Resumable task record; no lost work [R06] |
