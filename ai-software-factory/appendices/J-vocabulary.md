# Appendix J – Canonical vocabulary

Values are defined in [`aisf-vocabulary.yaml`](../aisf-vocabulary.yaml).

## J.1. Governance

| Term | Values | Defined in |
|---|---|---|
| Risk tier | `Low`, `Medium`, `High` (a profile MAY define sub-tiers, each mapping to exactly one tier) | §9.6 |
| Reversibility | `HIGH`, `MEDIUM`, `LOW` | §9.5 |
| Trust level | `T0`, `T1`, `T2`, per task class and agent configuration; task class `generic` does not exceed `T0` | §9.4 |
| Trust profile status | `CURRENT`, `RE-EVALUATION_REQUIRED` | §9.4, §9.9 |
| `allowedAutonomy` | `propose-only` < `implement-and-request-review` < `verify-and-open-pr` < `merge-on-eligible-path` | §9.4 |
| Protected categories and floor tiers | `auth-and-secrets` High, `billing-and-payments` High, `legal-and-privacy` High, `destructive-or-regulated-data` High, `public-api-breaking` High, `public-api-additive` Medium, `ci-release-and-branch-protection` Medium (loosening: High), `production-operations` High (includes executing releases to production and data plane operations), `control-file` Medium (loosening: High) | §9.6 |
| Trust evaluation components | `representativeness`, `critical-scenarios`, `holdout`, `regression`, `confidence` | §9.4 |
| Eligibility | Ownership trust AND execution trust (T2) | §9.5 |
| Autonomy level | `A0`, `A1`, `A2`, `A3`, defined as sets of approval slots | §9.5, §9.10 |
| Approval slot | `requester-signoff`, `independent`, `code-owner`, `control-file-owner`, `security-owner`, `business-owner`, `plan-approval`, `product-owner`, `architect`, `service-owner`, `incident-commander`, `domain-owner` (scope defined by the profile) | §6.12, §9.10, App. K |
| Requester source | `ci-dispatch`, `issue-command`, `cli-login`, `automation-owner`, `unknown` | §9.10 |

## J.2. Verification and evidence

| Term | Values | Defined in |
|---|---|---|
| Check status | `PASS`, `FAIL`, `ERROR`, `NOT_ASSESSED` | §7.5 |
| Disposition | `not-applicable`, `assessed-elsewhere`, `baselined`, `waived`, `deferred` | §7.5 |
| Gate | `merge`, `publish`, `post-release` | §7.4 |
| Check lifecycle | `planned`, `shadow`, `active`, `deprecated` | §8.7 |
| Check requirement level in a profile | `required`, `advisory` | §8.7 |
| Check basis | `standard`, `contract`, `security`, `privacy`, `legal`, `policy`, `record` | §8.7 |
| Dependency policy status | `APPROVED`, `CONDITIONAL`, `DEPRECATED`, `PROHIBITED` | §11.4 |
| Vulnerability severity | `critical` (*suggested* remediation within 7 days), `high` (30 days), `medium` (90 days), `low` (next planned upgrade); profiles may only shorten these | §11.7 |
| License policy | `allow`, `deny`, `review`; licenses not on the list are `review` | §11.8 |
| Rule lifecycle | `PROPOSED`, `DRAFT_WITH_FIXTURES`, `OBSERVE`, `WARN`, `BLOCKING`, `MONITOR`, `REVISE`, `DEPRECATE`, `RETIRE` | §8.3 |
| Author response to a finding | `fixed`, `wrong`, `intentional`, `didnt-matter` | §7.12 |
| Rule category | Core: `CODE`, `TYPE`, `ARCH`, `API`, `SEC`, `DEP`, `TEST`, `UX`, `FLAG`, `PERM`, `STATE`, `PERF`, `OPS`, `AI`, `DOC`; profiles register more (Frontend 1.1: `NEXT`, `CTX`, `DS`, `I18N`, `CFG`) | §8.1 |
| Enforcement level | `observe`, `warn`, `blocking` | §7.5, §8.2 |
| Check severity | `BLOCKER`, `WARNING`, `INFO` (derived from enforcement) | §7.5 |
| Check ownership | `local`, `cross-team-contract` | §8.7 |
| Profile layer | `core`, `domain`, `organization`, `repository`, `surface`, `task-class`, `risk`, `task` | §7.3 |
| Profile conflict resolution | `stricter`, `union`, `decision`, `rejected` | §7.3 |
| Verification executor | `ci`, `agent-sandbox`, `local` (`local` never satisfies a required check) | §7.16 |
| AI review mode / depth | `shadow`, `advisory`, `blocking` / `quick`, `standard`, `deep` | §7.9 |
| Finding tag | `Important`, `Nit`, `Pre-existing` | §7.9 |

## J.3. Task and workflow

