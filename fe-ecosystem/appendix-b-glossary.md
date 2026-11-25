[Contents](README.md) · [← Appendix A — Resolve pseudo-code](appendix-a-resolve.md)

<a id="appendix-b"></a>

# Appendix B — Glossary

| Term | Meaning |
|---|---|
| AISF Standard | AI Software Factory Standard 4.0: the Factory's common rules (vocabulary, governance, evidence, conformance). This document is its frontend profile |
| Profile | How the Standard applies to one domain. A profile MAY clarify and tighten the Standard, never loosen it |
| Conformance level / claim | L1 Described, L2 Guarded, L3 Agent-ready, L4 Autonomy-eligible; a claim is a registry record with attached evidence, assessed by someone outside the owning team |
| Pilot mode / pilot claim | The mode the Standard defines for the first pilot on one repo (Minimum Viable Harness, Standard §15.0): the integrity sensor at `warn` except for skip/focus and control-file changes, records inferred from platform events, some components deferred. A pilot claim is L1 plus a list of CORE-IDs, not L2. Slice 0 is this profile's pilot ([§20.2](20-showcase.md#s20-2)) |
| Minimum Viable Harness | The minimum pilot scope per Standard §15.0: one repo, the Bug Fix workflow, a reproduction test written by the verifier, an SoD check, a centralized required check, 3–5 rules, a degraded-mode runbook and break-glass |
| Surface | A group of routes or modules with a consistent archetype and criticality; the unit to which policy applies |
| Archetype | A type of experience (operational, public-web, …) |
| Capability pack | A shared functional block with its own patterns, components and checks |
| Golden path | Next.js plus generators and templates maintained by the platform; repos and modules created from it start at L2 |
| Rendering profile | `authenticated`, `public-static` or `docs`: variants within the same golden path, declared per surface |
| Sub-tier | R0–R3, this profile's subdivision of the Standard's risk tiers (R0/R1 → Low, R2 → Medium, R3 → High) |
| Trust T0–T2 | The Standard's trust levels, by task class and agent configuration |
| Autonomy A0–A3 | The Standard's autonomy levels, defined by approval slots ([§10.4](10-autonomy-policy.md#s10-4)) |
| Approval slot | A required approval (independent, code owner, control-file owner, security, business, DS owner) recorded in the `PolicyDecision` for a specific SHA |
| `propose-only` | The Standard's lowest `allowedAutonomy` value: the agent only proposes, and a human writes the change and opens the PR, regardless of trust level. The profile declares it per protected group ([§10.4](10-autonomy-policy.md#s10-4)) |
| Indirectly affected surface | A consumer of a changed shared package, token, contract or content model; it contributes to risk like a directly modified surface, and is covered by approval under the declared deviation DEV-FE-01 ([§10.4](10-autonomy-policy.md#s10-4), [§18.1](18-aisf-conformance.md#s18-1)) |
| Cost ceiling (`cost`) | The limit on attributable AI and CI cost for a task, set by `min(policy, playbook, task)`; on reaching the ceiling the task stops at a checkpoint and becomes `ESCALATED` ([§9.3](09-agent-operating-model.md#s9-3)) |
| Requester | The authenticated person (or automation owner) who triggers a work item; they sign off but are never counted as an independent approver |
| Disposition | The reason a non-`PASS` result is still counted as satisfied: `not-applicable`, `assessed-elsewhere`, `baselined`, `waived`, `deferred` |
| Control file | Any file that controls agents, feeds risk classification, or defines what a required check runs ([§8.7](08-verification.md#s8-7)) |
| Trusted mechanism | A registered automation whose output only tightens or is identical to a template, and so is merged without a human |
| Vertical owner | The owner of a product or service: repo, features, operations |
| Platform repo | The platform team's monorepo containing the DS, app shell, `@platform/*`, presets and generators; releases versioned packages |
| Portfolio index | An index generated automatically from descriptors, lockfiles and import/token scans of every repo; the source of the consumer graph |
| Version skew policy | The version window for Next.js and `@platform/*` that every repo MUST stay within |
| Console Module Contract | The standard manifest for plugging a service into the Console |
| Server layer (integration BFF) | The server part of each Next.js app, built on `@platform/server`: holds tokens, calls services, handles mutations and streaming |
| Policy repo | The repo containing the verifiers (CI, adapters, profiles); applies to every repo and cannot be modified by a PR |
| Attestation | The signature of a trusted job on a verification result |
| Mutation proof | Evidence that a check fails on a violating fixture |
| Ratchet | A baseline of violations that CI snapshots when a check is activated; it can only shrink. Never applied to a check with `baselineable: false` |
| Journey Registry | A list of journeys with owner, criticality, persona, scenario, requirement and acceptance criterion; the source for E2E, coverage, traceability and L3 criteria |
| `factory.config.yaml` | A project's schema-validated override of the platform preset; the direction of each change can be analyzed |
| Escape hatch | Config in code that goes beyond the schema: R3, verified by dual run (Standard §7.13). Not permitted until there is a measured need ([§8.7](08-verification.md#s8-7)) |
| Change channel | `code`, `content`, `config` (from the Standard) or `release` (an extension of this profile); each channel has its own rules for evidence, gates and approval |
| Verify-on-publish | Verification of CMS content before publishing (content channel) |
| Merge queue | Reruns the deterministic required checks on the latest main before merging; failures are attributed to operations |
| Change Shepherd | An agent that follows one change throughout its rollout, within the scope of audited tools |
| Break-glass | A two-person, time-limited exception used when Factory components are down; full verification is performed afterward |
| Sentinel set | Representative surfaces that are always verified when a shared asset changes |
| Agent-legibility eval | Measures how well the platform, DS and docs help an agent act correctly |
| Showcase scenario | A runnable scenario (SC-xx) that proves a Standard requirement on a real repo ([§20](20-showcase.md#s20)) |
| Testing & Quality Engineering | A shared capability that turns requirements and risks into runnable tests, selects and runs the right tests, and manages test assets, test data and environments ([§3.8](03-target-architecture.md#s3-8)) |
| Documentation & Knowledge | A shared capability ensuring people and agents find the right document, the right version, the right authority, with citations ([§3.8](03-target-architecture.md#s3-8)) |
| Test Planner | A deterministic component that generates a TestPlan (which tests, which environment, which browser) with reasons; it does not run tests ([§8.10](08-verification.md#s8-10)) |
| Test asset | A means of execution for verification: suite, test case, scenario, fixture, persona, mock, visual baseline, seeded defect, report. Distinct from a check, which is policy ([§8.11](08-verification.md#s8-11)) |
| Test Asset Catalog | A test asset inventory generated automatically in CI (not a catalog service), the source for the Test Planner and `factory trace`; built only when `factory trace` v0 is no longer sufficient ([§8.11](08-verification.md#s8-11)) |
| Selection miss | A test that fails in the nightly run and would have caught the defect at PR time but was not selected in that PR's plan ([§8.10](08-verification.md#s8-10)) |
| Pre-production gate | Checks that run on the release candidate in staging and block promotion to production; represented by the `release` channel with the `publish` gate ([§8.1](08-verification.md#s8-1)) |
| Environment manifest | A record of the service versions, data seeds and flags a run executed with ([§8.8](08-verification.md#s8-8)) |
| Oracle | Tests and scenarios used to decide whether a change is right or wrong; locked against the implementer ([§8.6](08-verification.md#s8-6), [§9.10](09-agent-operating-model.md#s9-10)) |
| Knowledge Index | An automatically generated index, with per-source access control, linking records, contracts, SDKs, components, owners and versions; for retrieval only, carries no authority; also the dependency index that `knowledge.impact` queries ([§7.8](07-factory-metadata-registry.md#s7-8)) |
| Knowledge Record | This profile's name for the Standard's knowledge artifact: has `status` (`DRAFT` … `ARCHIVED`), `authority` and the relations `describes`, `basedOn`, …; only `ACTIVE` is a default basis ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| `knowledge.impact` | A check on every PR that lists affected records; `FAIL` when an `intent`, `decision`, `procedure` or `guidance` record is affected without a reconciliation decision ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| Reconciliation | The process of reevaluating records when the system changes, ending in a `keep`, `update`, `supersede`, `deprecate` or `review` decision ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| Playbook (Task Workflow Specification) | A versioned contract defining the workflow of a task class: steps, sub-states, checkpoints, handoffs, recovery, budget ([§9.2](09-agent-operating-model.md#s9-2)) |
| Basis | The records (`id@version`), contract versions and confirmed assumptions a task relies on; a basis change blocks the next gate until a decision is made ([§9.3](09-agent-operating-model.md#s9-3)) |
| Review checkpoint | A review, approval or verification point with required conditions and approvers, recorded as its own `CheckpointRecord` ([§9.11](09-agent-operating-model.md#s9-11)) |
| Handoff | A record of transferring responsibility for a step between roles, with required artifacts and excluded context ([§9.12](09-agent-operating-model.md#s9-12)) |
| `safeState` | The declared safe state of a feature flag; only returning the flag to `safeState` counts as a mitigation action ([§11.4](11-delivery-change-management.md#s11-4)) |
| Source of truth (documents) | The single place where a type of document is edited; every publishing channel and index is a copy with provenance ([§7.3](07-factory-metadata-registry.md#s7-3)) |
| `consumer-authored` | A value of the `authority` field on a contract registry entry: an OpenAPI spec written by FE (or derived from recorded traffic) and reviewed by the backend owner, used when the backend has not yet published a contract; superseded when the backend publishes ([§4.2](04-technical-foundation.md#s4-2)) |
| Docs bundle | A versioned documentation package that a source publishes to the Docs Portal, with manifest and provenance ([§6.8](06-product-patterns.md#s6-8)) |

---

[Contents](README.md) · [← Appendix A — Resolve pseudo-code](appendix-a-resolve.md)
