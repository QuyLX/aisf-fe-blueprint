# 16. Initial pilot backlog

P0 is the Minimum Viable Harness (§15.0), implementing the pilot mode of §20.8.

| Priority | Deliverable | Acceptance evidence |
|---|---|---|
| P0 | Pilot Charter and baseline report | Owner, task class, duration, staffing, budget, stop criteria, baseline metrics (Appendix H) |
| P0 | Canonical vocabulary and schemas in the policy repository | Registry CI rejects undefined terms and schema-invalid evidence (Appendix J, K) |
| P0 | Central required check definitions | A PR that modifies CI or an adapter cannot change its own verification result (§7.15) |
| P0 | Degraded mode and break-glass | Runbook rehearsed once by simulation; break-glass requires two people and is audited (§12.7) |
| P0 | `AGENTS.md`, `REVIEW.md`, PR template | Concise entry point; review guidance written by the owner; template used on every pilot PR |
| P0 | CODEOWNERS, branch protection, control file protection, backup owners | A planted change to `REVIEW.md`, `catalog/**` or the `scripts` of `package.json` cannot merge without an independent code owner; every area has a backup owner |
| P0 | Path → tier mapping table | `riskMap` in `factory.yaml` is an owned control file; undeclared paths resolve to High; a PR that lowers its own tier cannot merge without an independent code owner |
| P0 | Separation-of-duties check | Rejects: a requester approving their own agent's Medium PR; an out-of-team requester self-approving a Low tier; an approval that only matches via a forged `Requested-by:` trailer |
| P0 | Integrity sensor (diff scanning) | Catches planted cases: test deletion, skip markers, suppressions, edited expected values and threshold changes |
| P0 | 3–5 rules with fixtures | Each rule has positive/negative fixtures and an owner |
| P0 | CI verification baseline | Build, lint, type, test and basic architecture/security checks run reproducibly |
| P0 | Bug Fix workflow spec `ACTIVE` (Appendix C) | Used on real issues; state derived from labels and events (sub-states derived or skipped in the pilot, §20.8); bug reproduction evidence required; implementer → verifier handoff excludes the implementer's context |
| P0 | Checkpoint records in the issue/PR | `requirement-review`, `implementation-review` and `independent-verification` are recorded separately for every pilot task; review, approval and verification do not substitute for one another |
| P0 | Knowledge artifact metadata and `knowledge.impact` | A change touching a path listed in `describes` lists the affected artifacts and requires a reconciliation decision; a planted scenario (code differs from spec) shows the agent recording a knowledge conflict instead of editing the spec |
| P0 | Verification adapter for the pilot toolchain | Required checks run through the adapter; timeouts and tool failures yield `ERROR`, not `FAIL` (§7.16) |
| P0 | Agent permission/security policy | Scoped tools, sandbox, secret policy, audit |
| P1 | Off-the-shelf AI review in shadow → advisory mode | Collect disposition data; agree on promotion thresholds |
| P1 | Merge queue and flaky registry | Main is not broken by combined PRs; only pre-existing flakes are quarantined, by the check owner |
| P1 | Runtime/browser verification | One critical workflow is exercised and evidence is collected |
| P1 | Initial eval dataset | Representative cases, cases derived from incidents/reverts, known-clean PRs, baseline |
| P1 | Mutation testing on changed lines | Runs on Medium/High PRs within the CI budget |
| P1 | Basis change handling | A planted scenario (a spec in the basis is superseded while the task is `EXECUTING`) blocks the next gate until a decision is made (§6.5) |
| P1 | Minimum workflow metrics | Success, retries, approval wait time and escalations recorded per workflow version (§13.7) |
| P2 | Verification CLI, evidence manifest, metadata schema | Introduce only when aggregation or a second repository requires it |
| P2 | Feature Development and Refactoring workflows to `ACTIVE`; codebase gardening/documentation gardening | Full spec per §6.7.1; small proposal PRs with recurrence metrics |
| P2 | Trust evaluation per §9.4 | Stratified task set, critical scenarios and holdout for the pilot task class |
| P2 | Human attention budget | Review capacity declared for the pilot area; automation-generated tasks are queued when capacity is exceeded; attention minutes per work item metric reported (§13.6) |
| P2 | Production signals → Bug Fix, and Lookback | One signal source connected, with fingerprint-based deduplication; Lookback runs periodically and reports the recurrence rate (§12.4) |
| P2 | Trust/risk policy evaluator; zero-human path for one opted-in area | Explainable decisions; the §9.9 breaker is operational and AI review has been promoted to `blocking` in that area first |
| P2 | Change Shepherd for one service | Per-change signals and health reports during staged rollout |
| P3 | Central catalog and cross-repo impact analysis | Only after a second repository has been onboarded |
| P3 | Custom multi-agent review pipeline, specialized agents | Only when the measured limits of the off-the-shelf reviewer justify it |
| P3 | Knowledge graph service, dedicated workflow engine | Only when the thresholds in §5.9 are reached or when the issue/PR task runner is measured to be insufficient |
