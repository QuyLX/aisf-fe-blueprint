[Contents](README.md) · [← 17. Decisions to make (Phase 0)](17-decisions-to-make.md) · [19. Enterprise NFR baseline →](19-nfr-enterprise.md)

<a id="s18"></a>

# 18. AISF Standard 4.0 conformance (profile declaration)

<a id="s18-1"></a>

## 18.1. Declaration

| Field | Value |
|---|---|
| Profile | `aisf-frontend@1.1.0` (this document) |
| Standard | `aisf@4.0.0`; vocabulary `aisf-vocabulary.yaml` |
| Profile owner | Frontend Platform Owner |
| Profile type | Domain profile (`domain` layer) for frontend; does not use the Standard's prebuilt Core Profile (Standard §20.7), which is intended for scopes that belong to no domain |
| Supported levels | L1–L4, plus pilot claim under the Standard's pilot mode (Standard §20.8) |
| Targets | Slice 0: **pilot claim** for `console/compute` (L1 plus the pilot's list of CORE-IDs, [§20.2](20-showcase.md#s20-2)) · Generator-created repos: L2 from creation, from the end of Phase 1b · Wave 1 (Console, Admin, platform/DS): L3 at Phase 2 exit · At least one surface: L4 in Phase 6 |
| Deviation | One declared deviation, awaiting approval by the Standard's owner via ADR (PC-4): **DEV-FE-01** (table below). Every other rule in this document is equal to or stricter than the Standard |
| Evidence | Showcase scenarios SC-01…SC-21 ([§20](20-showcase.md#s20)), attested runs (from the Phase 1a follow-up), registry records. Requirements marked *from 2027-04-01* or *from 2027-07-01* in Standard §20.6 are implemented before the claim that needs them and no later than their effective date, so L2 and L3 claims remain valid after those dates |

**Declared deviation (PC-4).**

| ID | Standard rule | Profile rule | Rationale | Scope | Revisit when | Status |
|---|---|---|---|---|---|---|
| DEV-FE-01 | A1 (when the requester's sign-off is not counted) and A3 need a code owner slot for **each affected area** (Standard §9.5 item 6, §9.10) | Directly modified surface: independent code owner slot. Surface indirectly affected through a shared asset: covered by consumer verification evidence on that surface, the shared asset owner's slot, and the consumer code owner's visual baseline approval when there is a visual diff ([§10.4](10-autonomy-policy.md#s10-4)) | A package, token, contract or content model change can have dozens of consumers; one slot per consumer makes shared changes unmergeable, while consumer verification verifies the direct impact better than a diff read by the consumer owner | Only surfaces indirectly affected through a shared asset, at A1 and A3. Does not apply to the requester's sign-off at A1, to risk computation, or to A2 | The Standard accepts or rejects the corresponding upstream proposal ([§18.5](#s18-5)); or an escaped defect on an indirect consumer is attributed to this coverage | Awaiting ADR approval by the Standard's owner. Until approved, the Standard applies verbatim |

<a id="s18-2"></a>

## 18.2. Vocabulary mapping (PC-2)

```yaml
profile: aisf-frontend@1.1.0
riskSubTiers: { R0: Low, R1: Low, R2: Medium, R3: High }
trust: standard            # T0–T2 used verbatim
autonomy: standard         # A0–A3 used verbatim; slot details in §10.4
dataClass:                 # contract field x-data-class (§19.7) → Standard dataClass (§10.6)
  public: public
  internal: internal
  personal: confidential
  sensitive: restricted    # special-category personal data, payment data, secrets
  unknown: restricted
retentionClass: standard   # ephemeral/standard/audit/legal-hold; durations decided in §17 item 11
statuses: standard         # no profile statuses
dispositions: standard     # no profile dispositions
findingSeverities:         # inside checks only; they decide FAIL
  axe: { critical: FAIL, serious: FAIL, moderate: report, minor: report }
  lab-perf-budget: { over-budget: FAIL }
evidenceLayers: [static, dom, a11y-tree, browser, runtime-api, field]
gates: [merge, publish, post-release]   # publish also gates the release channel; post-release only for field evidence (§8.1)
channels: [code, content, config, release]   # code/content/config: Standard; release: registered profile extension (§18.3)
taskStates: standard       # Standard §6.5 states; content/config/release use channelState
knowledgeStatuses: standard   # DRAFT … ARCHIVED; only ACTIVE is default basis (§7.3)
knowledgeKinds:            # kind → authority (PC-2)
  spec: intent             # incl. journey acceptance criteria in journeys.yaml (§7.7)
  component: intent        # Contract Spec of a DS component (§5.4); registered kind
  policy: intent           # registered kind
  adr: decision
  proposal: decision
  threat-model: decision   # registered kind
  design: implementation-description
  living-doc: implementation-description   # API/SDK reference, guides, README, *.doc.ts
  runbook: procedure
  playbook: procedure
  agent-guide: guidance    # AGENTS.md, REVIEW.md
  skill: guidance
workflowSubStates:         # sub-state → exactly one parent task state (Standard §6.5)
  coreWorkflows: standard  # bug-fix, feature-development, refactoring, … as in the vocabulary
  console-module:          { PLANNED: [MANIFEST_DESIGNED], EXECUTING: [IMPLEMENTING_SLICE], VERIFYING: [JOURNEY_VERIFIED] }
  design-system-change:    { CONTEXT_READY: [CONSUMERS_IDENTIFIED], PLANNED: [CONTRACT_PROPOSED, CONTRACT_APPROVED], VERIFYING: [PRERELEASE_VERIFIED] }
  content-model-change:    { PLANNED: [MIGRATION_DESIGNED], VERIFYING: [MIGRATION_VALIDATED, PAGES_VERIFIED] }
  codemod-migration:       { PLANNED: [TRANSFORM_REVIEWED], EXECUTING: [CODEMOD_APPLYING], VERIFYING: [EQUIVALENCE_VERIFIED] }
  test-design:             { PLANNED: [SCENARIOS_PROPOSED, SCENARIOS_APPROVED] }
  regression-from-incident: { CONTEXT_READY: [REPRODUCING, REPRODUCED], PLANNED: [SCENARIO_APPROVED], EXECUTING: [FIXING] }
profileLayers: standard    # this profile is the `domain` layer; product profiles are `surface`-layer additions
```

<a id="s18-3"></a>

## 18.3. Registered extensions (PC-5)

| Extension point | Extension |
|---|---|
| Composition dimensions | Archetype ([§2.3](02-portfolio-scope.md#s2-3)), capability pack ([§2.4](02-portfolio-scope.md#s2-4)), cross-cutting profile ([§2.5](02-portfolio-scope.md#s2-5)), rendering profile ([§3.2](03-target-architecture.md#s3-2)), surface ([§7.2](07-factory-metadata-registry.md#s7-2)) |
| Rule groups | `NEXT`, `CTX`, `DS`, `I18N`, `CFG` |
| Checks | Catalog in [§8.4](08-verification.md#s8-4), including the additional sensor `integrity.oracle-lock` and the Documentation & Knowledge checks |
| Evidence fields | `x-fe` on `VerificationRun`, `CheckResult` and `PolicyDecision` ([§8.5](08-verification.md#s8-5)), including test plan, environment manifest, journey/scenario/AC links and failure attribution |
| Change channels | `content` (CMS) and `config` (feature flag) ([§11.4](11-delivery-change-management.md#s11-4)); `release`: promote a release candidate to production, pre-production gate ([§8.1](08-verification.md#s8-1), [§11.5](11-delivery-change-management.md#s11-5)); conforms to the channel requirements of Standard §12.6 |
| Task class, workflow spec (playbook), skill | Task classes `docs.*`, `web.copy-change`, `deps.*`, `codemod.apply`, `factory.generate`, `gardening.*`, each bound to a playbook `id@major`, scope and ceiling sub-tier ([§7.1](07-factory-metadata-registry.md#s7-1)); the profile's own playbooks and their sub-states ([§9.2](09-agent-operating-model.md#s9-2), [§18.2](#s18-2)); stricter checkpoints ([§9.11](09-agent-operating-model.md#s9-11)) |
| Knowledge artifact | `kind` `component`, `threat-model`, `policy`; relations `x-fe:consumes`, `x-fe:owned-by`; fields `x-fe.appliesTo`, `x-fe.visibility` ([§7.3](07-factory-metadata-registry.md#s7-3), [§7.8](07-factory-metadata-registry.md#s7-8)) |
| Verification adapter | Adapters for Vitest, Storybook, Playwright, Pact, axe, Stryker and the frontend integrity detectors ([§8.7](08-verification.md#s8-7) item 6) |
| Conformance criteria | Frontend criteria per level ([§7.5](07-factory-metadata-registry.md#s7-5)) |
| Domain approval slots | DS owner (public DS API); business owner (pricing, terms, legal content) |

<a id="s18-4"></a>

## 18.4. Tightening (stricter than the Standard)

| Standard | This profile |
|---|---|
| A2 needs a code owner only for sensitive areas | A2 always needs an independent code owner of each directly modified surface, plus the shared asset owner when there are indirectly affected surfaces |
| A0 for the Low tier with reversibility HIGH or MEDIUM | A0 only for R0 with HIGH or MEDIUM, or R1 with HIGH ([§10.4](10-autonomy-policy.md#s10-4)) |
| `propose-only` is mandatory for `auth-and-secrets` | Adds `ci-release-and-branch-protection`, `production-operations`, `legal-and-privacy` and loosening changes to control files ([§10.4](10-autonomy-policy.md#s10-4)) |
| `release-readiness` for the High tier; at Medium, the readiness record is confirmed by the `implementation-review` approver | Promotion through the `release` channel at R2+ always needs the release owner's `release-readiness` ([§9.11](09-agent-operating-model.md#s9-11)) |
| Tier by protected group (Standard §9.6) | Every `x:privileged-admin` surface has an R2 (Medium) floor, privileged paths are R3 (High) ([§10.1](10-autonomy-policy.md#s10-1)) |
| Low + reversibility LOW → A2 | R1 + LOW → A3 |
| Low, not eligible → A1 | R1 + MEDIUM without T2 → A2; with T2 → A1, never A0 |
| Attested evidence is mandatory at L3 | Mandatory from L2; generator-created repos attest from day one |
| An escaped defect attributed to the agent lowers trust by at least one level | Lowers trust to T0 |
| Waivers per Standard §9.10 | A `baselineable: false` check can be waived only by the security owner; a waiver longer than 90 days only by the Frontend Platform Owner; an FP-SLO waiver for at most 14 days |
| Freshness window for `assessed-elsewhere` declared by the profile | ≤ 7 days, or until the provider's next deployment, whichever comes first |
| Check budget per wave | Wave 1 ≈ 85 active checks ([§8.9](08-verification.md#s8-9)) |

<a id="s18-5"></a>

## 18.5. Upstream proposals (PC-7)

Open proposals:

| Proposal | Rationale |
|---|---|
| Add a pre-production gate (for example `release`) to the Standard's gate vocabulary | The Standard currently has only `merge`, `publish` and `post-release`. Staging tests (DAST, smoke, load test) must complete before production but do not belong to the merge gate, and placing them at `post-release` is misleading. In the meantime, the profile uses the `release` channel with the `publish` gate ([§8.1](08-verification.md#s8-1)) |
| Rule: `gate: post-release` is valid only with evidence taken from production (`field`) | Prevents mandatory pre-production testing from being moved to after the release |
| Requirement → AC → test → evidence traceability as a standard field of the Task contract | Makes traceability comparable across profiles ([§7.7](07-factory-metadata-registry.md#s7-7)) |
| Retrieval rules of the knowledge index: access per source, default to the correct version of `ACTIVE`, always cite URI@SHA, authority still goes through path lookup, retrieved content is untrusted data, index latency is published ([§7.8](07-factory-metadata-registry.md#s7-8)) | The Standard has a dependency index (Standard §5.9) but no rules for cross-repo retrieval; every multi-repo profile faces the same risks of data exposure and wrong-version use |
| Documentation & Knowledge as a platform capability with a boundary, an owner, a source of truth per document type and a publishing channel that is a copy with provenance ([§3.8](03-target-architecture.md#s3-8), [§7.3](07-factory-metadata-registry.md#s7-3)) | The Standard defines knowledge artifacts but does not define who operates them as a shared capability |
| `failureAttribution` (`product`, `test`, `environment`, `test-data`, `harness`) and the environment manifest as standard evidence fields ([§8.8](08-verification.md#s8-8)) | Separates PR failures from environment failures in a way comparable across profiles; currently an `x-fe` field |
| Test planner rule: when mapping is not possible, run wider, never narrower; the floor level cannot be reduced by anyone ([§8.10](08-verification.md#s8-10)) | Applies to every change-based test selection mechanism, not only frontend |
| Approval scope for shared assets: indirectly affected surfaces are covered by consumer verification plus the shared asset owner's slot (DEV-FE-01, [§18.1](#s18-1), [§10.4](10-autonomy-policy.md#s10-4)) | Replaces the declared deviation with a Standard rule usable by every profile that has shared assets |

Until accepted upstream, the rules above remain profile rules and keep their place in this document. New proposals come from Slice 0–2 data, through the Standard's change process (Standard §17.4).

<a id="s18-6"></a>

## 18.6. Conformance matrix

How this profile implements each core requirement, and which showcase scenario ([§20.3](20-showcase.md#s20-3)) demonstrates it. The Slice 0 pilot claim consists of L1 plus the IDs listed in [§20.2](20-showcase.md#s20-2); they are demonstrated by the pilot portion of SC-01–SC-05 and SC-16, plus registry records and the baseline report.

| Core requirement | Implementation in this profile | Evidence |
|---|---|---|
| CORE-KNW-01 | AGENTS.md ≤ ~150 lines, with a part maintained by the generator ([§7.4](07-factory-metadata-registry.md#s7-4)) | SC-01 |
| CORE-KNW-02 | Per-module `REVIEW.md`, initialized from the archetype and packs ([§7.4](07-factory-metadata-registry.md#s7-4)) | SC-01 |
| CORE-KNW-03 | `owner`/`backupOwner` in `factory.yaml`, CODEOWNERS, `factory doctor --owners` ([§7.2](07-factory-metadata-registry.md#s7-2), [§10.5](10-autonomy-policy.md#s10-5)) | SC-03 |
| CORE-KNW-04 | `meta.descriptor-drift`; portfolio index generated automatically ([§7.2](07-factory-metadata-registry.md#s7-2)) | SC-01 |
| CORE-KNW-05 | Agents escalate conflicting `ACTIVE` records instead of choosing on their own ([§9.4](09-agent-operating-model.md#s9-4)); retrieval treats only the correct version of `ACTIVE` as the default basis ([§7.8](07-factory-metadata-registry.md#s7-8)) | SC-01, SC-18 |
| CORE-KNW-06 | Mandatory frontmatter and the Standard's `DRAFT` … `ARCHIVED` lifecycle; automation only transitions to `NEEDS_REVALIDATION`; validity is not inferred from time ([§7.3](07-factory-metadata-registry.md#s7-3)) | SC-18, SC-20 |
| CORE-KNW-07 | The Knowledge Index is also the dependency index; `knowledge.impact` on every PR ([§7.3](07-factory-metadata-registry.md#s7-3), [§7.8](07-factory-metadata-registry.md#s7-8)) | SC-20 |
| CORE-KNW-08 | Six-step reconciliation, `ReconciliationRecord`, decision rights by authority; `knowledge-reconciliation` playbook ([§7.3](07-factory-metadata-registry.md#s7-3), [§9.2](09-agent-operating-model.md#s9-2)) | SC-20 |
| CORE-KNW-09 | A mismatch between code and an `intent`/`decision` record is a knowledge conflict; agents do not edit specs, ACs, scenarios, oracles or contracts to match the code ([§7.3](07-factory-metadata-registry.md#s7-3), [§9.10](09-agent-operating-model.md#s9-10)) | SC-02, SC-19, SC-20 |
| CORE-WRK-01 | Standard Task contract plus `surfaces` and `journeys` ([§9.3](09-agent-operating-model.md#s9-3)) | SC-01 |
| CORE-WRK-02 | Requester identity taken from the task runner ([§10.5](10-autonomy-policy.md#s10-5)) | SC-03 |
| CORE-WRK-03 | Standard states mapped to frontend steps ([§9.3](09-agent-operating-model.md#s9-3)) | SC-01 |
| CORE-WRK-04 | Loop budget in every playbook ([§9.2](09-agent-operating-model.md#s9-2)) | SC-08 |
| CORE-WRK-05 | One worktree or sandbox per task ([§9.5](09-agent-operating-model.md#s9-5)) | SC-01 |
| CORE-WRK-06 | Merge queue on every repo, from the Phase 1a follow-up ([§11.6](11-delivery-change-management.md#s11-6)) | SC-07 |
| CORE-WRK-07 | Each task class is bound to an `ACTIVE` playbook with all the components of Standard §6.7.1, mapped into the §6.6 framework ([§7.1](07-factory-metadata-registry.md#s7-1), [§9.2](09-agent-operating-model.md#s9-2)) | SC-21 |
| CORE-WRK-08 | A sub-state belongs to exactly one parent state; policy references only parent states ([§9.2](09-agent-operating-model.md#s9-2), [§18.2](#s18-2)) | SC-21 |
| CORE-WRK-09 | Workflow adaptation is additive only; `when` conditions are evaluated by policy ([§9.2](09-agent-operating-model.md#s9-2)) | SC-21 |
| CORE-WRK-10 | `basis` in the Task contract; a changed basis blocks the next gate until a `continue`/`replan`/`reapprove`/`stop` decision ([§9.3](09-agent-operating-model.md#s9-3)) | SC-20 |
| CORE-WRK-11 | `Handoff` record, the recipient checks the artifact, list of context excluded on entry to the verifier ([§9.12](09-agent-operating-model.md#s9-12)) | SC-21 |
| CORE-WRK-12 | Recovery properties per step; retry only for idempotent steps; `irreversible` steps follow A3 controls ([§9.12](09-agent-operating-model.md#s9-12)) | SC-21 |
| CORE-WRK-13 | `cost` ceiling in `policy.budgets` of the Task contract and playbook, per `min(policy, playbook, task)`, with profile defaults by sub-tier and pilot defaults; warning at 80%, on reaching the ceiling stop at a checkpoint and `ESCALATED` ([§9.3](09-agent-operating-model.md#s9-3)) | SC-08 |
| CORE-VER-01 | Surface resolve with sentinel set ([§8.3](08-verification.md#s8-3), [Appendix A](appendix-a-resolve.md#appendix-a)); Test Planner with floor level ([§8.10](08-verification.md#s8-10)) | SC-09, SC-17 |
| CORE-VER-02 | Standard schema plus `x-fe` ([§8.5](08-verification.md#s8-5)) | SC-01 |
| CORE-VER-03 | Standard status and disposition ([§8.5](08-verification.md#s8-5)) | SC-01 |
| CORE-VER-04 | Tests written by the verifier, FE-TEST-003, `integrity.oracle-lock` ([§8.6](08-verification.md#s8-6)); separation of the test designer, verifier and implementer roles ([§9.10](09-agent-operating-model.md#s9-10)) | SC-02, SC-19 |
| CORE-VER-05 | The mandatory pipeline is taken from the policy repo (Slice 0: pilot repo; from the Phase 1a follow-up: all repos); adapters never run repo scripts ([§8.7](08-verification.md#s8-7)) | SC-04 |
| CORE-VER-06 | Separate verify and attest jobs, signing, evidence store, from the Phase 1a follow-up ([§8.7](08-verification.md#s8-7)) | SC-04 |
| CORE-VER-07 | The control file set is derived; change-direction analysis ([§8.7](08-verification.md#s8-7), [§10.1](10-autonomy-policy.md#s10-1)) | SC-02 |
| CORE-VER-08 | Standard integrity checks with frontend detectors, at default enforcement from Phase 1b; the Slice 0 pilot mode does not count for this requirement ([§8.6](08-verification.md#s8-6)) | SC-02 |
| CORE-VER-09 | Flaky registry generated from main; stress mode (*suggested* 5×); only the owner can quarantine ([§8.8](08-verification.md#s8-8)) | SC-06 |
| CORE-VER-10 | Check Registry with mutation proof, fpSlo, budgets and ratchet ([§8.2](08-verification.md#s8-2)) | SC-10 |
| CORE-VER-11 | `assessed-elsewhere` may be used only from when the first `cross-team-contract` check goes `active`, with validity ≤ 7 days or until the provider's next deployment; before that the check is `NOT_ASSESSED` and blocks ([§8.2](08-verification.md#s8-2)) | SC-12 |
| CORE-VER-12 | Check adapters per the Verification Adapter Contract; standard commands, the Test Planner and playbooks reference check IDs only ([§8.7](08-verification.md#s8-7) item 6) | SC-04, SC-21 |
| CORE-VER-13 | Resolve through eight profile layers; resolution trace in evidence; incomparable conflicts make the task `BLOCKED` from Phase 2 or from 2027-04-01, whichever comes first, and before that are `warn` with a recorded owner decision ([§8.3](08-verification.md#s8-3), [Appendix A](appendix-a-resolve.md#appendix-a)) | SC-21 |
| CORE-VER-14 | PRs created by an agent on a developer machine go through exactly the same gates as any PR; `executor: local` results (including local `factory verify` and `factory preflight`) never satisfy a required check; interactive sessions do not accumulate trust and do not go through A0 ([§8.7](08-verification.md#s8-7) item 6, [§9.5](09-agent-operating-model.md#s9-5)) | SC-04 |
| CORE-REV-01 | AI review can only add, never subtract ([§9.9](09-agent-operating-model.md#s9-9)) | SC-05 |
| CORE-REV-02 | `review.aiMode` upgrade per surface ([§9.9](09-agent-operating-model.md#s9-9)) | SC-13 |
| CORE-REV-03 | `review.injection-prescan` ([§9.9](09-agent-operating-model.md#s9-9)) | SC-05 |
| CORE-REV-04 | Finding disposition and dismiss rights ([§9.9](09-agent-operating-model.md#s9-9), [§10.5](10-autonomy-policy.md#s10-5)) | SC-05 |
| CORE-REV-05 | Limit of 3 rounds; re-review on the diff only ([§9.9](09-agent-operating-model.md#s9-9)) | SC-08 |
| CORE-REV-06 | Reviewer quality eval ([§12](12-evaluation.md#s12)) | SC-13 |
| CORE-REV-07 | Seven review checkpoints with conditions and approvers for frontend, including `design-review` when creating a new repo, app, module or package; dedicated `CheckpointRecord` ([§9.11](09-agent-operating-model.md#s9-11)) | SC-21 |
| CORE-GOV-01 | Deterministic classifier with sub-tiers and overrides; Slice 0 uses a deterministic path→tier map (one control file), classifier v0 from the Phase 1a follow-up ([§10.1](10-autonomy-policy.md#s10-1)) | SC-03 |
| CORE-GOV-02 | Reversibility levels ([§10.2](10-autonomy-policy.md#s10-2)) | SC-14 |
| CORE-GOV-03 | Trust registry per task class and model snapshot; `propose-only` per protected group, in effect regardless of trust ([§10.3](10-autonomy-policy.md#s10-3), [§10.4](10-autonomy-policy.md#s10-4)) | SC-13 |
| CORE-GOV-04 | The matrix is at least as strict as the Standard ([§10.4](10-autonomy-policy.md#s10-4), [§18.4](#s18-4)); approval scope for indirectly affected surfaces is deviation DEV-FE-01, awaiting approval ([§18.1](#s18-1)) | SC-03, SC-09 |
| CORE-GOV-05 | Required check `policy.sod`; the requester's sign-off fills the A1 slot at R0/R1 only when the requester owns every affected surface, no breaker has tripped and there is no `blocking` integrity hit ([§10.4](10-autonomy-policy.md#s10-4), [§10.5](10-autonomy-policy.md#s10-5)) | SC-03, SC-15 |
| CORE-GOV-06 | Approval is bound to the SHA; stale approvals are dismissed ([§10.5](10-autonomy-policy.md#s10-5)) | SC-03 |
| CORE-GOV-07 | A0 conditions ([§10.4](10-autonomy-policy.md#s10-4)) | SC-15 |
| CORE-GOV-08 | Circuit breaker ([§10.6](10-autonomy-policy.md#s10-6)) | SC-15 |
| CORE-GOV-09 | Waiver approver ([§10.5](10-autonomy-policy.md#s10-5)) | SC-03, SC-10 |
| CORE-GOV-10 | Trust evaluation per task class with difficulty tiers, critical scenarios, holdout, regression and 95% confidence bounds ([§10.3](10-autonomy-policy.md#s10-3)) | SC-13 |
| CORE-SEC-01 | Agent sandbox and least privilege ([§9.5](09-agent-operating-model.md#s9-5)) | SC-05 |
| CORE-SEC-02 | Handling of untrusted data; injection and tampering evals ([§9.5](09-agent-operating-model.md#s9-5), [§12](12-evaluation.md#s12)) | SC-05 |
| CORE-SEC-03 | Per-provider data policy, decided in Phase 0 ([§17](17-decisions-to-make.md#s17), [§19.7](19-nfr-enterprise.md#s19-7)) | Registry record |
| CORE-SEC-04 | Audit of every agent action ([§9.5](09-agent-operating-model.md#s9-5)) | SC-01 |
| CORE-SEC-05 | Skills and playbooks have a single source, pinned and reviewed ([§7.4](07-factory-metadata-registry.md#s7-4), [§9.2](09-agent-operating-model.md#s9-2)) | Registry record |
| CORE-SEC-06 | The Standard's data classes (Standard §10.6); the contract's `x-data-class` is mapped per [§18.2](#s18-2), unknown = `restricted` ([§19.7](19-nfr-enterprise.md#s19-7)) | Registry record; contract review |
| CORE-SEC-07 | `sec.secrets` on every PR, plus scanning for secrets and personal data before storing evidence, traces and agent output; retention per the Standard's `retentionClass`, durations decided in [§17](17-decisions-to-make.md#s17) item 11 ([§19.1](19-nfr-enterprise.md#s19-1)) | SC-05; evidence store scan report |
| CORE-SEC-08 | Threat model per archetype ([§19.1](19-nfr-enterprise.md#s19-1)), plus a threat model for each agent runtime and tool/MCP integration before entering the allowlist ([§9.5](09-agent-operating-model.md#s9-5)) | Knowledge Record `kind: threat-model` `ACTIVE`; allowlist record |
| CORE-SEC-09 | Prompt injection, tool misuse and exfiltration evals when the runtime, model or tool changes, and at least quarterly ([§9.5](09-agent-operating-model.md#s9-5), [§12](12-evaluation.md#s12)) | SC-05; eval report |
| CORE-SEC-10 | A separate security review pass is the security owner's slot for `x:security-sensitive` and the privileged paths of `x:privileged-admin` ([§10.4](10-autonomy-policy.md#s10-4)) | SC-03; `PolicyDecision` |
| CORE-GOV-11 | Eval lifecycle per Standard §9.2: eval sets have a version, an owner and access control; regression evals run automatically on version change ([§12](12-evaluation.md#s12)) | Phase 3 exit; eval report |
| CORE-DEP-01 | Dependency decision record via FE-DEP-001 ([§4.5](04-technical-foundation.md#s4-5)); lifecycle and support window of foundation dependencies ([§4.8](04-technical-foundation.md#s4-8)) | `DependencyDecisionRecord` linked from the PR |
| CORE-DEP-02 | Vulnerability remediation SLA: critical ≤ 7 days, high ≤ 30 days; medium and low per the Standard §11.7 defaults; overdue blocks release ([§19.1](19-nfr-enterprise.md#s19-1)) | `deps.policy`; gardening report |
| CORE-DEP-03 | License allowlist centralized via the registry proxy and FE-DEP-001; the license check is a required check on every manifest or lockfile change ([§19.2](19-nfr-enterprise.md#s19-2)) | `deps.policy` |
| CORE-DEP-04 | The model of agent configurations and of `x:ai-surface` surfaces is snapshot-pinned, owned, with an evaluated fallback; changing the snapshot reruns evals and resets trust per Standard §11.9 ([§9.2](09-agent-operating-model.md#s9-2), [§12](12-evaluation.md#s12)) | Registry record; eval report |
| CORE-OPS-01 | Content and flag channels ([§11.4](11-delivery-change-management.md#s11-4)) | SC-14 |
| CORE-OPS-02 | Readiness record; rollback = redeploy plus revert on main ([§11.5](11-delivery-change-management.md#s11-5), [§11.7](11-delivery-change-management.md#s11-7)) | SC-15 |
| CORE-OPS-03 | Immutable image, provenance, SBOM ([§3.6](03-target-architecture.md#s3-6), [§19.2](19-nfr-enterprise.md#s19-2)) | SC-04 |
| CORE-OPS-04 | Incident classification leads to a rule, test or eval case ([§11.5](11-delivery-change-management.md#s11-5)) | SC-15 |
| CORE-OPS-05 | Degraded mode and break-glass ([§11.8](11-delivery-change-management.md#s11-8)) | SC-16 |
| CORE-OPS-06 | Limits of the Change Shepherd ([§11.7](11-delivery-change-management.md#s11-7)) | SC-15 |
| CORE-MET-01 | Baseline from existing systems ([§16.2](16-current-state.md#s16-2)); North Star per task class ([§13](13-metrics.md#s13)) | Baseline report |
| CORE-MET-02 | Profile changes are recorded as metric-affecting events ([§13](13-metrics.md#s13)) | Registry record |
| CORE-MET-03 | Workflow metrics per playbook `id@version`; a playbook change that weakens a gate or checkpoint is loosening and does not count as an improvement ([§13](13-metrics.md#s13), [§9.2](09-agent-operating-model.md#s9-2)) | Workflow metrics report |
| CORE-PRF-01 | This declaration ([§18](#s18)) | Assessment record |

---

[Contents](README.md) · [← 17. Decisions to make (Phase 0)](17-decisions-to-make.md) · [19. Enterprise NFR baseline →](19-nfr-enterprise.md)
