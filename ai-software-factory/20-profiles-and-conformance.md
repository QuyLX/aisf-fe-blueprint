# 20. Profiles, extension points and conformance

## 20.1. What a profile is

A **profile** is the application of the Standard to a domain (frontend, backend service, SDK, AI application, infrastructure). A profile supplies what the core deliberately leaves open: concrete checks and detectors, composition dimensions, sub-tiers, task classes, workflows, a reference stack and stricter thresholds.

1. A repository MUST conform to the Standard through a registered domain profile or through the Core Profile (§20.7).

**Rationale.** The Standard deliberately leaves domain-dependent matters open. A profile fills those gaps for one kind of software and may be stricter than the Standard, but never looser.

**Example.** Relationships between the layers:

```text
AISF Standard ── normative core · vocabulary · schemas
   │  refined and tightened by
   ▼
Profiles ──────── Core Profile aisf-core (built in, L1–L2, any domain)
                   AISF Frontend Profile 1.1 (showcase); later backend, SDK, AI-application profiles
   │  implemented by
   ▼
Repositories and platforms ── publish conformance claims (L1–L4) backed by evidence
```

## 20.2. Profile Contract

| ID | A profile MUST |
|---|---|
| PC-1 | Declare the exact Standard version it conforms to and the conformance levels it supports |
| PC-2 | Map every concept it uses to the canonical vocabulary (Appendix J): risk sub-tiers to tiers, finding severities to `FAIL`, display labels to one state plus a disposition, workflow sub-states to exactly one parent task state, knowledge artifact kinds to one `authority` |
| PC-3 | Only refine or tighten. It may require additional checks, heavier autonomy levels and stricter thresholds; it may not loosen any core requirement or allow an autonomy level lighter than §9.5 permits |
| PC-4 | Record every deviation as an ADR with rationale, scope and an expiry date or review condition, approved by the Standard owner (or by the profile owner within delegated limits, §17.4 item 19), and list it in the conformance declaration |
| PC-5 | Register every extension (§20.3) in the registry before use |
| PC-6 | Publish conformance evidence: runnable scenarios proving the core requirements it claims (§20.5) |
| PC-7 | Propose upstream, through the Standard change process (§17.4), practices that benefit other profiles |

## 20.3. Extension points

| Extension point | A profile MAY | A profile MUST NOT |
|---|---|---|
| Risk sub-tier | Subdivide tiers (e.g., R0–R3) | Map one sub-tier to more than one tier, or downgrade a protected category |
| Autonomy matrix | Require a heavier level in any cell; add slots for domain owners | Allow a level lighter than §9.5 permits |
| Composition dimension | Define kinds, capability packs, cross-cutting profiles, rendering profiles or runtime profiles | Bypass the resolution rules of §7.3 |
| Check | Add checks to the registry; supply detectors for core integrity checks | Remove core checks or lower their target enforcement |
| Rule category | Register new categories | Redefine core categories |
| Finding severity | Define severities within a check | Add check states or dispositions that allow a pass |
| Evidence field | Add namespaced extension fields (`x-<profile>`) | Change the meaning of core fields |
| Change channel | Govern additional channels; declare sensitivity types | Bypass the channel requirements of §12.6 |
| Task class, workflow spec (playbook), skill | Define them, with a loop budget equal to or below the policy ceiling; define sub-states; add steps, checkpoints or stricter conditions | Exceed the loop budget ceiling; define a sub-state outside a parent state; drop a mandatory framework step (§6.6) or a mandatory checkpoint (§6.12) |
| Knowledge artifact | Register additional `kind` values; declare additional namespaced relationships | Add a state treated as in effect other than `ACTIVE`; let automation make state transitions other than `ACTIVE → NEEDS_REVALIDATION` |
| Verification adapter | Supply adapters for the domain's toolchains | Invoke verifiers outside the adapter contract (§7.16) |
| Profile layer | Add more specific composition layers | Implicitly resolve an incomparable conflict (§7.3) |
| Conformance criteria | Add stricter criteria for any level | Lower a level's core criteria |
| Trust threshold | Raise the minimum evidence level; add difficulty strata and critical scenarios | Lower that level; drop any component of trust evaluation (§9.4) |

## 20.4. Conformance levels

