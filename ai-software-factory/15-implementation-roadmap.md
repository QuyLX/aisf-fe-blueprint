# 15. Implementation roadmap

The roadmap is deliberately execution-oriented. Each phase has deliverables, Exit criteria and explicit scope limits. A phase may be shortened or merged if the pilot already has the required capability.

**Sizing.** Each phase must have a target duration, staffing, budget and stop criteria recorded in the Pilot Charter (Appendix H) before it starts.

**Execution order.** Fix the inner loop before automating the outer loop: (1) get the codebase right and encode recurring corrections as constraints; (2) strengthen verification; (3) only then automate outer-loop workflows such as unattended PRs, scheduled fuzzing or auto-merge. Automating an inner loop that produces poor code only produces more poor code [R05]. Phases 1–3 are therefore prerequisites for Phase 4, not parallel tracks (P24).

**Showcase.** The Standard is validated by fully implementing one domain profile: the AISF Frontend Profile. Its roadmap cuts **showcase slices** through all three planes (§3.1) instead of completing one plane at a time. Slice 0 is the Minimum Viable Harness of this section applied to one frontend surface, and every slice Exits on runnable scenarios that prove specific core requirements (§20.6). The phases below remain the common path for every later profile.

## 15.0. Minimum Viable Harness

The normative definition of pilot mode, including the permitted loosenings and what is never loosened, is in §20.8; this section is the roadmap for implementing it. Non-frontend teams without a domain profile use the Core Profile (§20.7) for the pilot.

The Standard describes the target system; the pilot should start much smaller, consistent with "prove it on one repository first". Do not implement a workflow engine, knowledge graph, registry service, dashboards and specialized agents at the same time. Leverage what the Standard and the repository already have, and first prove **one end-to-end flow**:

```text
one repository
 → Bug Fix workflow (Task Workflow Specification ACTIVE, Appendix C)
 → review checkpoints and approval (requirement, implementation; plan when tier High)
 → independent verification via the verification adapter
 → evidence bound to the commit
 → knowledge reconciliation when the change touches a described artifact
```

Every other component is added only once this flow runs on real issues. Target for the first 6–8 weeks (confirmed in the Pilot Charter):

| Included | Deferred until the pilot shows a need |
|---|---|
| `AGENTS.md`, `REVIEW.md`, PR template (Testing / Risk / How to revert) | Metadata manifest and JSON Schema (§4) |
| CODEOWNERS and branch protection, with protected control files (§7.13) | Evidence bundle JSON schema; use CI artifacts + the PR template |
| Integrity sensor (cheap diff scanning; mutation testing later), running at `warn` in the pilot except `integrity.skip-focus` and `integrity.control-file-change` at `blocking` (§7.13) | Custom multi-agent AI review pipeline; use an off-the-shelf AI reviewer in `shadow`/`advisory` mode |
| 3–5 high-value rules with fixtures | The full catalog of 10–15 rules |
| Separation-of-duties check with authenticated requester identity (§9.10) | Policy decision service, trust evaluator |
| Required checks from a central definition that PRs cannot modify (§7.15, items 1–2); canonical vocabulary and schemas (Appendix J, K) | Attestation signing (§7.15, items 3–4), unless the profile requires it earlier |
| Factory degraded mode runbook and break-glass procedure (§12.7) | Automatic switch to degraded mode |
| One workflow: Bug Fix in the minimal pilot version (Appendix C.4): steps, checkpoints, reproduction test written by the verifier; records derived from platform events | Sub-states, handoff records and recovery records as artifacts; other workflows (kept at outline, Appendix D); a dedicated workflow engine |
| Checkpoints and knowledge impact summarized in the PR template, derived from events (Appendix G); the PR's "Review findings" and "Knowledge impact" sections are the compact form of the Learning step (§6.6 step 16) | Dedicated checkpoint store and policy service |
| Deterministic path → tier mapping table (`riskMap` in `factory.yaml`), owned by owners like CODEOWNERS; paths not in the table are High; authors do not choose their own tier | Automatic tier classifier |
| Lifecycle metadata (frontmatter) for the 3–5 knowledge artifacts the pilot actually uses (e.g., `AGENTS.md`, `REVIEW.md`, architecture documentation for the pilot area); `knowledge.impact` check in `warn` mode based on `describes` globs; reconciliation decisions recorded in the PR (§5.8–5.10) | Knowledge graph, graph database, central Knowledge Registry; scheduled reconciliation; `knowledge.impact` at `blocking` |
| Verification adapter for the pilot repository's toolchain (§7.16) | Adapters for other toolchains |
| Baseline measurement, including human attention minutes per work item (§13.6), and minimum workflow metrics (success, retries, approval wait time, escalations) | Dashboards, central catalog, circuit breaker automation (replaced by weekly manual review), specialized agents |