| Term | Values | Defined in |
|---|---|---|
| Work item | One accepted outcome, represented by a Task contract; may span multiple PRs or multiple publishes | §6.4, §13.3 |
| Task state | `REQUESTED`, `TRIAGED`, `NEEDS_CLARIFICATION`, `REJECTED`, `CONTEXT_READY`, `PLANNED`, `AWAITING_PLAN_APPROVAL`, `EXECUTING`, `VERIFYING`, `REPAIRING`, `REVIEWING`, `AWAITING_MERGE_APPROVAL`, `IN_MERGE_QUEUE`, `MERGED`, `RELEASING`, `PAUSED`, `OBSERVING`, `ROLLED_BACK`, `REVERTED_ON_MAIN`, `INCIDENT_ANALYSIS`, `FORWARD_FIX`, `COMPLETED`, `BLOCKED`, `ESCALATED`, `CANCELLED` | §6.5 |
| Task state transitions | Full list in `taskState.transitions`; `CONTEXT_READY`, `PLANNED` and `AWAITING_PLAN_APPROVAL` may transition to `NEEDS_CLARIFICATION`; `PLANNED` and `AWAITING_PLAN_APPROVAL` may transition to `REJECTED`; terminal states: `COMPLETED`, `REJECTED`, `CANCELLED`, `FORWARD_FIX` | §6.5 |
| Task sub-state | Registered by the workflow spec; each sub-state belongs to exactly one parent state; sub-states of the core workflows are listed in Appendix D | §6.5 |
| Change channel state | `REQUESTED`, `VERIFYING`, `AWAITING_APPROVAL`, `PUBLISHED`, `OBSERVING`, `COMPLETED`, `ROLLED_BACK`, `BLOCKED` | §12.6 |
| Change channel | `code`, `content`, `config`, `data` | §12.6 |
| Core workflows | `generic`, `bug-fix`, `feature-development`, `refactoring`, `dependency-change`, `api-contract-change`, `database-migration`, `incident-response`, `release-deployment`, `investigation`, `codebase-gardening`, `lookback`, `knowledge-reconciliation`, `sdk-release`, `ai-application-change`, `performance-hill-climb`, `pr-babysit`, `session-pickup` | §6.7.2, App. D |
| Step kind | `mandatory`, `conditional`, `optional` | §6.7.3 |
| Step side effect | `none`, `reversible`, `compensable`, `irreversible` | §6.14 |
| Recovery action | `retry`, `resume`, `rollback`, `compensate`, `escalate` | §6.14 |
| Review checkpoint | `requirement-review`, `specification-review`, `design-review`, `plan-review`, `implementation-review`, `independent-verification`, `release-readiness` | §6.12 |
| Checkpoint outcome | `passed`, `failed`, `not-applicable` | §6.12 |
| Handoff status | `accepted`, `rejected` | §6.13 |
| Decision on basis change | `continue`, `replan`, `reapprove`, `stop` | §6.5 |
| Loop budget | `verifyRepairIterations` (≤ 3), `reviewRounds` (≤ 3), `mergeQueueAttempts` (≤ 2) | §6.4 |
| Cost ceiling | `cost` in `usd` or `tokens` | §6.4 |
| Acceptance criterion | `oracle`: `existing-test`, `new-independent-test`, `contract`, `runtime-check`, `human-judgment`; `critical`: `true`, `false` | §6.4.1 |
| Assumption status | `confirmed`, `unconfirmed` | §6.4.1 |

## J.4. Knowledge

| Term | Values | Defined in |
|---|---|---|
| Knowledge artifact status | `DRAFT`, `PROPOSED`, `ACTIVE`, `NEEDS_REVALIDATION`, `SUPERSEDED`, `DEPRECATED`, `ARCHIVED` | §5.8 |
| Knowledge artifact kind | `spec`, `design`, `adr`, `proposal`, `living-doc`, `runbook`, `agent-guide`, `skill`, `playbook` | §5.8 |
| Knowledge artifact authority | `intent`, `decision`, `implementation-description`, `procedure`, `guidance` | §5.8 |
| Knowledge relations | `describes`, `basedOn`, `generatedFrom`, `verifiedBy`, `implements`, `supersedes` | §4.3, §5.9 |
| Reconciliation decision | `keep`, `update`, `supersede`, `deprecate`, `review`; status `proposed`, `decided` | §5.10 |
| Knowledge conflict resolution outcome | `implementation-defect`, `intent-changed`, `artifact-corrected`, `superseded`, `no-conflict` | §4.7 |

## J.5. Conformance and ID conventions

| Term | Values | Defined in |
|---|---|---|
| Conformance level | `L1` Described, `L2` Guarded, `L3` Agent-ready, `L4` Autonomy-eligible | §20.4 |
| Rule ID | `<SCOPE>-<CATEGORY>-<NNN>`, e.g. `ARCH-001`, `FE-API-001` | §7.5, §8.1 |
| Check ID | `<domain>.<name>`, e.g. `integrity.skip-focus`, `knowledge.impact` | §7.5, §8.7 |
| Core requirement ID | `CORE-<AREA>-<NN>`, e.g. `CORE-GOV-05`; areas: `KNW`, `WRK`, `VER`, `REV`, `GOV`, `SEC`, `DEP`, `OPS`, `MET`, `PRF` | §20.6 |
| Core Profile | `aisf-core`, same version as the Standard (e.g. `aisf-core@4.0.0`); at most L2 | §20.7 |
| Claim mode | `pilot`, `standard` | §20.5, §20.8 |
| Checkpoint record, handoff, reconciliation, conflict IDs | `CP-…`, `HO-…`, `REC-…`, `KC-…` (suggested) | App. K |

## J.6. Security and data

| Term | Values | Defined in |
|---|---|---|
| Data class (`dataClass`) | `public` < `internal` < `confidential` < `restricted`; unknown is `restricted`; a task, trace or evidence bundle takes the highest class of all its inputs | §10.6 |
| Retention class | `ephemeral` (*suggested* 30 days), `standard` (180 days), `audit` (2 years, no shorter than the term of any claim relying on it), `legal-hold` (until released) | §10.7 |