1. Conformance MUST be claimed per **scope** (a repository, or an area within a repository) at a level.
2. Each level MUST include every requirement of the levels below it. The Core Requirements Index (§20.6) lists requirement IDs by level.
3. Requirements marked *agents* in §20.6 MUST be assessed only where agents operate; a scope without operating agents MAY conform at L1–L2.

| Level | Name | Purpose |
|---|---|---|
| **L1** | Described | The scope is legible and owned: entry instructions, ownership with backup owners, canonical status reporting, data governance for every AI provider in use, and a measured baseline |
| **L2** | Guarded | No change can bypass verification or approval: central required checks, control file protection, integrity sensors at default enforcement, separation of duties (SoD), commit-bound approval, merge queue, flaky policy, Check Registry, degraded mode, review checkpoints, knowledge artifact lifecycle and impact analysis, reconciliation, verification adapter |
| **L3** | Agent-ready | Agents can do bounded work with evidence: Task contract and loop budget, independent oracles, attested evidence, calibrated AI review, convergence controls, trust tracked per task class, workflow specs with sub-states, handoffs and recovery, source-of-truth conflict handling, and learning from incidents |
| **L4** | Autonomy-eligible | Zero-human merge may be enabled: the zero-human conditions are in effect, the circuit breaker has been drilled, rollback has been rehearsed, and at least one task class has reached T2 |

**Rationale.** The levels progress from "legible and owned" (L1), through "no change bypasses controls" (L2) and "agents can do bounded work with evidence" (L3), to "zero-human merge may be enabled" (L4). A pilot usually does not target L2 directly: pilot mode (§20.8, roadmap in §15.0) defines a pilot conformance claim of the form L1 plus a list of CORE-IDs, because many L2 requirements (merge queue, Check Registry, tier classifier) belong to later phases.

## 20.5. Conformance claims

A **conformance claim** is a record in the registry following the `ConformanceClaim` schema (Appendix K).

1. Every requirement of the claimed level MUST have evidence: a runnable scenario run, an attested check result, or a registry record.
2. A claim MAY claim a level plus a list of `additionalRequirements` from higher levels. Each requirement in the list MUST be assessed as a requirement of that level, and the claim MUST NOT be presented as a claim of the higher level.
3. The assessor MUST be independent of the team that owns the claimed scope.
4. Claims MUST have an expiry (*suggested* six months) and MUST be reassessed on expiry, on a new major version of the Standard or of the profile, and after any incident attributed to a gap in a claimed requirement.
5. A failed reassessment MUST lower the claimed level until the gap is remediated, and any autonomy that only a higher level permits (e.g., A0) MUST be revoked immediately.
6. A claim MUST record the profile it uses (`profile`, Appendix K): a registered domain profile, or the Core Profile (§20.7).
7. A claim MUST record the `pilot` or `standard` mode (`mode`). Claims in `pilot` mode MUST follow §20.8; claims in `standard` mode MUST be accepted only when the condition in §17.4 item 18 is satisfied.

**Example.** A claim:

```yaml
kind: ConformanceClaim
scope: { repo: console, areas: [console/compute] }
standard: aisf@4.0.0
profile: aisf-frontend@1.1.0
mode: standard                      # pilot | standard (§20.8)
level: L2
additionalRequirements: []          # CORE-IDs above the claimed level, e.g. a pilot claim "L1 + list" (§20.8)
evidence:
  - requirement: CORE-GOV-05
    scenario: SC-03                 # executable scenario in the profile's showcase plan
    run: run-2026-12-14-0412        # attested VerificationRun
deviations: []                      # ADR references (PC-4)
assessedBy: user:independent-assessor   # not a member of the scope's owning team
assessedAt: 2026-12-15
validUntil: 2027-06-15
```

Pilot claim: `mode: pilot`, `level: L1` with `additionalRequirements` set to the CORE-ID list in §20.8 item 3. This claim states exactly what the pilot scope has proven, without implying that L2 has been reached.

## 20.6. Core Requirements Index

1. **The Index is the sole conformance surface.** Conformance claims MUST be assessed only against the CORE-IDs in the table below, plus the stricter criteria declared by the profile in use (§20.3). A MUST requirement of the Standard not referenced by the Index is a design requirement for harness and profile builders; on its own it does not fail a claim.
2. The cited section determines the meaning of an Index entry. An Index entry MUST NOT be stricter than the section it cites; when the two differ, the cited section prevails and the Index is corrected.
3. Effective dates in the table are read against the Standard version the claim pins (§17.4 item 9).