**Effort estimate.** With 3–5 engineers, building the harness above takes most of the 6–8 weeks, so the end-to-end flow typically runs on real issues only in the last few weeks. The Pilot Charter needs to allow enough run time to obtain a measurable sample (*suggested* ≥ 15–20 real bug fixes) before the Exit assessment; if that is not enough, extend the pilot rather than drawing conclusions from a small sample (§13.4).

**Exit:**

- The end-to-end flow above runs on ≥ 15–20 real bug fixes (*suggested*), compared against the baseline for lead time, human attention minutes, people touching each task and escaped defects (§13.2, §13.6).
- At least one change touches a described artifact and is reconciled.
- No gate is bypassed.
- A **pilot conformance claim** (§20.5, §20.8 item 3) at level L1 plus the following requirements: CORE-KNW-01, CORE-KNW-02, CORE-KNW-03, CORE-KNW-09, CORE-WRK-02, CORE-VER-03, CORE-VER-04, CORE-VER-05, CORE-VER-07, CORE-VER-12, CORE-VER-14, CORE-REV-01, CORE-GOV-05, CORE-GOV-06, CORE-GOV-09, CORE-SEC-01, CORE-SEC-03, CORE-OPS-05, CORE-MET-01 (baseline), CORE-PRF-01. A full L2 claim is not a pilot Exit condition; it is the goal of Phases 1–2.
- A decision on what to add next, based on workflow metrics (§13.7).

## Phase 0 – Baseline and pilot selection

**Goal:** select a representative repository and measure the current engineering workflow.

Deliverables:

- Repository selection decision.
- Repository/product/domain map.
- Summary of current architecture and ownership.
- Dependency inventory.
- Current CI/test inventory.
- Baseline task groups and measurements.
- The most frequently recurring defects/review findings.
- Data/security constraints.

Exit criteria:

- The repository owner and the Factory pilot owner are named.
- At least one repeatable task class is selected.
- Baseline metrics can be collected consistently.
- The repository has a working build/test path or a clear plan to establish one.

Do not build a central platform in this phase.

## Phase 1 – Repository harness foundation

**Goal:** make the repository legible, constrained and safe for bounded agent work.

Deliverables:

- A concise `AGENTS.md` entry point.
- Documentation of the current architecture and module boundaries.
- Rule catalog: 3–5 high-value rules in the Minimum Viable Harness, growing gradually to 10–15 only when recurrence data supports it.
- Rule fixtures and a false positive handling policy.
- Repository metadata manifest and schema check (may be deferred; see §15.0).
- Dependency inventory and basic graph checks.
- Standard build, lint, type and test commands.
- Agent security baseline and approved tool list.
- Evidence template for PRs.
- PR template with mandatory Testing / Risk / How-to-revert sections (Appendix G).
- `REVIEW.md` for the pilot repository, written by its code owners (Appendix F).
- Verification integrity sensor and protected harness paths (§7.13).
- CODEOWNERS and branch protection for paths that always require humans, including CI workflow files.

Exit criteria:

- Agents can find the right local context without loading the entire knowledge base.
- Mandatory architecture constraints are checked mechanically.
- Existing checks run reproducibly in CI.
- Rule exceptions have an owner, a rationale and an expiry/review date.
- Agent workspaces and permissions are bounded.

## Phase 2 – Verification environment

**Goal:** enable agents to prove behavior, not merely report that code was changed.

Deliverables:

- Verification CLI/runner.
- Standard schema for verification results.
- Execution in a local sandbox/worktree.
- Runtime environment startup and health checks.
- Product-specific verification profiles (first version).
- Playwright/browser driver for a frontend pilot, if applicable.
- Evidence storage and artifact references.
- Independent scenario tests for critical acceptance criteria.
- Flaky check quarantine policy (§7.14).
- Merge queue enabled for the pilot repository.

