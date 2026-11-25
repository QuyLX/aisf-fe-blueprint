# Appendix H – Pilot Charter template

| Field | Content |
|---|---|
| Pilot repository and owner | |
| Factory pilot owner | |
| Task classes in scope (1–2) | e.g. bug fix, small feature |
| Workflow specs used in the pilot | Bug Fix `ACTIVE` (Appendix C); other workflows in outline |
| Managed knowledge artifacts | List of artifacts with lifecycle metadata, owner and `describes` (§5.8) |
| Explicitly out of scope | e.g. multi-repository changes, zero-human merge, production agents |
| Metadata entities used | Keep to a minimum, e.g. Repository, Domain, Feature, Rule, Evidence |
| Task state representation | PR state + labels (including sub-states) + CI check runs; checkpoints and handoffs recorded in the issue/PR (no separate task store) |
| Evidence representation | CI artifacts + PR template; use custom schemas only when aggregation requires them |
| Baseline period and metrics | |
| Phases in scope, target duration, staffing, budget | |
| Success criteria | Agreed after the baseline is available |
| Stop / rollback criteria | e.g. a guardrail degrades for N consecutive weeks |
| Code owners and backups per area | At least two qualified owners or one named backup (§9.10); choose a pilot area with at least three people qualified to review, or designate a fallback reviewer from another team |
| Git/CI platform and agent runtime | Decisions §18 items 1–3, and how the controls are implemented on the chosen platform (§14.4) |
| Cost ceiling | AI + CI cost ceiling for the whole pilot and the default per task (§6.4) |
| Minimum measurement sample | Number of real bug fixes to run before evaluating Exit (*suggested* ≥ 15–20, §15.0) |
| Review cadence | Weekly pilot review (§17.2) |