*Level* is the lowest conformance level that requires the entry. *agents* marks requirements that apply only where agents operate. *from <date>* marks a requirement as SHOULD before that date and MUST from that date (§17.4).

| ID | Requirement | Section | Level |
|---|---|---|---|
| CORE-KNW-01 | A root agent instruction file (e.g., `AGENTS.md`) states the map, commands, constraints and approval boundaries | §5.2 item 4 | L1 |
| CORE-KNW-02 | Review guidance written by code owners (`REVIEW.md` or equivalent) for every area with AI review enabled | §5.2.1 item 11 | L2 *agents* |
| CORE-KNW-03 | Ownership is declared; every area and every control file has ≥ 2 eligible owners or a named backup | §4.2, §9.10 | L1 |
| CORE-KNW-04 | Declared metadata is never silently overwritten by discovered or inferred metadata; drift is recorded as a finding | §4.4 item 2, §4.7 item 11 | L2 · *from 2027-07-01* |
| CORE-KNW-05 | Agents stop and surface source-of-truth conflicts that affect correctness or risk | §4.7 | L3 *agents* · *from 2027-07-01* |
| CORE-KNW-06 | Knowledge artifacts have metadata and a lifecycle state; only `ACTIVE` artifacts are the default basis; automation may only transition to `NEEDS_REVALIDATION`; validity is not inferred from modification time | §5.8 | L2 · *from 2027-04-01* |
| CORE-KNW-07 | Knowledge artifact dependency index and a `knowledge.impact` check on every `code` change | §5.9 | L2 · *from 2027-04-01* |
| CORE-KNW-08 | Reconciliation follows the six-step process with recorded decisions; decisions for non-`generatedFrom` artifacts are made by a person holding the authority, and `knowledge.impact` checks this; `intent` and `decision` artifacts are never automatically edited to match the implementation | §5.9, §5.10 | L2 · *from 2027-04-01* |
| CORE-KNW-09 | Divergence between implementation and spec is handled as a knowledge conflict by authority; agents do not edit specs, acceptance criteria, oracles or contracts to match the code | §4.7 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-01 | Every work item has a Task contract; Change Brief fields are complete before `PLANNED` | §6.4, §6.4.1 | L2 · *from 2027-07-01* |
| CORE-WRK-02 | Requester identity is authenticated from the triggering event; automations record their owner | §9.10 | L2 |
| CORE-WRK-03 | Task lifecycle: only transitions in `taskState.transitions`; fixes return to `VERIFYING`; no state advances on evidence from an earlier commit; leaving `ESCALATED` requires a recorded decision and does not reset the budget | §6.5 | L2 · *from 2027-07-01* |
| CORE-WRK-04 | The loop budget is `min(policy, playbook, task)`; budget exhaustion leads to `ESCALATED` | §6.4 | L3 *agents* · *from 2027-07-01* |
| CORE-WRK-05 | Each agent-executed task has an isolated workspace with a pinned base commit | §6.9 items 1–2 | L3 *agents* · *from 2027-07-01* |
| CORE-WRK-06 | The merge queue reruns deterministic required checks on the latest main, with operational accountability | §6.4, §6.9, §7.4 | L2 · *from 2027-07-01* |
| CORE-WRK-07 | Every agent-executed task runs under an `ACTIVE` Task Workflow Specification with all the components of §6.7.1, mapped onto the §6.6 framework without omitting mandatory steps; task types without a spec use the `generic` workflow with trust not exceeding T0 | §6.6, §6.7 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-08 | Sub-states belong to exactly one parent task state; policy, approval and budget reference only parent states | §6.5 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-09 | Workflow adaptation is additive only; `mandatory` steps, `conditional` steps whose condition holds, checkpoints and gates are never skipped; conditions are evaluated by policy; the official tier is computed from the diff, only increases, and crossing the threshold of a pre-implementation checkpoint that has not run requires `reapprove` | §6.7.3 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-10 | Tasks record their `basis`; a basis change blocks the next gate until the authority decides `continue`, `replan`, `reapprove` or `stop`; completed tasks keep their evidence and basis snapshot | §6.4, §6.5 | L2 · *from 2027-04-01* |
| CORE-WRK-11 | Handoffs are recorded; the receiver checks required artifacts and rejects when they are missing; context passes through the task record; handoff to the verifier excludes the implementer's context | §6.13 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-12 | Each step declares timeout, retry, idempotency and side-effect type; automatic retry only for idempotent steps; compensation is declared in advance; an `irreversible` step makes reversibility LOW and requires `plan-review`, `release-readiness` and an explicit go command | §6.14 | L3 *agents* · *from 2027-04-01* |
| CORE-WRK-13 | Each task has a cost ceiling of `min(policy, playbook, task)`; reaching the ceiling stops at a checkpoint and goes to `ESCALATED` | §6.4 | L3 *agents* · *from 2027-04-01* |
| CORE-VER-01 | Required checks are resolved for each change according to its surface composition | §7.3 | L2 · *from 2027-07-01* |
| CORE-VER-02 | Evidence conforms to the canonical schema and is bound to a commit or revision | §7.5, App. K | L2 · *from 2027-07-01* |
| CORE-VER-03 | Canonical states and dispositions; `NOT_ASSESSED` is never `PASS`; a required check without a valid disposition blocks | §7.5 | L1 |
| CORE-VER-04 | Independent oracles for every `critical` acceptance criterion, with the `critical` value confirmed at `requirement-review`; Medium/High scenarios are confirmed before implementation and are read-only to the implementer | §6.4.1, §7.7, §7.13 | L3 *agents* |
| CORE-VER-05 | Required checks run from a central definition that the change cannot modify; repository-defined entry points are invoked only when their definition is a control file, digest-recorded and unchanged or approved per direction | §7.15 (1–2) | L2 |
| CORE-VER-06 | Attested evidence: separate verify and attest jobs, trusted runners, evidence store | §7.15 (3–4) | L3 · *from 2027-07-01* |
| CORE-VER-07 | Control files are protected and derived from required check inputs; classification is direction-aware; trusted mechanisms are registered | §7.13 | L2 |
| CORE-VER-08 | Integrity sensors at default enforcement | §7.13 | L2 · *from 2027-07-01* |
| CORE-VER-09 | Flaky policy: only pre-existing flakes count as flakes; only owners may quarantine; stress mode for timing-sensitive code | §7.14 | L2 · *from 2027-07-01* |
| CORE-VER-10 | Check Registry: mutation proof, owner, lifecycle, false-positive SLO, budget, ratchet | §8.7 | L2 · *from 2027-07-01* |
| CORE-VER-11 | `assessed-elsewhere` is used only when its conditions are satisfied | §7.5, §8.7 | L2 · *from 2027-07-01* |
| CORE-VER-12 | Required checks run through a verification adapter conforming to the §7.16 contract; workflows do not define their own way of invoking verifiers | §7.16 | L2 · *from 2027-04-01* |
| CORE-VER-13 | The check plan is resolved across profile layers; incomparable conflicts fail resolution and require a recorded decision; the resolution trace is included in evidence | §7.3 | L2 · *from 2027-04-01* |
| CORE-VER-14 | Gates apply uniformly to every PR in the governed scope, including PRs from agents used on developer machines; evidence from a `local` executor never satisfies a required check | §6.15, §7.16 | L2 · *from 2027-04-01* |
| CORE-REV-01 | AI review is add-only: it never approves, never fills a human slot, and is never the sole evidence for a merge | §7.9, P20 | L2 *agents* |
| CORE-REV-02 | AI review mode per area (shadow → advisory → blocking), promoted based on measured thresholds | §7.9, §7.12 | L3 *agents* · *from 2027-07-01* |
| CORE-REV-03 | Diffs are treated as untrusted input; deterministic prompt injection pre-scanning | §7.9 | L2 *agents* · *from 2027-07-01* |
| CORE-REV-04 | Every posted finding receives a response; dismissal rights per §9.10; the implementer (implementing agent, or the developer steering a local agent) never dismisses | §7.9, §9.10 | L2 *agents* · *from 2027-07-01* |
| CORE-REV-05 | Convergence controls: round limits, re-review on the diff only, no repeated findings | §7.11 | L3 *agents* · *from 2027-07-01* |
| CORE-REV-06 | Reviewer quality is measured with eval sets derived from incidents and known-clean eval sets | §7.12 | L3 *agents* · *from 2027-07-01* |
| CORE-REV-07 | The review checkpoints of §6.12 are applied with their mandatory conditions, approvers and pass/fail criteria, including `design-review` when creating a new repository, service or public package; review, approval and verification are recorded separately; approval is bound to the object version and basis | §6.12, §9.10 | L2 · *from 2027-04-01* |
| CORE-GOV-01 | The risk tier comes from the change; tier classifier plus override check; each protected group has a floor tier and can only be raised | §9.5, §9.6 | L2 · *from 2027-07-01* |
| CORE-GOV-02 | Reversibility only tightens; unknown means worst case | §9.5 | L2 · *from 2027-07-01* |
| CORE-GOV-03 | Trust T0–T2 per task class and pinned agent configuration; a version change triggers reassessment | §9.4, §9.9 | L3 *agents* · *from 2027-07-01* |
| CORE-GOV-04 | The autonomy level is never lighter than §9.5 permits | §9.5 | L2 · *from 2027-07-01* |
| CORE-GOV-05 | Separation of duties is enforced by a required check: excluding the requester, the author and pushers; slots are combined by union. The only exception: the requester's sign-off fills the A1 slot at the Low tier when the requester owns every affected area, no breaker is tripped and there is no `blocking` integrity sensor hit | §9.5, §9.10 | L2 |
| CORE-GOV-06 | Approval is bound to the commit SHA; stale approvals are dismissed | §9.8 | L2 |
| CORE-GOV-07 | Zero-human merge only when every A0 condition is satisfied | §9.5, §9.6 | L4 · *from 2027-07-01* |
| CORE-GOV-08 | The circuit breaker is operational and has been drilled before zero-human merge | §9.9 | L4 · *from 2027-07-01* |
| CORE-GOV-09 | Waivers have an owner, rationale, scope, expiry and an approver permitted under §9.10; time-limited waivers do not take the control file loosening path | §7.5, §7.13 item 44, §8.7, §9.10 | L2 |
| CORE-GOV-10 | Trust evaluation per task class is explicitly defined, including difficulty stratification, critical scenarios, holdout, regression and confidence bounds | §9.4 | L3 *agents* · *from 2027-04-01* |
| CORE-GOV-11 | Eval lifecycle: a dataset from real tasks and a baseline; evals rerun when the model, prompt, skill, tool or policy changes; only versions meeting thresholds and without regression are promoted; escaped defects are added to the dataset; humans grade blind | §9.2 (1–4, 8–10, 12–13) | L3 *agents* · *from 2027-07-01* |
| CORE-SEC-01 | Least privilege, sandboxed workspaces, no production credentials for coding agents | §10.1 | L2 *agents* |
| CORE-SEC-02 | Untrusted content is separated from instructions; evals for prompt injection, data leakage and tool misuse | §9.1, §10.1 | L3 *agents* · *from 2027-07-01* |
| CORE-SEC-03 | Data governance is defined for every model provider and tool before use | §10.2 | L1 *agents* |
| CORE-SEC-04 | Agent identity and audit trail, without chain-of-thought | §10.3 | L2 *agents* · *from 2027-07-01* |
| CORE-SEC-05 | Factory supply chain: skills, hooks and MCP servers are version-pinned, reviewed and inventoried | §10.5 | L2 *agents* · *from 2027-07-01* |
| CORE-SEC-06 | Data classes are assigned to context sources and tasks, unknown means `restricted`; no data is sent above the class a provider or tool is permitted to receive; `restricted` data and secrets do not enter a model provider's context | §10.6 | L2 *agents* · *from 2027-07-01* |
| CORE-SEC-07 | Agent output, traces and evidence bundles are scanned for secrets and personal data before storage; every stored artifact has a retention class; records that claims, approvals or releases rely on belong to `audit` | §10.7 | L2 *agents* · *from 2027-07-01* |
| CORE-SEC-08 | A threat model approved by the security owner for each agent runtime and tool/MCP integration before it enters the allowlist; each allowlist entry has an owner, pinned version, permission scope and data class | §10.1 item 5, §10.8 | L2 *agents* · *from 2027-07-01* |
| CORE-SEC-09 | Agents with tool permissions over untrusted input are tested for prompt injection before approval, on version change and periodically | §10.9 | L3 *agents* · *from 2027-07-01* |
| CORE-SEC-10 | A separate security review pass on the changes that §10.10 requires; its findings are dismissed only per §9.10 | §7.9 item 9, §10.10 | L2 · *from 2027-07-01* |
| CORE-DEP-01 | Every dependency added, major-upgraded, replaced or removed has a decision record; versions are locked; the SBOM and inventory are updated; security, license and data policies are hard gates | §11.2, §11.3, §11.6 | L2 · *from 2027-07-01* |
| CORE-DEP-02 | Dependency vulnerabilities are scanned and remediated within a severity-based SLA; exceeding the SLA escalates and blocks new dependencies in the scope | §11.7 | L2 · *from 2027-07-01* |
| CORE-DEP-03 | A central license list; `deny` licenses are not added; licenses not yet on the list require a decision; license checking is a required check | §11.8 | L2 · *from 2027-07-01* |
| CORE-DEP-04 | AI/model dependencies pin snapshots, have an owner tracking deprecation, are re-evaluated before switching, and have a declared fallback on production paths | §11.9 | L2 · *from 2027-07-01* |
| CORE-OPS-01 | Every change channel in use meets §12.6 | §12.6 | L2 · *from 2027-07-01* |
| CORE-OPS-02 | Per-change readiness and a recovery path; rollback is complete only when reverted on main or deployment is blocked | §12.2, §6.5 | L3 · *from 2027-07-01* |
| CORE-OPS-03 | Release provenance | §12.3 | L2 · *from 2027-07-01* |
| CORE-OPS-04 | Incident reviews answer the question "which layer should have caught this defect"; eval sets grow from incidents | §12.4 | L3 · *from 2027-07-01* |
| CORE-OPS-05 | Factory degraded mode and break-glass | §12.7 | L2 |
| CORE-OPS-06 | The Change Shepherd acts only through approved, audited tools within policy scope; "never blocked" mitigation actions apply only to `safe-off` objects, tied to an incident or alert | §12.2.1, §12.6 | L2 *agents* · *from 2027-07-01* |
| CORE-MET-01 | A baseline is measured before any improvement claim; the North Star is reported per task class with guardrails | §13.3, §13.4 | L1 (baseline), L3 (North Star) |
| CORE-MET-02 | Changes to verification profiles are logged as events affecting metrics | §13.3 | L3 · *from 2027-07-01* |
| CORE-MET-03 | Metrics are measured per workflow and its version; workflow changes must not weaken gates or checkpoints | §13.7 | L3 · *from 2027-04-01* |
| CORE-PRF-01 | The scope conforms through a registered domain profile meeting PC-1 to PC-7, or through the Core Profile | §20.1, §20.2, §20.7 | L1 |

