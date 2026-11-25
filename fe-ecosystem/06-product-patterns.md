[Contents](README.md) · [← 5. Design System](05-design-system.md) · [7. Factory: metadata, registries, repository standard →](07-factory-metadata-registry.md)

<a id="s6"></a>

# 6. Patterns by product

<a id="s6-1"></a>

## 6.1. Console

| Pattern | Standard |
|---|---|
| Context | Organization → Project → Region; the URL is the source of truth; changing context invalidates the cache; never display data from the previous context |
| IAM-aware UI | Permissions come from the backend; no permission → hide/disable with a reason; permission matrix tests are generated from the IAM role catalog |
| Long-running operation | Operation ID, pending/progress/failure, idempotent retry, double-submit protection |
| Resource state machine | Available actions are declared per state (`cap:workflow-state`) |
| Error taxonomy | Validation, permission denied, quota, conflict, throttled, partial failure, unavailable — standard component + message |
| Destructive action | Confirmation proportional to the blast radius (type the resource name), consequences displayed |
| Units, time, money | Consistent units, time zones and locale-based formatting; prices come from the pricing API |

<a id="s6-2"></a>

## 6.2. Admin (`x:privileged-admin`, R2 floor, privileged path R3)

Separate repo, origin and session; internal network/zero-trust; not indexed; no third-party scripts · **Step-up auth** for sensitive actions · **Impersonation:** reason + ticket ID, always-visible banner, time-limited, read-only by default, audit "admin X acting as Y" · **Break-glass** is time-limited and approved · **Cross-tenant bulk:** dry-run + scope preview + typed confirmation · **Four-eyes** for the highest-risk actions · **PII masking** by default, per-field reveal with audit · Exports are limited, audited and protected against CSV injection.

<a id="s6-3"></a>

## 6.3. Service Desk

| Pattern | Standard |
|---|---|
| Two views, two contracts | Customers and agents use different APIs/DTOs; internal fields are **never** present in responses to customers |
| Internal note | Visually distinct; test: the customer persona never sees an internal note through the **UI, API, email, notification or export** |
| SLA | The deadline is computed by the server; the client only counts down |
| Collision | Optimistic locking; "X is replying"; conflicts are resolved through the UI |
| Email content | Strong sanitization, isolated rendering, images through a proxy |
| Macro | Escape variables, always preview before sending |
| Ticket creation | Form + attachment; article suggestions from the Docs Portal |

<a id="s6-4"></a>

## 6.4. Content Studio & content flow

