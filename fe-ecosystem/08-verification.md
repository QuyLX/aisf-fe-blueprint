[Contents](README.md) · [← 7. Factory: metadata, registries, repository standard](07-factory-metadata-registry.md) · [9. Agent Operating Model →](09-agent-operating-model.md)

<a id="s8"></a>

# 8. Verification

<a id="s8-1"></a>

## 8.1. Verification layers

| Layer | Tooling | Backend | Stage | Gate |
|---|---|---|---|---|
| Static | Type, lint, boundary, dependency, secret, SAST | — | Every PR | `merge` |
| Unit/integration | Vitest | Mock | Every PR (per test plan, [§8.10](#s8-10)) | `merge` |
| Component | Storybook interaction/a11y/visual | Mock | Every PR (per test plan) | `merge` |
| E2E journey | Playwright | **MSW generated from OpenAPI** + scenario fixture | Every PR (affected journeys) | `merge` |
| Contract (consumer) | Pact consumer test | Broker | Every PR + provider pipeline | `merge` |
| Pre-production testing | Staging smoke, DAST, load test, `can-i-deploy` | Real backend on staging, test tenant, mail sink ([§8.8](#s8-8)) | Each release candidate | `publish` of the `release` channel |
| Production monitoring | RUM, CWV field data, error rate, SLO | Production | Continuously after deploy | `post-release` |

**Three groups of gates, not to be confused:**

1. **PR/merge gate** (`merge`): everything that can be checked against mocks or code, run in the PR and the merge queue.
2. **Pre-production gate** (`release`): checks that need a production-like environment (real backend, staging) run against the **release candidate**, i.e., the immutable merged image, and **block promotion of that image to production**. The profile represents promotion as a change in the `release` channel ([§11.5](11-delivery-change-management.md#s11-5)), so these checks block at the Standard's `publish` gate, like content and config: before a non-PR change takes effect. The `release` channel is an extension the profile registers per PC-5 ([§18.3](18-aisf-conformance.md#s18-3)) and conforms to all channel requirements of Standard §12.6; bringing a pre-production gate into the Standard's own vocabulary is proposed upstream ([§18.5](18-aisf-conformance.md#s18-5)).
3. **Post-release monitoring** (`post-release`): only for signals **observable only in production** (`evidenceLayer: field`): RUM, CWV field data, error rate, SLO, canary. They block by pausing or rolling back the rollout, and do not replace any testing that should have been completed before production.

Registry rule: a check may only declare `gate: post-release` when `evidenceLayer: field`. Every check that can run on staging belongs to the pre-production gate. Periodic activities (DR drills, rollback drills, annual pentest and a11y audit, nightly mutation testing) **are not gates**: they produce findings and records, and only block a release under named conditions (e.g., pentest before GA, [§19.1](19-nfr-enterprise.md#s19-1)).

**Mapping of NFR ([§19](19-nfr-enterprise.md#s19)) and quality checks to gates:**

| Gate | Security | Performance | Resilience | Accessibility | Other |
|---|---|---|---|---|---|
| PR/merge | `sec.sast`, `sec.secrets`, `deps.policy`, `sec.headers-csp`, `sec.authz-tests`, `sec.server-cache-scope` | `perf.lab-budget`, `perf.route-budget`, `ds.bundle-budget` | `stream.fault-injection`, `realtime.reconnect` (on mock) | `a11y.static-lint`, `a11y.axe-journeys`, `a11y.keyboard-journeys` | `test.e2e-journeys`, contract consumer, `integrity.*` |
| Pre-production (release) | `sec.dast-staging`; pentest record before GA and for Admin/billing at R3 | `perf.load-test` (2× expected peak, surfaces with `criticality` `high` or above) | `test.staging-smoke`; failover of critical upstreams; `contract.can-i-deploy` | Manual audit before GA (attestation) | `ops.readiness-review` (first production deploy) |
| Post-release (monitoring) | WAF and auth anomaly alerts | `perf.field-cwv`, `perf.ttft-ui` | Error rate, SLO, error budget ([§19.4](19-nfr-enterprise.md#s19-4)); canary | User a11y feedback (SLA, [§19.8](19-nfr-enterprise.md#s19-8)) | Change Shepherd ([§11.7](11-delivery-change-management.md#s11-7)) |
| Periodic, not a gate | Annual pentest | — | Annual DR drill, quarterly rollback drill | Annual audit | `integrity.mutation-score` nightly while in shadow |

<a id="s8-2"></a>

## 8.2. Check Registry

Every check is defined **once**, as a Check Registry entry in the Standard (Standard §8.7). This section adds the frontend-specific part:

```yaml
kind: Check
id: seo.canonical
version: 1.2.0
owner: web-platform
lifecycle: active             # planned | shadow | active | deprecated
ownership: local           # local | cross-team-contract
fixOwner: web-platform
implementsRules: []           # rules this check enforces, e.g. FE-API-001
baselineable: true            # false: never ratcheted (secrets, tenant leaks, critical vulns...)
fpSlo: 0.05
autoDemote: true              # must be false when basis.class ∈ {security, privacy, wcag, contract}
basis: { class: standard, source: "Google Search Essentials — canonicalization" }
evidenceLayer: dom            # static | dom | a11y-tree | browser | runtime-api | field
fixtures:
  conforming: [fixtures/seo-canonical/pass]
  violating:  [fixtures/seo-canonical/missing, fixtures/seo-canonical/relative-url]
remediation: "Use the golden path's metadata helper"
```

**Rules for every check:**
1. **Mutation proof:** violating fixtures are mandatory; registry CI proves the check fails on those fixtures.
2. **Enforcement follows the source:** only `required` when the source is a mandatory standard (WCAG A/AA, a contract, an `ACTIVE` record).
3. **Honest evidence layer:** if the harness cannot observe the required layer → `NOT_ASSESSED`, never a false PASS.

**Lifecycle & budget:**
- `planned → shadow` (runs for real, never blocks, minimum 2–4 weeks) `→ active → deprecated`. Only `active` checks may be `required`.
- A **false positive** only counts when the check owner (or a delegate, not the PR author) confirms it. Disputes are counted separately; a dispute rate above threshold is reviewed by the Frontend Platform Owner.
- Exceeding `fpSlo`: `autoDemote: true` → a bot opens an audited policy change to demote the check to `advisory`; `autoDemote: false` → the check keeps blocking, a P1 work item is opened, and only scoped waivers of at most 14 days are allowed.
- **Budget of `active` checks per wave**; registry CI rejects anything over budget (suggested Wave 1 ≈ 85, [§8.9](#s8-9)). Each `active` check costs about 0.5–2 person-days per quarter to maintain.
- `baselineable: false` → violations must be fixed immediately or have a waiver from the security owner; never baselined.
- **Activating checks on existing code:** when a `baselineable: true` check moves `shadow → active`, pre-existing violations — in **any** code, including new foundation code — are snapshotted by CI into a baseline on the target branch (not via anyone's PR); new violations are blocked; the baseline can only shrink. A `baselineable: false` check may only become `active` when violations reach 0.

**Ownership:** `cross-team-contract` checks are detected in the frontend but their cause lies in the backend (internal note visibility via API/email, SLA, step-up auth, deletion at the API). These checks are implemented as contract tests in the provider's pipeline; the **`assessed-elsewhere`** disposition (on status `NOT_ASSESSED`, Standard §7.5) is only valid when: (1) the provider run is attested, (2) the verified provider version is exactly the version deployed (or about to be deployed) in the target environment (the broker records deployments, `can-i-deploy` style), (3) the run is still valid — **≤ 7 days or until the provider's next deploy, whichever comes first**. Otherwise → `NOT_ASSESSED` → blocks, assigned to `fixOwner`.

**When to enable `assessed-elsewhere`.** The validity logic above is only built when the first `cross-team-contract` check becomes `active` (expected to be Admin's `admin.step-up-auth`, Phase 2, SC-12). Before then no check uses this disposition: the result is `NOT_ASSESSED` and blocks under the general rule, so CORE-VER-11 is still satisfied ([§15.3](15-roadmap.md#s15-3)).

<a id="s8-3"></a>

## 8.3. Profiles: three orthogonal dimensions

| Dimension | Values | Meaning |
|---|---|---|
| `requirement` | `required` \| `advisory` | Whether it blocks |
| `when` | expression (default `true`) | Whether it applies to the surface |
| `gate` | `merge` \| `publish` \| `post-release` | Where it blocks (Standard §7.4). `publish` is used for the content, config and release channels (pre-production gate); `post-release` only for `evidenceLayer: field` checks ([§8.1](#s8-1)) |

Enforcement per the Standard: `shadow` → `observe`, `active` + `advisory` → `warn`, `active` + `required` → `blocking` (Standard §7.5). The severity of findings within a check is defined by the profile and only determines whether the check `FAIL`s (e.g., axe findings at `critical` and `serious` make the check `FAIL`).

```yaml
kind: Profile
id: archetype:public-web
version: 1.0.0
checks:
  - { id: seo.metadata,           requirement: required }
  - { id: seo.canonical,          requirement: required }
  - { id: seo.structured-data,    requirement: required, when: "surface.features.structuredData" }
  - { id: privacy.consent-gating, requirement: required }
  - { id: perf.field-cwv,         requirement: required, gate: post-release }
```

**Resolve** ([Appendix A](appendix-a-resolve.md#appendix-a)):

1. **Affected surfaces** = direct + via the repo's import graph + consumers in the portfolio index when a shared asset changes (platform repo packages and tokens, contracts in the contract registry, schemas in the `content-models` repo) + sentinel set.
2. **Profile layers** (Standard §7.3), from general to specific: `core` (Standard) → `domain` (this profile) → `organization` → `repository` (`factory.config.yaml`) → `surface` (`baseline ∪ archetype ∪ packs ∪ cross-cutting ∪ product`) → `task-class` (playbook) → `risk` (sub-tier and reversibility) → `task` (Task contract, may only add checks).
3. **Merge per check:** `requirement` takes the max, `when` takes OR, `gate` takes the stricter value, ordered params take the stricter value, set-valued params take the union. A layer attempting to loosen another layer is rejected.
4. **Incomparable conflicts** (two runtime versions, two mutually exclusive params, two gates that cannot both be satisfied) make resolve fail: the task moves to `BLOCKED` with a conflict record; the owners of the profiles involved decide; the decision is recorded as a versioned resolution entry in the more specific profile, or as an ADR when it sets a precedent. There are no implicit overrides.
   **When to enable blocking:** until Phase 2, or until 2027-04-01 when CORE-VER-13 becomes MUST if that date comes first, incomparable conflicts yield `warn` instead of `BLOCKED`: the plan provisionally uses the value of the more specific layer, the trace records `provisional`, and the owners of the profiles involved must record a decision as above ([§15.3](15-roadmap.md#s15-3)).
5. **The resolution trace** (layers and versions, conflicts, rules or decisions applied) is recorded in the evidence (`resolution`, Standard Appendix K).

`shadow` checks run with `observe` enforcement (shown in the PR, never blocking). A waiver (owner, reason, expiry, scoped approver) is the only way to bypass a `required` check.

<a id="s8-4"></a>

## 8.4. Check catalog (target, not v1 scope)

| Profile/pack | Key checks |
|---|---|
| **Standard core** (every surface; cannot be removed) | Blocking: `policy.sod`, `review.injection-prescan`, `integrity.test-count-delta`, `integrity.skip-focus`, `integrity.suppressions`, `integrity.control-file-change`, `integrity.expected-value-edit`, `integrity.self-mocking`, `integrity.vacuous-test`, `integrity.oracle-lock` (profile addition), `knowledge.impact` (`FAIL` for `intent`/`decision`/`procedure`/`guidance` records, `warn` for `implementation-description`; [§7.3](07-factory-metadata-registry.md#s7-3)). Warn: `integrity.tolerance-inflation`, `integrity.broad-catch`, `integrity.snapshot-churn`, `integrity.threshold-change`. `integrity.mutation-score` at `observe` (shadow, nightly); required for R2+ is deferred until there is a measured need ([§8.6](#s8-6)) |
| **Enterprise NFR** ([§19](19-nfr-enterprise.md#s19)) | `sec.sast`, `supply.provenance`, `supply.sbom`, `api.problem-details`, `browser.support-matrix`; pre-production gate ([§8.1](#s8-1)): `sec.dast-staging`, `test.staging-smoke`, `perf.load-test` (when), `contract.can-i-deploy`, `ops.readiness-review` (an app's first production deploy) |
| **Documentation & Knowledge** ([§5.6](05-design-system.md#s5-6), [§7.3–7.4](07-factory-metadata-registry.md#s7-3), [§7.8](07-factory-metadata-registry.md#s7-8)) | `meta.agents-md-drift`, `knowledge.record-schema` (valid frontmatter, `appliesTo`, `visibility`), `knowledge.references` (mentioned record IDs, paths and commands all exist), `ds.docs-api-sync` (in `design-system`), `docs.freshness` (advisory) |
| `baseline` | `build.install-frozen`, `build.compile`, `quality.typecheck`, `quality.lint`, `arch.boundaries`, `arch.cycles`, `deps.policy`, `sec.secrets`†, `test.unit-affected`, `meta.config-parity`, `sec.sast`, `sec.cache-headers` and `sec.server-cache-scope`† (when `rendering: authenticated`; the second check uses lint plus a runtime test: two personas in two tenants call the same route in succession, and the second response contains none of the first persona's data), `a11y.static-lint`, `meta.descriptor-drift`, `docs.public-contract-change` (when) |
| `operational` | `test.e2e-journeys`, `authz.permission-matrix`, `authz.negative`, `tenant.context-isolation`†, `api.client-contract`, `state.ui-states`, `state.lro-lifecycle` (when), `visual.critical-surfaces` (when) |
| `content-studio` | `content.schema-validate`, `content.lifecycle`, `content.revision-restore`, `authz.content-roles`, `content.preview-parity`, `content.locale-fallback` (when), `publish.revalidation`, `a11y.authoring-ui` |
| `public-web` | `seo.metadata`, `seo.canonical`, `seo.sitemap-robots`, `links.integrity`, `responsive.viewports`, `perf.lab-budget`, `privacy.consent-gating`, `thirdparty.script-policy`, `seo.structured-data` (when), `i18n.hreflang` (when), `perf.field-cwv` (post-release) |
| `docs-portal` | `docs.build`, `links.integrity`, `docs.nav-search-smoke`, `docs.code-samples` (when), `docs.api-reference-sync`, `docs.source-provenance` |
| `conversational-ai` | `chat.render-sanitization`, `chat.ime-composition`, `chat.history-crud`, `chat.error-taxonomy`, `chat.long-thread-perf`, `chat.tool-action-confirmation` (when), `perf.ttft-ui` (post-release) |
| `embedded-ui` | `embed.lifecycle`, `embed.css-isolation`, `embed.event-contract`, `sec.postmessage-origin`, `sec.csp-compat` |
| `design-system` | `ds.story-interaction`, `visual.stories`, `a11y.pattern-contract`, `ds.token-schema` (contrast), `ds.api-compat`, `ds.semver-changelog`, `ds.consumer-verification`, `ds.bundle-budget`, `ds.docs-api-sync`, `docs.code-samples` |
| `x:a11y` | `a11y.axe-journeys`, `a11y.keyboard-journeys`, `a11y.focus-management` |
| `x:security-sensitive` | `sec.authz-tests`, `sec.headers-csp`, `sec.open-redirect` |
| `x:privileged-admin` | `admin.impersonation-controls`, `admin.cross-tenant-confirmation`, `admin.pii-masking`, `admin.no-third-party-scripts`, `audit.events`; `admin.step-up-auth`, `admin.network-exposure` (cross-team-contract) |
| `x:ai-surface` | `ai.output-sanitization`, `ai.link-safety`, `ai.streaming-states`, `ai.action-confirmation`, `a11y.live-region`, `privacy.conversation-telemetry`† |
| `x:i18n` | `i18n.missing-keys`, `i18n.pseudo-locale-layout`, `i18n.formatting` (numbers, currency, dates per locale) |
| `x:perf-app` | `perf.route-budget` (per-route bundle, time to primary interaction) |
| `x:perf-public` | `perf.lab-budget`, `perf.field-cwv` (post-release) — merged with `public-web` per the resolve rules |
| `x:theming` | `visual.theme-matrix`, `ds.token-schema` (contrast for every theme/mode) |
| `cap:workflow-state` | `workflow.transition-matrix`, `workflow.illegal-transition-blocked`, `workflow.concurrent-update` |
| `cap:threaded-conversation` | `thread.visibility-separation`† (UI/API/email/notification/export), `thread.html-email-sanitization`, `thread.reply-mode-clarity` |
| `cap:attachments` | `asset.upload-limits`, `asset.access-control`, `attach.preview-safe`, `doc.transfer-integrity` |
| `cap:import-export` | `export.csv-injection` (text cells starting with `=`, `+`, `-`, `@`, tab or CR are neutralized; numeric cells are written as numbers), `export.scope-permission` |
| `cap:billing-payments` | `pay.hosted-fields`† (card data never touches our DOM), `pay.idempotent-submit`, `pay.price-source` |
| `cap:streaming-response` | `stream.lifecycle`, `stream.fault-injection`, `stream.render-performance` |
| `cap:records-retention` | `records.version-restore`, `records.retention-deletion`† (deleted items disappear from UI, API, search, export), `records.legal-hold` (when) |
| Other packs | `sla.*`, `queue.*`, `notify.content-leak`, `search.tenant-scope`†, `bulk.dry-run-preview`, `bulk.partial-failure`, `audit.display-integrity`, `richtext.paste-sanitization`, `realtime.reconnect`, `viz.data-freshness`, `viz.a11y-alternative`, `forms.validation`, `forms.double-submit` |

† `baselineable: false`.

Human review (security review, manual a11y audit) is **not** a check — it is an approval/attestation in the `PolicyDecision`.

<a id="s8-5"></a>

## 8.5. Evidence contract

Evidence uses the **Standard's schema** unchanged (Standard §7.5 and Appendix K: `VerificationRun`, `CheckResult`, `PolicyDecision`, `ApprovalSlot`, `Waiver`). This profile only adds namespaced fields:

```typescript
// Frontend extension fields, stored under the key "x-fe" in `extensions` (Standard Appendix K)
interface FeRunExtension {            // VerificationRun.extensions["x-fe"]
  surfaces: string[];                 // affected surfaces after resolution (§8.3)
  sentinelSurfaces: string[];
  browsers?: Array<"chromium" | "webkit" | "firefox">;
  dataFixture?: string;               // scenario fixture set (§8.8)
  testPlan: string;                   // digest of the TestPlan artifact (§8.10)
  environment?: { id: string; kind: "mock" | "preview" | "staging" | "production-canary"; manifestDigest: string }; // §8.8
}
interface FeCheckExtension {          // CheckResult.extensions["x-fe"]
  evidenceLayer: "static" | "dom" | "a11y-tree" | "browser" | "runtime-api" | "field";
  ownership: "local" | "cross-team-contract";
  lifecycle: "shadow" | "active";
  journeys?: string[];                // Journey Registry IDs (§7.7)
  scenarios?: string[];               // scenario IDs covered
  acceptanceCriteria?: string[];      // AC IDs covered (§7.7)
  failureAttribution?: "product" | "test" | "environment" | "test-data" | "harness"; // set on FAIL/ERROR after triage (§8.8)
}
interface FePolicyExtension {         // PolicyDecision.extensions["x-fe"]
  subTier: "R0" | "R1" | "R2" | "R3";
  surfaces: string[];
  zeroHumanOptIn: boolean;
}
```

Status and disposition belong to the Standard. A `required` check is satisfied by `PASS`, or by a status other than `PASS` with a valid disposition:

| Situation in this profile | Status and disposition |
|---|---|
| The `when` expression is false for every affected surface | `NOT_ASSESSED` + `not-applicable` |
| A `cross-team-contract` check is verified in the provider's pipeline under the conditions in [§8.2](#s8-2) | `NOT_ASSESSED` + `assessed-elsewhere` |
| A violation matches the baseline CI snapshotted when the check was activated ([§8.2](#s8-2)) | `FAIL` + `baselined` |
| A PR fixes a violation that is in the baseline | `PASS`; a PR is never blocked for having fixed a violation. The old entry is invalidated immediately (not used to match any new violation, even at the same location), and the baseline-shrinking bot (a trusted mechanism) removes it in an automated commit after merge |
| Scoped, time-boxed exception | `FAIL` or `NOT_ASSESSED` + `waived` (approver per [§10.5](10-autonomy-policy.md#s10-5)) |
| The browser or runtime layer cannot be observed in the run | `NOT_ASSESSED`. This status blocks, unless an authorized person sets `deferred` to the pre-production gate (`release` channel), where the check must be run against the release candidate. Only checks with `evidenceLayer: field` may be deferred to `post-release` ([§8.1](#s8-1)) |
| The test environment or test data is not ready (environment health check fails, [§8.8](#s8-8)) | `ERROR`, with `failureAttribution: environment` or `test-data`. Never a PR `FAIL` |
| Infrastructure error (exit code `2`, [§7.6](07-factory-metadata-registry.md#s7-6)) | `ERROR`. Rerun; never counted as a pass; does not consume repair budget |

Every other result fails and blocks at the check's gate. Results are never aggregated into a score.

<a id="s8-6"></a>

## 8.6. Test Oracle Integrity

The Standard's integrity checks, their default enforcement modes and the pilot mode belong to the Standard (Standard §7.13). This profile provides the frontend detectors and one additional sensor:

| Standard check | Frontend detector |
|---|---|
| `integrity.test-count-delta` | Number of Vitest, Storybook interaction and Playwright tests and assertions per affected surface, compared with the base commit |
| `integrity.skip-focus` | New `.only`, `.skip`, `.todo`, `test.fixme`, `describe.skip`, `xit`, or a Storybook test-exclusion tag |
| `integrity.suppressions` | `eslint-disable*`, `@ts-ignore`, `@ts-expect-error`, `as any`, `as unknown as`, and comments justifying a workaround (FE-CODE-001, starting at observe) |
| `integrity.snapshot-churn` | `__snapshots__`, visual baselines or golden files updated without being linked to an acceptance criterion. Agents never accept baselines (FE-TEST-002) |
| `integrity.threshold-change` | Bundle budgets, coverage thresholds, lab-performance budgets, axe rule configuration |
| `integrity.control-file-change` | Every file in the control file set ([§8.7](#s8-7)) |
| `integrity.expected-value-edit` | Changed `expect(...)` values or matcher forms, existing scenario fixtures, Pact interaction expectations |
| `integrity.self-mocking` | Mocking (`vi.mock`, MSW override, `page.route`) the very module under test or the code path the acceptance criterion checks; hand-editing generated MSW handlers (FE-TEST-004) |
| `integrity.vacuous-test` | Tests without assertions, early `return`, `expect(true)`, assertions in callbacks that are never awaited, Playwright tests without web-first assertions |
| `integrity.tolerance-inflation` | Increased Playwright `timeout` or `retries`, `waitForTimeout`, larger visual-diff thresholds, fake timers advanced to mask ordering |
| `integrity.broad-catch` | New empty `catch` blocks or ones that ignore errors, swallowed promise rejections, error boundaries that mask errors without reporting |
| `integrity.mutation-score` | Stryker on changed lines, at `observe` (nightly). **When to enable required for R2+:** deferred until there is a measured need (e.g., an escaped defect where tests touched the code but did not assert the behavior), with the threshold set after a baseline ([§15.3](15-roadmap.md#s15-3)) |
| `integrity.oracle-lock` *(profile addition)* | Files the verifier wrote for the work item (reproduction tests and acceptance tests, [§9.3](09-agent-operating-model.md#s9-3)) are not changed by the implementer |

Rules FE-TEST-001…004 ([§4.5](04-technical-foundation.md#s4-5)) complement the detectors. For the frontend, the "check that should have caught the change" that the Standard forbids weakening in the same PR includes visual baselines, scenario fixtures and generated mocks (FE-TEST-002, FE-TEST-004).

**Pilot mode (Slice 0).** Per the Standard's pilot mode, sensors run at `warn`, except `integrity.skip-focus` and `integrity.control-file-change` at `blocking`. The profile additionally runs `integrity.oracle-lock` at `blocking`, because CORE-VER-04 is part of the pilot claim. Pilot mode does not count toward CORE-VER-08; default enforcement is enabled in Phase 1b ([§15](15-roadmap.md#s15)).

How agents take part in test design, generate regression tests from bugs or incidents, and how the acceptance oracle is protected from the implementing agent is covered in [§9.10](09-agent-operating-model.md#s9-10).

<a id="s8-7"></a>

## 8.7. Trustworthy evidence: verifier separated from the PR (Standard §7.15)

1. **A separate policy repo** contains the standard CI workflows, check adapters, profiles and kit versions. It is applied to **every frontend repo in the organization** as a **mandatory pipeline that PRs cannot modify**. The CI platform must meet the checklist at the end of this section.
2. **Job separation:** the *verify* job runs the PR's code and has no permission to sign or write to the evidence store; the *attest* job runs on a trusted runner, never checks out or runs PR code, receives results by digest, checks `verifierRef`, and signs **keyless via OIDC** (Sigstore/SLSA style).
3. **Tiered configuration.** Presets (lint, TS, test, bundle budget) are released from the platform repo, but **the preset version used for verification is specified by the policy repo**, not taken from the `package.json` of the repo being verified. Each project only declares differences in a schema-validated `factory.config.yaml` (adding path aliases, adding rules, changing thresholds), so the classifier can analyze the direction of change as with any policy file: tightening or neutral keeps the PR's risk, loosening is R3. The project's `eslint.config.*`/`tsconfig*` files are **generated** from preset + overrides and checked for template parity (FE-CFG-001).
   **Escape hatch** — when code-form config beyond the schema's capabilities is needed: **dual run** (verifier with the base config — result signed; with the PR config — unsigned, for reference); the PR is R3/A3; after merge, **post-merge verify** on main, fail → automatic revert. An escape hatch recurring ≥ 2 times is a signal to extend the schema.
   **When to enable the escape hatch:** deferred until there is a measured need, i.e., a code-form config need that cannot be solved by extending the schema. Until then the escape hatch is not allowed, and such needs are solved by extending the `factory.config.yaml` schema ([§15.3](15-roadmap.md#s15-3)).
4. Changes to the policy repo have their own review pipeline; verifier version upgrades are applied to repos via separate changes in the policy repo, with a shadow run on every repo first.
5. **Residual risk:** tests still run PR code, so malicious code can recognize the test environment. Mitigated in layers: independent reviewer, seeded defects, staging smoke, canary.
6. **Check adapters per the Verification Adapter Contract** (Standard §7.16). Each adapter (Vitest, Storybook, Playwright, Pact, axe, Stryker, integrity detectors…) has an `adapterId`, version and owner, and is registered in the policy repo's registry; receives a `VerificationRequest` (subject, resolved plan, work item, gate); pins tool versions and image digests; runs with `executor` `ci`, `agent-sandbox` or `local`, egress blocked by default, no secrets by default, source read-only outside the output directory; declares the check IDs and evidence layers it can observe; returns `CheckResult` per the standard schema, with timeouts and infrastructure errors as `ERROR`. Adapters have fixture repos and mutation proofs like checks. Standard commands ([§7.6](07-factory-metadata-registry.md#s7-6)), the Test Planner and playbooks only reference check IDs and always go through adapters; there is no other path for invoking a verifier. Changing an adapter is a verifier change (item 4). `executor: local` results (developer machine or local agent, [§9.5](09-agent-operating-model.md#s9-5)) are for reference only and never satisfy a required check at the `merge` or `publish` gate; only results from the mandatory pipeline count (Standard §6.15).

7. **Control files in product repos** (Standard §7.13). This set is **derived** from what the mandatory pipeline's adapters execute or read, plus the declared groups. `factory doctor --control-files` reports every input of a required check that this set does not cover. At minimum:

| Group | Files |
|---|---|
| Risk classification inputs | `factory.yaml`, `journeys.yaml` (criticality, personas), `CODEOWNERS` |
| Check configuration | `factory.config.yaml`; the generated files `eslint.config.*`, `tsconfig*.json`, `vitest.config.*`, `playwright.config.*`, Storybook test configuration, test setup files |
| Check execution inputs | `package.json` `scripts`, `packageManager` and resolution overrides; any local CI files in the repo (there should be none) |
| Agent steering files | `AGENTS.md` and `REVIEW.md` at every level; `.factory/**` (skills, playbooks, hooks, agent and MCP configuration) |
| Policy repo | The entire repository. It is outside every product PR and has its own review pipeline (item 4) |

A lockfile change is a dependency change (`deps.policy`), not a control file change. Scenario fixtures and generated mocks are test oracles, governed by `integrity.expected-value-edit` and FE-TEST-004 rather than the control file rules, which keeps adding new scenarios cheap.

**CI platform checklist** (solves problem P3 — the ecosystem uses **one** CI platform, chosen in Phase 0 via a spike):

| Requirement | Why | Example implementations |
|---|---|---|
| Mandatory pipeline imposed by central policy, pinned by ref, which PRs cannot modify or bypass | The verifier is outside the PR author's control | GitHub org ruleset + required workflow; GitLab pipeline execution policy; Jenkins trusted shared library + admin-managed job |
| OIDC workload identity for jobs | Keyless signing; cloud access without static secrets | GitHub `id-token`; GitLab `id_tokens`; OIDC provider plugin for Jenkins |
| Dedicated trusted runners for the attest job | The attest job never shares a machine with PR code | Protected/tagged runners, dedicated runner pool |
| Attestation signing | Evidence cannot be forged | Sigstore keyless when the OIDC issuer is accepted; **self-hosted CI**: private Sigstore, or a KMS key accessible only to the attest runner's identity |
| Evidence store outside the repo, with retention | Only the attest job can write | Object storage with identity-based write policy |

Mandatory pipeline features are often tied to higher license tiers — the Phase 0 spike must confirm them on the exact tier that will be used.

<a id="s8-8"></a>

## 8.8. Test data, test environments and stability

### Test Data Management

| Data layer | Used in | How created | Reset |
|---|---|---|---|
| Scenario fixture (mock) | Unit, component, E2E on PRs | Generated from OpenAPI + versioned scenario files ([§4.2](04-technical-foundation.md#s4-2)), fixed seed so deterministic | Not needed: each test has its own in-memory copy |
| Synthetic data | Preview and staging | Data builders in `@platform/testing`, generated from the contract schema and `x-data-class` ([§19.7](19-nfr-enterprise.md#s19-7)), fixed seed per run | Per the test tenant lifecycle (below) |
| Reference data (catalog, price list, roles) | Staging | Versioned seed owned by the backend team | When the seed changes version |
| Canary data | Smoke in production | Read-only canary accounts | No writes |

- **No production data** in tests, including masked copies. Exceptions require an ADR and privacy owner approval.
- Personas are generated from the **IAM role catalog** (owner, admin, viewer, billing-only, no-access, support agent, impersonating admin).
- Versioned scenario fixtures per journey (resource states, errors, quota, partial failure); shared pack scenarios live in `@platform/testing` ([§8.11](#s8-11)).
- Staging credentials: short-lived tokens from a secret broker, never in the repo or in agent context.
- **Creation and reset on staging.** The backend provides a **test-data API** (committed in Phase 0, [§17](17-decisions-to-make.md#s17)): provision test tenants, seed per scenario, and delete. Each E2E run leases a tenant from a **test tenant pool**, seeds data per scenario, prefixes every resource with the run ID, and returns the tenant when done. A cleanup job deletes resources past their TTL (*suggested* 24 h).
- **Tenant isolation.** Each parallel worker uses its own tenant; cross-tenant tests (`tenant.context-isolation`, `search.tenant-scope`) use a pair of tenants leased together. No test depends on data created by another test.
- **Simulated auxiliary systems:** mail sink, payment gateway sandbox, mock streaming server for LLMs ([§6.6](06-product-patterns.md#s6-6)), each with an owner and a health check.

### Test Environment Management

| Environment | Lifecycle | Backend | Owner |
|---|---|---|---|
| Local / agent sandbox | Per working session | Mock | User or agent |
| CI (PR) | Per job | Mock (MSW) | Frontend Platform team |
| Preview | Created when a PR is opened, updated on each push, destroyed on merge or close, or after an inactivity TTL (*suggested* 7 days) | Mock by default; test tenant on staging when the surface declares the need | Frontend Platform team (template), infrastructure (running) |
| Staging | Shared, long-lived | Real backend at the version about to go to production | Backend/infra environment owner |
| Production canary | Continuous | Production, read-only | Vertical owner |

- **Environment manifest.** Each run on preview or staging records the version of each backend service, the data seed and the flag configuration it ran with, and puts the digest into the evidence (`x-fe.environment`, [§8.5](#s8-5)). As a result, a result always knows what it was checked against.
- **Pre-run health check.** `factory env status` checks the environment and auxiliary systems before running. If the environment is unhealthy the run returns `ERROR` (not `FAIL`), does not count against the PR and does not consume repair budget.
- **Failure attribution.** Each `FAIL` or `ERROR`, once triaged, is tagged `product`, `test`, `environment`, `test-data` or `harness`. The rate per category is a Testing Platform metric ([§13](13-metrics.md#s13)) and determines who must fix it ([§14.7](14-organization-and-operations.md#s14-7)).
- **When the environment is unstable.** Recurring environment failures are opened as incidents for the environment owner. If staging is unhealthy beyond the threshold (*suggested* 2 h continuously), the pre-production gate ([§8.1](#s8-1)) waits; it is never skipped. Emergency patches follow break-glass ([§11.8](11-delivery-change-management.md#s11-8)).
- **Staging contention.** Release candidates use staging via a queue, with per-surface locks, so that two test runs do not corrupt each other's data.
- **Smoke in production** only uses read-only canary accounts.

### Flakiness (Standard §7.14)

- At most one automatic retry, with both results recorded. Fail-then-pass only counts as `flaky` **when** the test or check is in the flaky registry, which is generated from intermittent failures on `main` (*suggested* window: the last 30 days). Otherwise the result is `FAIL`: it is new non-determinism, especially when the test covers touched code.
- **Stress mode.** Before review, tests of touched code in timing-sensitive areas are run repeatedly (*suggested* 5×, Playwright `--repeat-each`). **When to enable 20×:** deferred until timing defects escape or the flaky rate exceeds the agreed threshold ([§15.3](15-roadmap.md#s15-3)). Timing-sensitive areas include `cap:realtime-presence`, `cap:streaming-response`, `state.lro-lifecycle`, `forms.double-submit`, and anything using timers or retries.
- **Quarantine** is decided only by the owner of the test or check, never by the agent working on the task. Quarantined tests still run and report, are excluded from blocking and from triggering repair, and have an owner and a fix-by date. A quarantined test that protects a `critical` journey requires a replacement verifier or a disposition from an independent person. A quarantined `required` check is `NOT_ASSESSED` and requires a disposition.
- A retry ending in `ERROR` is an infrastructure event, not a flake. The flaky rate is a metric ([§13](13-metrics.md#s13)) and an L3 criterion ([§7.5](07-factory-metadata-registry.md#s7-5)).

<a id="s8-9"></a>

## 8.9. Wave 1 check plan (suggested)

This plan must cover **the full resolved profile** of the Wave 1 surfaces, because that is an L3 criterion ([§7.5](07-factory-metadata-registry.md#s7-5)). Wave 1 (Console + Admin + DS) activates about **85 checks**:

| Group | Number of checks | Note |
|---|---|---|
| Standard core | 15 | 14 integrity and policy checks, cheap because they only scan the diff, plus `knowledge.impact`; `integrity.mutation-score` stays in shadow |
| `baseline` | 16 | Includes `sec.server-cache-scope` |
| `operational` | 8 | |
| `x:a11y`, `x:security-sensitive` | 3 + 3 | |
| `x:privileged-admin` | 7 | Includes 2 `cross-team-contract` checks (`admin.step-up-auth`, `admin.network-exposure`) |
| `x:i18n`, `x:perf-app`, `x:theming` | 3 + 1 + 1 | Per the Console composition recipe ([§2.6](02-portfolio-scope.md#s2-6)) |
| `design-system` | 8 | Excluding `ds.docs-api-sync` and `docs.code-samples` |
| Foundation packs | ≈ 11 | `workflow-state`, `search-filter`, `bulk-actions`, `forms-wizard`, `audit-trail`, `notifications`, `import-export`; prioritizing permissions, tenancy, destructive operations, CSV injection |
| NFR at the merge gate | 4 | `supply.provenance`, `supply.sbom`, `api.problem-details`, `browser.support-matrix` |
| Pre-production gate | 5 | `sec.dast-staging`, `test.staging-smoke`, `perf.load-test`, `contract.can-i-deploy`, `ops.readiness-review` |

If `console/billing` is part of Wave 1, `pay.hosted-fields` is additionally activated. Outside this plan, the profile only allows two groups in `shadow` in Wave 1, and does not count them against the budget: `integrity.mutation-score`, and the Documentation & Knowledge checks (`ds.docs-api-sync`, `docs.code-samples` for DS, `meta.agents-md-drift`, `knowledge.record-schema`, `knowledge.references`, `docs.freshness`). They are considered for activation once FP data is available.

<a id="s8-10"></a>

## 8.10. Test Planner: test planning and orchestration

Profile resolution ([§8.3](#s8-3)) answers "**which checks** apply". The Test Planner answers the next question: "**which tests**, on which **environment**, with which **browsers**, at which **stage**". The two concerns are split into two components:

| Component | Role | Lives in |
|---|---|---|
| **Test Planner** (`factory plan`) | Deterministic. Takes inputs and produces a **TestPlan** with a rationale for each selection. Does not run tests | Factory CLI, version pinned by the policy repo; runs in the verify job of the mandatory pipeline, never via repo scripts |
| **Runner** | Executes the TestPlan: shards, runs Vitest/Storybook/Playwright/Pact via check adapters, reports `CheckResult` | Check adapters in the policy repo |
| **Evidence** | Stores the TestPlan digest (`x-fe.testPlan`) alongside results, to show which tests were selected and why | Evidence store ([§8.7](#s8-7)) |

**When to enable.** Test Planner v0 (Phase 1b) is a Factory CLI module, consisting only of the floor, the import graph (affected) and the surface map; cross-repo input arrives with the portfolio index in Phase 2. The parts that make it a product of its own (per-test coverage map, Test Asset Catalog as input, selection-miss metric and test selection eval) are deferred until there is a measured need: defects slipping through a PR's plan are detected in nightly runs or incidents, or run time exceeds the command budget ([§7.6](07-factory-metadata-registry.md#s7-6), [§15.3](15-roadmap.md#s15-3)). The design below is the target state.

**Planner inputs:**

| Input | Contribution |
|---|---|
| Changed files | Starting point |
| Repo import graph (Nx/Turborepo affected) | Modules and tests that depend on the changed files |
| Portfolio index + Token Usage Index (monorepo option: project graph + Token Usage Index) | Consumer repos or projects when a shared package or token changes ([§5.5](05-design-system.md#s5-5), [§11.2](11-delivery-change-management.md#s11-2)) |
| Surface map (`factory.yaml`) | Affected surfaces, rendering profile, criticality |
| Journey Registry + route map | Journeys passing through the touched routes or modules |
| Test Asset Catalog + per-test coverage map (*proposed*, generated nightly; only once enabled, see the “When to enable” paragraph) | Which tests execute which files |
| Resolved profile ([Appendix A](appendix-a-resolve.md#appendix-a)) | Which checks are required, at which gate |
| Risk sub-tier ([§10.1](10-autonomy-policy.md#s10-1)) | Breadth of the plan |
| Flaky registry ([§8.8](#s8-8)) | Which tests run with registered retries |

**Selection rules (the planner only adds, never removes below the floor):**

1. The floor is always included: every test directly touching a changed file; every test of `critical` journeys on affected surfaces, for every declared persona and scenario (on Chromium, per rule 4); the sentinel set ([§5.5](05-design-system.md#s5-5)) when a shared asset changes.
2. When the planner cannot map a changed file (coverage map missing or stale) → run the full suite of that surface. When unsure, run wider, never narrower.
3. By sub-tier: R0/R1 use the minimal plan; R2 adds all `high` journeys of the surface; R3 runs the full suite of affected surfaces on every browser in the matrix.
4. Browsers (default):
   - **Each push:** Chromium. `critical` journeys run full persona × scenario.
   - **Merge queue:** `critical` journeys only, adding the happy path of each persona on WebKit and Firefox, and blocking merge; browser-sensitive code (IME and composition, CSS layout, `cap:rich-text`, `cap:streaming-response`, `embedded-ui`) runs all three browsers for affected tests.
   - **Nightly and release candidate:** the full three-browser matrix for every journey.
   - **When to enable the full matrix in the merge queue** (`critical` journeys with full persona × scenario × three browsers): deferred until nightly runs show browser-specific defects escaping into main ([§15.3](15-roadmap.md#s15-3)).
5. Agents or humans may add tests to the plan (`--include`); no one can remove tests from the floor. The planner configuration lives in the policy repo.
6. The merge queue reuses the PR's plan, recomputed on the latest main.

**Measuring test selection quality** (once enabled, see the “When to enable” paragraph). Nightly runs execute the full suite. A test that fails in nightly and would have failed on the PR that caused the defect, but was not selected in that PR's plan, is a **selection miss**. The selection miss rate is a Testing Platform metric ([§13](13-metrics.md#s13)); a selection miss on a `critical` journey opens a work item for the Quality Engineering lead.

**Standard commands and the planner.** `verify:fast` = plan with static + unit scope on affected files; `verify` = all static/unit/integration checks of the resolved profile; `verify:runtime` = E2E, a11y and visual for affected journeys. `factory plan --json` prints the plan and the rationale for each selection, so humans and agents can preview what will run.

<a id="s8-11"></a>

## 8.11. Test asset lifecycle

A **test asset** is an execution vehicle for verification: suites, test cases, scenarios, fixtures, personas, mocks, visual baselines, seeded defects and reports. A **check** is policy: what must hold, at which gate, with which enforcement. The two are related but governed differently:

| | Check / policy ([§8.2](#s8-2)) | Test asset |
|---|---|---|
| Lives in | Check Registry, policy repo | The repo owning the asset: a product repo, or `@platform/testing` for shared assets |
| Owner | Check owner | Surface owner, or Quality Engineering for shared assets |
| Changed via | Policy repo review pipeline | Regular PR; subject to integrity checks ([§8.6](#s8-6)) |
| Lifecycle | `planned → shadow → active → deprecated` | Create → use → modify → retire (below) |
| Relationship | One check consumes many assets (e.g., `test.e2e-journeys` runs every journey-tagged test) | One asset may serve many checks |

**Identity, version, owner, storage and reuse:**

| Asset | Identity | Version | Owner | Storage | Reuse |
|---|---|---|---|---|---|
| Suite | By surface and layer (`console/compute:e2e`) | By repo commit | Surface owner | Product repo | — |
| Test case | Stable tags `@journey`, `@scenario`, `@ac`; other tests: path + name, with renames detected by the Test Asset Catalog | By commit | Surface owner | Product repo | Via `@platform/testing` helpers and page objects |
| Scenario fixture | `<journey>/<scenario>` | By the contract version the mocks are generated from | Journey owner | Product repo (scenario file) | Per-pack scenario templates in `@platform/testing` (SemVer) |
| Persona | By role in the IAM role catalog | By catalog version | IAM team (catalog); Quality Engineering (generator) | Generated into `@platform/testing` | Shared across all repos |
| Mock | Generated from the contract | By contract version | No one edits by hand (FE-TEST-004) | Build artifact | Shared |
| Visual baseline | Story or journey + theme + viewport + browser | By commit; approved only by the code owner (FE-TEST-002) | Surface owner; DS maintainer for DS stories | *Proposed:* visual testing service or Git LFS (ADR) | — |
| Seeded defect | `SD-<area>-<n>`; dev and hold-out sets | Versioned, hold-out rotated quarterly ([§12](12-evaluation.md#s12)) | Agent/Eval engineer | Separate eval repo, restricted read access, outside product repos | Used for every repo of the same archetype |
| Report and evidence | `VerificationRun` ID | Immutable | Attest job | Evidence store, per the retention policy ([§17](17-decisions-to-make.md#s17)) | Looked up via `factory trace` |

The **Test Asset Catalog** is generated (nightly and on merge) from tags, scenario files, the Journey Registry, the coverage map and the evidence store, like the portfolio index; there is no hand-written catalog, and no catalog service ([§7.1](07-factory-metadata-registry.md#s7-1)). It feeds the Test Planner ([§8.10](#s8-10)), `factory trace` ([§7.7](07-factory-metadata-registry.md#s7-7)) and `factory doctor`. **When to enable:** deferred until `factory trace` v0 (reading tags and the evidence store) and `factory doctor` are no longer sufficient, and only after an existing developer portal or catalog has been evaluated ([§17](17-decisions-to-make.md#s17), [§15.3](15-roadmap.md#s15-3)).

**Retirement.** Tests for journeys or scenarios removed from the registry are reported by `factory doctor` as orphaned. Deleting a test is always recorded by `integrity.test-count-delta` and must reference the deprecation of the corresponding journey, AC or feature. Fixtures and scenario templates no longer in use are reported by gardening ([§12](12-evaluation.md#s12)). Shared assets in `@platform/testing` are deprecated per SemVer with a codemod, like every shared package.

---

[Contents](README.md) · [← 7. Factory: metadata, registries, repository standard](07-factory-metadata-registry.md) · [9. Agent Operating Model →](09-agent-operating-model.md)
