[Contents](README.md) · [← 14. Organization & operations](14-organization-and-operations.md) · [16. Current state: problems and data, not constraints →](16-current-state.md)

<a id="s15"></a>

# 15. Roadmap

This roadmap builds the **platform + Factory** and the first **vertical slice** of each product. Building out each product's full feature set is the product roadmap, which runs on this platform.

**Showcase slices.** The roadmap is cut according to the showcase plan ([§20](20-showcase.md#s20)):

- **Slice 0** is the **AISF pilot** in the first 6–8 weeks of Phase 1a: the Standard's Minimum Viable Harness (Standard §15.0) on one Console surface. The target claim is a pilot claim: L1 plus the pilot's list of CORE-IDs ([§20.2](20-showcase.md#s20-2)), not L2.
- **Slice 1** starts with a short follow-up in Phase 1a (attestation, merge queue, mandatory pipeline on every repo), reaches L2 for generator-created repos at the end of Phase 1b, and closes with Phase 2: L3 and cross-repo changes.
- **Slice 2** spans Phases 4–6: content channel, circuit breaker, degraded mode, and the first A0.

Each phase exit below includes the scenarios of the corresponding slice and the Testing & Quality Engineering and Documentation & Knowledge capability milestones in [§15.2](#s15-2). Testing is not just the checks in the pipeline, and documentation is not just the Docs Portal: both are built up progressively from Phase 0.

## Phase 0 — Decisions & spikes (4–6 weeks)

- ADRs: stack ([§4.1](04-technical-foundation.md#s4-1)), repo topology and version skew policy ([§3.3](03-target-architecture.md#s3-3), [§4.8](04-technical-foundation.md#s4-8)), **one** CI platform meeting the [§8.7](08-verification.md#s8-7) checklist — including applying the mandatory pipeline to **every repo** in the organization (spike on the exact license tier, including signing), headless CMS (build vs buy), ticket backend (build vs buy), chat streaming protocol, IdP and session model ([§3.5](03-target-architecture.md#s3-5)).
- **Contract-first commitment** from backend teams: OpenAPI for every service the frontend calls, Pact provider verification, broker, test tenant + mail sink on staging. Only the first service used in Slice 0 MUST have a contract before Phase 1a; services that have not published OpenAPI use the interim `consumer-authored` path ([§4.2](04-technical-foundation.md#s4-2)).
- Spikes: one end-to-end Console screen on Next.js (`@platform/server` + client generated from OpenAPI + MSW + Playwright + attestation), plus a topology spike ([§3.3](03-target-architecture.md#s3-3)): for polyrepo, one DS prerelease → consumer verification in another repo; for monorepo, affected build, remote cache and merge queue on a simulated repo the size of the full portfolio.
- Finalize the Wave 1 check budget, trust thresholds, command time budgets, cost estimate and **staffing per phase** ([§15.1](#s15-1)).
- **Calibration set** for classifiers and eval from the history of existing systems ([§16.2](16-current-state.md#s16-2)).
- **Entry policy:** after the Phase 1b exit, every new frontend app/product MUST be created with a generator and run the mandatory pipeline.
- **Platform ↔ vertical agreement** ([§14.1](14-organization-and-operations.md#s14-1)): list of vertical owners and champions, support SLA, version window.

**Exit:** the decisions blocking Phase 1a ([§17](17-decisions-to-make.md#s17)) have `ACTIVE` ADRs; the remaining decisions have owners and deadlines; the spikes run; the backend commitment has an owner.

## Phase 1a — Factory spine (~1 quarter)

**Starts with Showcase Slice 0** ([§20.2](20-showcase.md#s20-2)): the AISF pilot following the Minimum Viable Harness (Standard §15.0) on one Console surface, for 6–8 weeks. Slice 0 only uses what exists when Phase 0 ends (Next.js spike, client and mock generated from the first service's OpenAPI); it does not depend on the app shell, `@platform/server` or the DS from Phase 1b, nor does it need the `release` channel from Phase 2.

Two parallel tracks:
- **Factory track, Slice 0 (6–8 weeks):** pilot repo `console` with a minimal pilot version of the `bug-fix` workflow and `docs.link-fix`; Task and `CheckpointRecord` derived from issue/PR/CI events, stored as CI artifacts and PR metadata ([§9.3](09-agent-operating-model.md#s9-3)); reproduction tests written by the verifier; policy repo with centralized required checks, applied only to the pilot repo; check adapters for the pilot repo's toolchain ([§8.7](08-verification.md#s8-7)); the Standard's core integrity checks in pilot mode ([§8.6](08-verification.md#s8-6)); 3–5 high-value rules with fixtures; **`policy.sod`** with authenticated requester identity ([§10.5](10-autonomy-policy.md#s10-5)); a deterministic path→tier map (one control file) instead of a classifier; frontmatter for 3–5 knowledge artifacts and `knowledge.impact` at `warn`; agent sandbox; AGENTS.md and `REVIEW.md` templates; Factory CLI v0 (`doctor`, `context`, `verify:*`); degraded mode runbook and one break-glass drill ([§11.8](11-delivery-change-management.md#s11-8)); baseline measurement.
- **Factory track, follow-up after Slice 0 (about 3–4 weeks, still within Phase 1a):** attestation (separate verify and attest jobs, evidence store, [§8.7](08-verification.md#s8-7)) on the pilot repo, then the mandatory pipeline and attest job on every repo; merge queue on every repo ([§11.6](11-delivery-change-management.md#s11-6)); registry (IDs, schema); **portfolio index v0** (descriptor + lockfile of every repo; for monorepo only needed for consumers outside the repo); `basis` in the Task contract; resolve across profile layers with trace ([§8.3](08-verification.md#s8-3)); `factory manifest`; baseline profile in shadow; **risk classifier v0** (surface/path map, control file change direction, import graph), running in parallel with the path→tier map to measure agreement.
- **Contract track:** `@platform/api-codegen` (OpenAPI → server-only client + MSW + scenarios); contract registry, including `consumer-authored` contracts when the backend has not published OpenAPI ([§4.2](04-technical-foundation.md#s4-2)); Pact broker; contracts for the first 2–3 services.

**Slice 0 exit:** the pilot portion of the Slice 0 scenarios passes ([§20.3](20-showcase.md#s20-3)); `console/compute` has a **pilot conformance claim** (L1 plus the CORE-ID list in [§20.2](20-showcase.md#s20-2)); the end-to-end flow runs on ≥ 15–20 real bug fixes (*suggested*), compared with the baseline on lead time, human attention minutes and escaped defects, and if the sample is insufficient the pilot is extended instead of concluding on a small sample; at least one change touching a described artifact has gone through reconciliation; no gate was bypassed; the sandbox passes tampering/injection eval (0 violations).

**Phase 1a exit:** no run lacking attestation is accepted by policy; the mandatory pipeline and merge queue apply to every repo; a PR modifying CI/adapters does not change its own verification result; classifier v0 agrees ≥ 85% with human labels on the calibration set ([§16.2](16-current-state.md#s16-2)), and every disagreement in the *lower risk* direction is analyzed; client + mock are generated automatically for the first service.

## Phase 1b — Platform skeleton (~1 quarter)

- **DS Foundation:** DTCG tokens, theming, 10–15 core controls built on a headless library (React Aria/Radix) — no hand-written focus/keyboard handling — with a11y pattern contracts; Storybook.
- **App shell**, `@platform/auth` + `@platform/server` (session, mutation wrapper, streaming, SLO/runbook template — [§3.5](03-target-architecture.md#s3-5)), `i18n`, `telemetry`, `flags`.
- **Generators:** `app --profile …` (create a new repo), `console-module`, `package`, `ds-component`; Journey Registry ([§7.7](07-factory-metadata-registry.md#s7-7)); `factory upgrade` (version bump + codemod).
- **Factory to L2:** full Check Registry (mutation proof, fpSlo, budget, ratchet, [§8.2](08-verification.md#s8-2)); Standard core at default enforcement; frontmatter for every Knowledge Record, dependency index in CI, `knowledge.impact` in target mode and `ReconciliationRecord` ([§7.3](07-factory-metadata-registry.md#s7-3)); Test Planner v0 and flaky registry ([§8.8](08-verification.md#s8-8), [§8.10](08-verification.md#s8-10)).
- Baseline profile shadow → active.

**Exit:** the generator creates new app repos that reach L2 immediately, and those repos publish an **L2 conformance claim**; the Slice 1 scenarios proving the L2 requirements pass ([§20.2](20-showcase.md#s20-2)); a real Console screen runs on Next.js with `@platform/server` and a client generated from OpenAPI, with journey + E2E + attestation, meeting its SLO under load test; a DS change is verified against ≥ 2 consumer apps (polyrepo: via prerelease; monorepo: in one PR).

## Phase 2 — Wave 1: Console + Admin + DS (1–2 quarters)

- Console: shell + first 2–3 modules following the Module Contract; minimal customer-side Desk.
- Admin: separate repo, origin, session; `x:privileged-admin`.
- Console and Admin vertical owners build features; the platform team provides the foundation, gates and support.
- Foundation packs; the full resolved profiles of Wave 1 surfaces following the check plan in [§8.9](08-verification.md#s8-9) (≈ 85 active checks).
- Core playbooks at `ACTIVE`: `investigation`, `bug-fix`, `feature-development`, `console-module` ([§9.2](09-agent-operating-model.md#s9-2)), with sub-states, checkpoints, handoffs and recovery properties.
- Token Usage Index + sentinel set; `factory impact`.
- **Pre-production gate** (channel `release`, [§8.1](08-verification.md#s8-1)) for Console and Admin, running before Wave 1's first production deployment, including `ops.readiness-review` ([§19.11](19-nfr-enterprise.md#s19-11)).
- AI review: use disposition data collected in `shadow` mode since Slice 0 to promote Wave 1 `critical` surfaces to `advisory` ([§9.9](09-agent-operating-model.md#s9-9)); this is an L3 criterion.

**Exit:** the Showcase Slice 1 scenarios pass; Console, Admin and platform/DS publish an **L3 conformance claim** ([§18](18-aisf-conformance.md#s18)); ≥ 5 real work items done by agents, with complete evidence, accepted; 100% of admin seeded defects caught; a token/component change correctly traces consumers across repos, 0 regressions escape; flaky rate below the threshold finalized from the baseline (*suggested* ≤ 2%); milestones QE-2 and DK-2 in [§15.2](#s15-2) are met.

## Phase 3 — Evaluation & trust (~1 quarter, overlapping the end of Phase 2)

- All eval layers in [§12](12-evaluation.md#s12), including reviewer quality eval; AI review promoted to `advisory` on the remaining surfaces, and to `blocking` where the threshold is met ([§9.9](09-agent-operating-model.md#s9-9)); trust registry; shadow mode for A0 candidates; circuit breaker operational and drilled via simulation ([§10.6](10-autonomy-policy.md#s10-6)).

**Exit:** regression eval runs automatically on version change; ≥ 1 task class reaches T1; agent legibility has a baseline and at least one convention decision is based on measurements.

## Phase 4 — Wave 2: Website + Content Studio + Docs + Service Desk (~2 quarters)

- Rendering profiles `public-static` and `docs`; headless CMS + content model + **verify-on-publish**; Docs with API reference generated from OpenAPI; Desk agent workspace.
- Profiles `public-web`, `content-studio`, `docs-portal`; Desk pack.

**Exit:** the Slice 2 content channel scenarios pass (SC-14); ≥ 3 archetypes have a vertical slice; publishing faulty content is blocked on seeded content; `thread.visibility-separation` catches a seeded leak on every channel; a content model change traces the affected pages.

## Phase 5 — Wave 3: Chat AI + embedded assistant + portfolio intelligence (1–2 quarters)

- Chat app + assistant in the app shell; mock streaming server; `conversational-ai`, `x:ai-surface`.
- Cross-repo change sets with merge/release ordering; full impact analysis for API/content/token.
- Document Management if needed.

**Exit:** seeded XSS/stream faults are caught; deprecate an API field end-to-end across Console + Docs + SDK examples.

## Phase 6 — Autonomy (ongoing, earliest around quarter 6–7)

A0 for task classes reaching T2; gardening creates PRs; dashboards; incident → learning loop.

**Exit:** an **L4 conformance claim** for at least one surface (SC-15 passes); ≥ 1 task class at A0 for ≥ 8 weeks without an increase in the escaped defect rate; a rollback drill has been performed; full audit.

<a id="s15-1"></a>

## 15.1. Estimates & cost

**Assumptions:** the backend keeps its contract-first commitment; each vertical has its own team building features and operating the app from Phase 2 — that staffing is **not** counted in the table below.

**Frontend Platform team staffing per phase** (suggested — the team builds both the platform and the Factory, so 4–6 people is not enough for Phases 1–3):

| Phase | FTE | Focus |
|---|---|---|
| 0 | 3–4 | ADRs, spikes |
| 1a–1b | 6–8 | Factory spine ∥ contract; DS foundation ∥ shell/`@platform/server` |
| 2–3 | 6–8 | Wave 1 (supporting Console/Admin verticals), eval/trust, maintaining ≈ 85 checks (≈ 0.7–2.8 FTE for this part alone; integrity checks and policy checks are cheap because they only scan the diff) |
| 4–6 | 5–7 | Waves 2–3; verticals build features, the platform operates gates, DS, foundation upgrades |

If only 4–6 people are available: keep the phase order, extend Phases 1a–1b by about one quarter, and narrow Wave 1 to Console + DS (Admin moves to Wave 2).

About **6–8 quarters** to reach Phase 6 for the **platform + Factory**.

**Not included in that figure:** fully building each product on the platform (Console with all modules, Admin, Website, CMS, Desk, Chat, DMS). Vertical owners estimate this part separately in Phase 0: Σ(surface × complexity) minus what generators/packs/DS provide out of the box, plus time running in parallel with existing systems until they are shut down ([§16.3](16-current-state.md#s16-3)). Approvers need to read **both figures together**.

Cost formulas, finalized in Phase 0:
- CI browser minutes ≈ (pushes/month × runs per push + PRs/month × runs in the merge queue) × duration of one run. Example journey `console/compute.create-instance` (4 personas × 4 scenarios): each push 16 Chromium runs; the merge queue adds 8 happy path runs on WebKit/Firefox once per PR; nightly runs the full matrix of 48 runs. If the full matrix is enabled in the merge queue, it is 48 runs per PR ([§8.10](08-verification.md#s8-10)).
- Preview environments ≈ PRs/month × lifetime × unit price.
- LLM ≈ agent tokens + reviewer + nightly eval.
- Visual testing ≈ snapshots/month; evidence storage ≈ artifacts × retention.
- Check maintenance ≈ `active` checks × 0.5–2 person-days/quarter (Wave 1 ≈ 85 checks ≈ 0.7–2.8 FTE).
- Staging and test tenants ≈ number of tenants in the pool × unit price + cleanup cost; Knowledge Index ≈ index size + embedding cost (if there is a semantic layer).

<a id="s15-2"></a>

## 15.2. Testing & Documentation capability milestones

Testing & Quality Engineering milestones have IDs `QE-n`, and Documentation & Knowledge milestones have IDs `DK-n`, to avoid confusion with trust levels T0–T2 ([§10.3](10-autonomy-policy.md#s10-3)).

| Phase | Testing & Quality Engineering | Documentation & Knowledge |
|---|---|---|
| 0 | **QE-0:** test stack ADR; test-data API and tenant pool commitment from the backend; preview environment spike; baseline measurement (test time, flakiness) of existing systems | **DK-0:** document classification and source of truth ([§7.3](07-factory-metadata-registry.md#s7-3)) approved; decision on the owner of Documentation & Knowledge and internal documentation ([§17](17-decisions-to-make.md#s17)) |
| 1a | **QE-1:** Slice 0: runner in the pilot repo's centralized required checks, reproduction tests written by the verifier; then: `@platform/testing` v0, runner in the mandatory pipeline of every repo | **DK-1:** Slice 0: frontmatter for 3–5 artifacts the pilot repo actually uses, `knowledge.impact` at `warn` by `describes` glob, reconciliation decisions recorded in the PR; then: versioned `factory context`; Platform facts block in AGENTS.md and `meta.agents-md-drift` in shadow |
| 1b | **QE-1b:** Test Planner v0 (import graph + surface + floor), `factory plan`, `x-fe.testPlan` in evidence; flaky registry; Journey Registry with `requirements` and `acceptance`; test generator per journey; persona generator; `factory trace` v0; per-PR preview environments | **DK-1b:** record frontmatter per Standard §5.8 (`status`, `authority`, `describes`, `basedOn`, `validatedAgainst`) plus `appliesTo` and `visibility` for every record; dependency index and `knowledge.impact` in target mode; `ReconciliationRecord`; component documentation per [§5.6](05-design-system.md#s5-6) for the first controls; `ds.docs-api-sync` in shadow; skills declare `appliesTo` |
| 2 | **QE-2:** pre-production gate (channel `release`) for Console and Admin: staging smoke, DAST, load test, `can-i-deploy`, readiness review; Test Planner uses the portfolio index (cross-repo); staging with tenant pool, environment manifest and failure attribution; playbooks `test-design` and `regression-from-incident`; Testing Platform metrics have a baseline | **DK-2:** Knowledge Index v0 (lexical + filter, access checked per source) on every Wave 1 repo, after evaluating existing developer portals ([§17](17-decisions-to-make.md#s17)); `factory search`; retrieval eval has a baseline |
| 3 | **QE-3:** flaky and run time thresholds finalized from data; selection miss and test selection eval only when the Test Planner goes beyond the affected level ([§8.10](08-verification.md#s8-10)) | **DK-3:** decide whether to add a semantic layer based on retrieval eval; agent-legibility eval includes documentation quality |
| 4 | **QE-4:** pre-production gate (channel `release`) extended to every Wave 2 app; testing for the content channel | **DK-4:** Docs Portal with docs bundles with provenance, ecosystem-wide search, API reference and SDK reference; freshness and docs update time metrics |
| 5–6 | **QE-5:** test data for streaming and AI; trust for the test gardening task class | **DK-5:** Knowledge Index covers every repo; contextual help in Console uses the shared Docs Portal search |

The showcase demonstrates these two capabilities through SC-17 (testing across surfaces and repos), SC-18 (retrieving the correct document, contract and version), SC-19 (traceability from requirement to evidence, regression from incident) and SC-20 (knowledge changing mid-task) ([§20.3](20-showcase.md#s20-3)).

<a id="s15-3"></a>

## 15.3. Minimum scope per slice

The Factory has many components, while platform staffing is limited ([§15.1](#s15-1)). Rule: **a component is built only when a showcase scenario or a conformance criterion of the current slice needs it.** Components not yet due use the simpler approach listed in the last column.

| Component | Needed from | Before that, use |
|---|---|---|
| Policy repo, centralized required checks only on the pilot repo, check adapters for the pilot repo's toolchain | Slice 0 (pilot) | — |
| The Standard's core integrity checks in pilot mode, 3–5 high-value rules with fixtures, `policy.sod`, deterministic path→tier map | Slice 0 (pilot) | — |
| Factory CLI: `doctor`, `context`, `verify:*` | Slice 0 (pilot) | — |
| Frontmatter for 3–5 knowledge artifacts, `knowledge.impact` at `warn`, reconciliation decisions recorded in the PR | Slice 0 (pilot) | — |
| Minimal pilot version of the `bug-fix` playbook (Standard Appendix C.4), `docs-update` | Slice 0 (pilot) | — |
| Task, `CheckpointRecord`, `Handoff`, `RecoveryEvent`, `basis`, `oracleRefs`: derived from issue/PR/CI events, stored as CI artifacts and PR metadata ([§9.3](09-agent-operating-model.md#s9-3)) | Slice 0 (Task, `CheckpointRecord`, `oracleRefs`), the remaining records when needed; this mechanism is used through the end of Slice 1 | — (no dedicated workflow engine; build vs buy decided before Phase 3, [§17](17-decisions-to-make.md#s17)) |
| Attest job, evidence store, merge queue, mandatory pipeline on every repo, `factory manifest` | Phase 1a follow-up (Slice 1) | Evidence is a CI artifact bound to the commit; branch protection with required checks |
| Risk classifier v0 | Phase 1a follow-up (Slice 1) | Deterministic path→tier map |
| Full Check Registry (mutation proof, fpSlo, budget, ratchet), Standard core at default enforcement | Phase 1b (L2) | 3–5 rules with fixtures; pilot mode |
| Frontmatter for every Knowledge Record, dependency index in each repo's CI, `knowledge.impact` in target mode, `ReconciliationRecord` | Phase 1b (L2, MUST from 2027-04-01) | As in Slice 0 |
| Test Planner v0 (floor + import graph + surface), `factory plan`, flaky registry | Phase 1b (L2) | `verify:*` runs the pilot repo's full suite |
| Playbooks meeting the full §9.2 contract with sub-states, handoffs and recovery; their records as separate artifacts | Phase 2 (L3, SC-21) | Minimal pilot version; summary in the PR template |
| Journey Registry, `factory trace` v0 (reads tags and the evidence store) | Slice 1 (L3, SC-19) | List of critical journeys in `factory.yaml` |
| Token Usage Index, sentinel set, `factory impact` | Slice 1 (SC-09) | Verify all consumers |
| Cross-repo portfolio index, prerelease consumer verification, adoption PR bot | Slice 1, **polyrepo only** | — (not needed for monorepo) |
| Version skew bot | Phase 2+, **polyrepo only** | Manual version bumps on a shared schedule |
| Cross-repo Knowledge Index, `factory search` | Phase 2 (SC-18), after evaluating existing developer portals ([§17](17-decisions-to-make.md#s17)) | Per-repo `factory context` |
| Test Planner as a separate product: per-test coverage map, selection-miss metric ([§8.10](08-verification.md#s8-10)) | When there is a measurable need: defects escaping the PR's plan detected in nightly or in an incident, or run time exceeding the command budget ([§7.6](07-factory-metadata-registry.md#s7-6)) | Floor + import graph; files that cannot be mapped run the surface's full suite |
| Test Asset Catalog ([§8.11](08-verification.md#s8-11)) | When `factory trace` v0 and `factory doctor` are no longer sufficient, after evaluating existing developer portals ([§17](17-decisions-to-make.md#s17)) | `factory trace` v0 + `factory doctor` |
| Agent-legibility eval, retrieval eval | Phase 3 | Manual review on a task sample |
| Knowledge Index semantic layer | When retrieval eval demonstrates benefit ([§7.8](07-factory-metadata-registry.md#s7-8)) | Lexical + filter |
| Change Shepherd | Slice 2 (L4, SC-15) | Release owner watches per-surface dashboards |
| `integrity.mutation-score` at required for R2+ | When there is a measurable need, after baseline ([§8.6](08-verification.md#s8-6)) | `observe`, nightly |
| Escape hatch with dual run ([§8.7](08-verification.md#s8-7)) | When a code-style config need cannot be met by extending the schema | No escape hatches allowed; extend the `factory.config.yaml` schema |
| `assessed-elsewhere` validity logic ([§8.2](08-verification.md#s8-2)) | When the first `cross-team-contract` check goes `active` (expected Admin, Phase 2, SC-12) | This disposition is not used: `NOT_ASSESSED` and block |
| Full three-browser matrix in the merge queue ([§8.10](08-verification.md#s8-10)) | When nightly shows browser-specific defects reaching main | Chromium on every push; happy path on WebKit/Firefox for `critical` journeys in the merge queue; full matrix in nightly |
| Stress mode 20× ([§8.8](08-verification.md#s8-8)) | When timing defects slip through or the flaky rate exceeds the threshold | Stress mode *suggested* 5× |
| Block the task when resolve hits an incomparable conflict ([§8.3](08-verification.md#s8-3)) | Phase 2, or 2027-04-01 if earlier | `warn` + a recorded owner decision |

---

[Contents](README.md) · [← 14. Organization & operations](14-organization-and-operations.md) · [16. Current state: problems and data, not constraints →](16-current-state.md)