Exit criteria:

- A representative task can be verified from a clean workspace.
- Required checks are mapped to task/AC IDs.
- Evidence is bound to the tested commit.
- Failure feedback is actionable and can feed into the repair loop.

## Phase 3 – Playbooks, skills and evals

**Goal:** turn recurring engineering work into reusable workflows and measurable agent capability.

Deliverables:

- Promote workflows from outline (Appendix D) to `ACTIVE` according to measured need: typically Investigation, Feature Development, Refactoring, Dependency Change, Release / Deployment.
- Reusable skills for repository exploration, testing, verification and evidence collection.
- Permission profiles by role/tool.
- An initial eval dataset of representative tasks.
- Trace collection and structured scoring.
- Versions and owners for skills/playbooks.
- Trial an AI review pipeline on one repository (§7.9–7.12), tuned via `REVIEW.md` over a few weeks before expanding.
- A reviewer eval set derived from incidents/reverts and a set of known-clean PRs.

Exit criteria:

- Each `ACTIVE` workflow has a full spec per §6.7.1 (trigger, inputs, steps, sub-states, checkpoints, handoffs, recovery), an owner and evals.
- The eval set has baseline results and human-calibrated scoring for subjective criteria.
- A change to a skill/playbook can trigger eval regression.

## Phase 4 – Trusted agentic delivery

**Goal:** safely automate a narrow set of low- and medium-risk workflows.

Deliverables:

- Task classifier and policy evaluation.
- Risk × Trust × Reversibility decision model.
- Independent review for selected workflows.
- Deduplication and disposition for review findings.
- Repair loop/time/cost limits.
- Automated PR creation and updates.
- A restricted auto-merge policy for a narrow low-risk group, only when evidence supports it.
- Tier classifier with an override check; commit-bound approval (§9.6–9.8).
- An operational autonomy circuit breaker, and AI review in `blocking` mode for that area, before any zero-human merge (§9.5, §9.9).

Exit criteria:

- No mandatory gate is bypassed.
- Agent trust is measured per task class.
- The auto-merge scope is explicit, reversible where possible, and auditable.
- Reliability guardrails remain within pilot thresholds.
- The circuit breaker has been exercised at least once by simulation.

## Phase 5 – Federation and multi-repository intelligence

**Goal:** expand from one repository to the product ecosystem and cross-repository changes.

Deliverables:

- Product/repository/service/SDK catalog.
- API and package consumer relationships.
- Cross-repository impact analysis.
- Multi-repo plans and approval gates.
- Workflows coordinating contracts and implementations.
- Distribution of shared skills/playbooks/policies.
- Centralized evidence aggregation.
- A knowledge dependency index aggregated across repositories by the Control Plane (§5.9).
- Verification adapters for new repositories' toolchains (§7.16).

Exit criteria:

- A sample cross-repository change can identify the affected consumers.
- The harness stops to await human confirmation when blast radius or compatibility risk requires it.
- Coordinated changes are verified against consumer/provider contracts and the relevant system tests.

Do not introduce a graph database unless the catalog/query workload proves it necessary.

## Phase 6 – Production intelligence and Factory optimization

**Goal:** close the feedback loops from production and optimize cost, quality and capacity.

Deliverables:

- Release provenance.
- Per-change monitoring for selected services.
- Workflows turning incidents into regression tests/rules/evals.
- A closed loop from production signals to Bug Fix, with deduplication, attention-budget limits, and the Lookback workflow (§12.4).
- Codebase gardening, documentation gardening and scheduled Knowledge Reconciliation workflows (§5.10).
- Metadata/architecture drift checks.
- Factory economics, capacity and workflow metrics dashboards (§13.7).
- Scheduled agent/skill/playbook evals.
- Periodic improvement reviews.
- Change Shepherd for selected services (§12.2.1); performance incident triage and an incident assistant in suggest-only mode (§12.5).
- A release-time exploratory fuzzing process that operates the application as a user and opens bug fix PRs; those PRs are merged only within the scope of the §9.5–9.9 policy [R05].

Exit criteria:

- A production signal can be linked to the change/release that caused it.
- Significant incidents produce owned preventive actions.
- Agent and rule regressions are detected before broad rollout.
- Factory cost and quality trends are measurable.