**Rationale.** Assessors do not grade every sentence of the Standard, but the CORE-IDs of the claimed level, according to the exact meaning of the section each CORE-ID cites. The other requirements describe how to build harnesses and profiles correctly.

## 20.7. Core Profile

The **Core Profile** (`aisf-core`) is a built-in profile owned by the Standard owner and released with the same version as the Standard (e.g., `aisf-core@4.0.0`). It lets a scope without a suitable domain profile (backend, data, infrastructure, internal tools) claim conformance without writing its own profile.

1. A scope MAY claim conformance at L1 or L2 through the Core Profile. L3 or L4 claims MUST use a registered domain profile.
2. The Core Profile has no extensions (§20.3), tightens no criteria of the Standard, and uses the vocabulary (Appendix J) as is. A scope using the Core Profile MUST NOT register sub-tiers, sub-states, rule categories, evidence fields or any other extensions; when it needs them, the scope MUST use or propose a domain profile.
3. The scope's required checks MUST be declared at the `repository` layer (§7.3) in `factory.yaml` (Appendix K) and MUST include at least the scope's build and test.
4. The Core Profile's conformance scenario (PC-6) is the end-to-end flow of §20.8 item 2, run on the claimed scope itself.
5. Detectors for integrity checks (§7.13 item 39) and verification adapters (§7.16) MUST be provided only to the extent required by the CORE-IDs of the claimed level; the scope owner provides them for their stack.
6. The Standard owner publishes the Core Profile's fulfillment of PC-1 to PC-7; a claim through the Core Profile satisfies CORE-PRF-01 without a separate profile. Deviations (PC-4) of a scope using the Core Profile MUST be recorded as ADRs and approved by the Standard owner.
7. When a domain profile for the scope's domain is registered, the scope SHOULD move to that profile at its next claim renewal.