- **Default: a mature headless CMS** (build-vs-buy recorded in an ADR), extended with typed blocks/schemas; do not build a CMS engine.
- The content model is a contract (`content-models` repo); changing the model is a reviewed code change.
- **Content is a type of change** — see [§11.4](11-delivery-change-management.md#s11-4) (verify-on-publish).
- Revision, restore, scheduled publish, locale fallback, preview = published.

<a id="s6-5"></a>

## 6.5. Public Website & Docs

- The site fetches content from the CMS at build/ISR time; revalidate on the publish webhook.
- SEO: metadata, canonical, sitemap/robots, structured data where used, hreflang.
- Consent: no non-essential tracking before consent; third-party scripts follow an allowlist.
- Docs: MDX, API reference **generated from OpenAPI**, code samples compiled/run, versioning only when truly needed. The system-level architecture of the Docs Portal is in [§6.8](#s6-8).

<a id="s6-6"></a>

## 6.6. Chat AI

| Pattern | Standard |
|---|---|
| Streaming | SSE through the app's route handler ([§3.5](03-target-architecture.md#s3-5)); efficient append; auto-scroll but respect the user scrolling up; Stop/Regenerate; resume or show a clear error when the connection is lost |
| Render | Markdown → HTML **through a sanitizer**; never raw HTML; links show the domain, `rel="noopener noreferrer"`; images through proxy/CSP |
| Composer | **IME-safe** (Vietnamese Telex/VNI, CJK): never send during composition — do not rely only on `isComposing` (Safari may fire Enter right after `compositionend` with `isComposing=false`); combine `keyCode 229` with a composition flag; test on WebKit, Chromium, Firefox |
| Conversation | Rename, pin, delete (real deletion, including in the API), search, revocable sharing, retention per policy |
| Tool call | Show each step; actions with side effects **MUST be confirmed by the user**; content from files/the web is untrusted data |
| Errors | Quota, rate limit, filtered, context exceeded, model unavailable |
| Privacy | Conversations/files never go into telemetry, session replay or error logs |
| A11y | `polite` live region announces on completion (does not read each token); focus after send/stop; `prefers-reduced-motion` |
| Verification | **Mock streaming server** with deterministic fixtures (slow, mid-stream error, XSS payload, tool call, blocked). Answer quality is the AI team's eval, not a frontend concern |

<a id="s6-7"></a>

## 6.7. Document Management

Upload/download with integrity checks (hash), type/size limits, malware scanning (backend), safe preview, version/restore, approval/signing through `cap:workflow-state`, audit, retention/deletion per policy (`cap:records-retention`).

<a id="s6-8"></a>

## 6.8. Docs Portal: system-level architecture

The Docs Portal is the **publishing channel** of the Documentation & Knowledge capability for external users ([§3.8](03-target-architecture.md#s3-8)). It aggregates content from multiple sources of truth but does not own that content: fix at the source, never on the published copy.

**Published document types.**

| Type (per Diátaxis, *proposed*) | Example | Source of truth ([§7.3](07-factory-metadata-registry.md#s7-3)) |
|---|---|---|
| Tutorial, how-to | Get started with compute, configure IAM | Docs-as-code (MDX) in the `docs` repo or the `docs/` directory of the service repo |
| Reference | API reference, SDK reference, CLI reference, limits and quotas | OpenAPI/AsyncAPI from the contract registry; SDK release; CLI manifest |
| Explanation | Concepts, resource model, security model | Docs-as-code |
| Changelog, deprecation, announcements | Release notes, API deprecation schedule | Generated from work items and the contract registry; editorial announcements from the CMS |
| Troubleshooting | Common errors, error codes | Docs-as-code; error codes generated from the RFC 9457 `type` registry ([§19.6](19-nfr-enterprise.md#s19-6)) |

Internal technical documents (ADRs, runbooks, platform standards, module documentation) are **not** published to the public Docs Portal. They are found through the Knowledge Index ([§7.8](07-factory-metadata-registry.md#s7-8)). Whether an authenticated internal surface (for example `docs/internal`) is needed in the same app is a Phase 0 decision ([§17](17-decisions-to-make.md#s17)).

**How content reaches the portal.** Each source publishes a versioned **docs bundle** (MDX directory + manifest: product, service, audience, type, applicable version, owner, source and commit). The Docs Portal build pulls the bundles at the pinned versions, plus contracts from the contract registry and SDK reference from SDK releases, then records the **provenance** of each page (source, commit, version). A service repo changes its docs through a PR in that same repo, in the same PR as the code change when the two must ship together.

**Organization.**

- Organizing dimensions: **product → service → audience → type → version**. URLs are stable in the form `/<product>/<service>/<type>/<slug>`; only versioned reference has the `/v<major>/` segment.
- Audience: customers, developers, partners. A page meant for a single audience is labeled, not duplicated.
- Version: API reference keeps every supported major; guides have only the current version, unless instructions differ between versions. Pages of an end-of-support version get a banner and are excluded from default search results.

**Cross-ecosystem search.** One search index for the entire Docs Portal, with facets for product, service, type and version. The same index provides article suggestions when creating a ticket ([§6.3](#s6-3)) and contextual help in the Console (by the open surface). Only public content goes into this index; internal search goes through the Knowledge Index ([§7.8](07-factory-metadata-registry.md#s7-8)). The search tool is a proposal, decided in Phase 0.

**Verification.** The `docs-portal` profile ([§8.4](08-verification.md#s8-4)) plus: `docs.source-provenance` (every page has a source and commit), `docs.api-reference-sync` (reference matches the deployed contract), `docs.code-samples` (samples run with the correct SDK version), `links.integrity`, and `docs.freshness` (pages whose source has changed since the last review, at `advisory`).

---

[Contents](README.md) · [← 5. Design System](05-design-system.md) · [7. Factory: metadata, registries, repository standard →](07-factory-metadata-registry.md)
