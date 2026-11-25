[Contents](README.md) · [← 18. AISF Standard 4.0 conformance (profile declaration)](18-aisf-conformance.md) · [20. Showcase plan →](20-showcase.md)

<a id="s19"></a>

# 19. Enterprise NFR baseline

Each requirement has an owner, a way to verify it (a check or a registry record), and the phase from which it becomes mandatory. The gate of each check (PR/merge, pre-production, post-release monitoring, or periodic) is mapped in [§8.1](08-verification.md#s8-1). *Suggested* values are decided in Phase 0 ([§17](17-decisions-to-make.md#s17)).

<a id="s19-1"></a>

## 19.1. Security

| Requirement | Verification | From |
|---|---|---|
| Threat model (STRIDE) per archetype, updated when a pack or data class is added; reviewed by the security owner | Knowledge Record `kind: threat-model`, `ACTIVE` | Phase 1b |
| SAST on every PR (CodeQL or Semgrep rules for TypeScript, React and Next.js); critical and high findings are never baselined | `sec.sast` | Phase 1a |
| DAST on staging: baseline scan plus authenticated scan for critical journeys | `sec.dast-staging` (pre-production gate, [§8.1](08-verification.md#s8-1)) | Phase 2 |
| Secret scanning and dependency vulnerabilities | `sec.secrets`, `deps.policy` | Phase 1a |
| CSP with nonce and a reporting endpoint; Trusted Types are mandatory on surfaces with `cap:rich-text`, `cap:threaded-conversation` or `x:ai-surface` | `sec.headers-csp`, `ai.output-sanitization` | Phase 2 |
| Session policy: idle and absolute timeouts per app (shorter for Admin), refresh token rotation, sign out everywhere, revocation on role change | `sec.authz-tests` scenarios; IdP configuration record | Phase 1b |
| Security headers: HSTS, `frame-ancestors`, `Referrer-Policy`, `Permissions-Policy`, COOP | `sec.headers-csp` | Phase 1b |
| Edge protection: WAF, bot management, rate limiting per route group | Configuration record plus a smoke test | Phase 2 |
| Penetration test before each app's GA and annually; for Admin and billing, every release train that contains R3 changes | Pentest report record | Phase 2 |
| Vulnerability remediation SLA (*suggested*: critical ≤ 7 days, high ≤ 30 days) | Gardening report; overdue items block release | Phase 1b |

<a id="s19-2"></a>

## 19.2. Supply chain

- Target **SLSA Build L3** for published `@platform/*` packages and app images, with SLSA L2 (`supply.provenance`) from when attestation is enabled in the Phase 1a follow-up.
- npm traffic goes through a registry proxy with an allowlist. Installs follow the lockfile only (`build.install-frozen`), and the lockfile is linted for registry and integrity.
- CI actions and images are pinned by digest and updated by Renovate.
- Each release produces a CycloneDX SBOM and stores it with the artifact (`supply.sbom`). Published packages carry npm provenance.
- A new dependency MUST go through FE-DEP-001, including the license allowlist.

<a id="s19-3"></a>

## 19.3. Resilience and disaster recovery

A surface uses a single `criticality` enum: `critical` \| `high` \| `normal`, the same as journeys ([§7.2](07-factory-metadata-registry.md#s7-2), [§7.7](07-factory-metadata-registry.md#s7-7)).

| Applies to | Example | RTO | RPO | Requirement |
|---|---|---|---|---|
| `criticality: critical` | Console authentication and billing flows, Admin | ≤ 1 h | ≤ 15 min (session store) | Multi-zone; tested cross-region failover plan; IdP-outage mode (clear banner, no data loss) |
| `criticality: high` | Console, Desk, Chat, DMS modules | ≤ 4 h | ≤ 1 h | Multi-zone |
| `criticality: normal` with `authenticated` rendering | Settings pages, auxiliary pages | ≤ 8 h | ≤ 4 h | Per the platform template |
| `public-static` or `docs` rendering (any criticality) | Website, Docs | ≤ 15 min | n/a | CDN `stale-if-error`; static fallback page |

*Suggested* values. Frontend apps are stateless, so RPO applies to the session store and the place where drafts are stored. Each app rehearses DR annually and rollback quarterly (an L4 criterion).

<a id="s19-4"></a>

## 19.4. SLOs and error budgets

- Every surface with `criticality` `high` or above declares SLOs in `factory.yaml`. *Suggested:* 99.95% availability for Console and Admin; route load time p75 and INP for the app's routes; Core Web Vitals p75 "good" for public surfaces; error rate.
- Error budget policy: when a surface's budget is exhausted, A0 stops on that surface ([§10.6](10-autonomy-policy.md#s10-6)), only reliability fixes are shipped, and a postmortem is held.
- SLOs are measured from RUM and server-side telemetry via `@platform/telemetry`, with dashboards generated automatically per surface.

<a id="s19-5"></a>

## 19.5. Browser and device support

- *Suggested:* the two latest major versions of Chrome, Edge, Firefox, Safari (macOS and iOS) and Samsung Internet; minimum viewport 360 px. The final matrix is decided from RUM data in Phase 0.
- This matrix determines the Playwright projects and the browserslist target in the preset; `browser.support-matrix` checks that they match.
- An unsupported browser receives a clear notice, never a broken app.

<a id="s19-6"></a>

## 19.6. API error model

- Every service and server layer returns errors as **RFC 9457 Problem Details** (`application/problem+json`), with a registered `type` for each entry of the error taxonomy ([§6.1](06-product-patterns.md#s6-1)). Generated clients map them to typed errors, and DS components display them.
- `api.problem-details` lints the OpenAPI contracts.

<a id="s19-7"></a>

## 19.7. Privacy and compliance

- Every field in a contract has a data classification (`x-data-class`: public, internal, personal, sensitive; mapped to the Standard §10.6 `dataClass` in [§18.2](18-aisf-conformance.md#s18-2), a field that cannot be classified is treated as `restricted`). The generated DTO filter uses this classification (FE-NEXT-004).
- Personal data processing complies with Vietnam's Personal Data Protection Law (effective 2026-01-01) and the guiding documents in force. The list of applicable instruments, including how far Decree 13/2023/NĐ-CP remains in force, is confirmed by the legal and compliance owner in Phase 0 ([§17](17-decisions-to-make.md#s17), item 18) and recorded as a Knowledge Record `kind: policy`; this design does not itself conclude on the validity of each instrument. Scope includes consent records and data subject requests end to end (export and deletion), including chat conversations and documents (`records.retention-deletion`).
- Data residency is declared per data class for telemetry, logs, evidence and LLM providers (Standard §10.2).
- Audit trails are retained per policy; the Admin audit trail is immutable.
- If SOC 2 or ISO 27001 applies, [§10.5](10-autonomy-policy.md#s10-5) and [§8.7](08-verification.md#s8-7) are mapped to their change-management controls, and the compliance owner confirms the mapping before Slice 0.

<a id="s19-8"></a>

## 19.8. Accessibility

- WCAG 2.2 AA (legal level confirmed in Phase 0), plus ATAG for Content Studio.
- The automated part comes from the `x:a11y` checks. An accessibility specialist manually audits every `critical` journey before GA and annually thereafter; the audit result is recorded as an attestation in the `PolicyDecision`.
- Every public product has an accessibility statement and a feedback channel, and a11y bugs are triaged within an SLA.

<a id="s19-9"></a>

## 19.9. Release management

- The change calendar has freeze windows: no A0 during a freeze, and R3 changes only with the release owner's approval.
- Release notes per app are generated from work items, plus a customer-facing changelog for Console and Docs.
- Hotfixes use the same pipeline through a priority queue and never bypass required checks (break-glass only per [§11.8](11-delivery-change-management.md#s11-8)).
- Deprecation policy for user-visible features and public URLs: notice period and redirects.

<a id="s19-10"></a>

## 19.10. Observability

- OpenTelemetry traces run from the browser through the server layer to the services, with a correlation ID that also appears in audit events.
- Logs are structured and contain no user content. Error tracking strips PII. Session replay is off on `x:privileged-admin` and `x:ai-surface`.
- Dashboards and alerts are generated per surface from `factory.yaml`.

<a id="s19-11"></a>

## 19.11. Production readiness review

Before a vertical's first production release on the platform, the following MUST be in place: SLOs and alerts; an on-call rotation trained on the server layer runbooks (IdP outage, slow upstream, session errors, deployment skew); load testing at 2× the expected peak; a DR plan; and a security review. This is recorded as `ops.readiness-review`, mandatory before each app's first production deployment.

---

[Contents](README.md) · [← 18. AISF Standard 4.0 conformance (profile declaration)](18-aisf-conformance.md) · [20. Showcase plan →](20-showcase.md)