**Rationale.** Not every team has a domain profile available. A backend, data or infrastructure team that wants to start should not have to write a profile before it can run a pilot. With the Core Profile, the team only needs to declare the repository's required checks in `factory.yaml`, provide adapters and detectors for its stack to the extent the level requires, and then claim L1 or L2. The Core Profile stops at L2 because L3 and above require domain-dependent elements (task classes, critical scenarios, specialized detectors) that only a domain profile can describe correctly.

**Example.**

```yaml
kind: ConformanceClaim
scope: { repo: billing-service }
standard: aisf@4.0.0
profile: aisf-core@4.0.0
mode: pilot
level: L1
additionalRequirements: [CORE-KNW-01, CORE-KNW-02, CORE-KNW-03, "…"]   # §20.8 item 3
```

## 20.8. Pilot mode

**Pilot mode** is a time-limited, declared mode in which a scope proves the Standard for the first time on one repository and one workflow. Pilot mode is the sole normative definition of pilot loosenings; every section that says "in the pilot" refers to this section.

1. **Declaration.** Pilot mode MUST be declared in a Pilot Charter approved by the repository owner and the Factory owner, stating: the repository, the workflow (default `bug-fix`), owners, start and end dates (*suggested* maximum 12 weeks; an extension is a recorded decision), and stop criteria. Pilot mode MUST apply to only one repository.
2. **End-to-end flow.** The pilot MUST prove the following flow on real work items: one repository → a workflow with an `ACTIVE` spec → review checkpoints and approval (`requirement-review`, `implementation-review`; `plan-review` when tier High) → independent verification via the verification adapter (§7.16) → evidence bound to the commit → knowledge reconciliation (§5.10) when the change touches a described artifact.
3. **Pilot conformance claim.** The pilot MUST conclude with a claim in `pilot` mode, level L1, with `additionalRequirements` comprising: CORE-KNW-01, CORE-KNW-02, CORE-KNW-03, CORE-KNW-09, CORE-WRK-02, CORE-VER-03, CORE-VER-04, CORE-VER-05, CORE-VER-07, CORE-VER-12, CORE-VER-14, CORE-REV-01, CORE-GOV-05, CORE-GOV-06, CORE-GOV-09, CORE-SEC-01, CORE-SEC-03, CORE-OPS-05, CORE-MET-01 (baseline), CORE-PRF-01. This claim MUST NOT be presented as L2 (§20.5 item 2).
4. **Permitted loosenings.** In pilot mode, and only in pilot mode, the following loosenings MAY be applied:
   1. Integrity sensors run at `warn` per §7.13 items 25–27.
   2. Job separation and the evidence store (§7.15 items 3–4) are deferred; §7.15 items 1–2 MUST still be met.
   3. State, checkpoint, approval and handoff records are derived from labels, PR events and CI runs instead of a task store (§6.5 items 9–10).
   4. Workflow sub-states (§6.5) are derived from platform events or skipped; handoff and recovery records as artifacts are deferred. Verifier independence (§7.7) and the rule that `ERROR` does not consume loop budget (§7.5) MUST still be upheld.
   5. The Learning step (§6.6 step 16) is satisfied by the "Review findings" and "Knowledge impact" sections of the PR description together with workflow metrics collected from events (§13.7), instead of a separate step.
   6. **Tier without a tier classifier.** The automatic tier classifier (§9.6 item 5) is deferred. In that case the tier MUST be taken from a deterministic path → tier mapping table (`riskMap` in `factory.yaml`, Appendix K), which is a control file (§7.13) owned like `CODEOWNERS`; paths not in the table MUST be High (§9.5 rule 4), and the floor tier of protected groups (§9.6 item 14) MUST be applied. Authors and requesters MUST NOT choose the tier. The override check (§9.6 item 5) MAY be the approver's judgment, and may only raise the tier.
   7. The merge queue, the full Check Registry and the automatic circuit breaker are deferred; zero-human merge (A0) MUST NOT be enabled.
5. **No loosening.** Pilot mode MUST NOT loosen: central required checks (§7.15 items 1–2), control file protection and approval (§7.13 items 11–14), separation of duties (§9.10), data governance (§10.2), degraded mode and break-glass (§12.7), or any mandatory workflow checkpoint (§6.12).
6. When pilot mode ends, the loosenings in item 4 cease to be in effect; the scope MUST fully meet the level its next claim declares.
