[Contents](README.md) · [← 3. Target architecture](03-target-architecture.md) · [5. Design System →](05-design-system.md)

<a id="s4"></a>

# 4. Technical foundation

<a id="s4-1"></a>

## 4.1. Proposed stack (default ADRs, pending confirmation in Phase 0)

Every tool in the table is a **proposal**, not yet a settled decision; each row becomes a decision when the corresponding ADR moves to `ACTIVE` after a spike in Phase 0 ([§17](17-decisions-to-make.md#s17)).

| Layer | Default | Note |
|---|---|---|
| Language | TypeScript `strict` | No plain JS in new code |
| Framework | **Next.js (App Router)** + React | The only golden path; rendering profile per surface ([§3.2](03-target-architecture.md#s3-2)) |
| Server helpers | `@platform/server`, `@platform/auth` | Session, service calls, mutation, streaming ([§3.5](03-target-architecture.md#s3-5)) |
| Data | Server-side data fetching via `@platform/server`; TanStack Query for client-side interaction | Cache keys **MUST** contain tenant/project/region |
| A11y primitives | Mature headless library (React Aria or Radix) | No hand-written focus management |
| Styling | CSS variables from tokens + one unified styling approach | Theme = CSS variable overrides |
| Form | Form engine + schema validation generated from OpenAPI | |
| Unit/integration | Vitest + Testing Library | |
| Component | Storybook (interaction, a11y, visual) | |
| E2E | Playwright (Chromium, WebKit, Firefox) | |
| Mock | MSW, generated from OpenAPI + scenario fixtures | |
| Contract | Pact (consumer-driven) + broker | `can-i-deploy` before deploy |
| Package manager / build | pnpm; Nx or Turborepo in the platform repo and multi-module repos (Console) | Affected builds, tag boundaries |
| CI | **One** platform that meets the [§8.7](08-verification.md#s8-7) checklist | Mandatory pipeline from the policy repo, OIDC, separate attestation runner |
| Telemetry | OpenTelemetry (web) + error tracking + RUM | Never record user content |
| i18n | ICU message format, `vi` + `en` at minimum | |
| Flags | OpenFeature | Every flag has an owner, default, fallback, `safeState`, expiry |

**Testing Platform: what the platform provides, what product teams own.** The toolchain above is only the tooling part. The shared Testing Platform is organized into three layers, and each layer has a clear owner:

| Layer | Components | Owner | What product teams do |
|---|---|---|---|
| **Framework & harness** | Vitest/Storybook/Playwright presets; `@platform/testing` (render helpers, persona login, page objects for the app shell and DS, data builders, a11y matchers); mocks generated from OpenAPI ([§4.2](#s4-2)); mock streaming server; scenario templates per pack ([§2.4](02-portfolio-scope.md#s2-4)); test generator for journeys ([§7.7](07-factory-metadata-registry.md#s7-7)) | Frontend Platform team (Quality Engineering) | Use, do not fork. Propose extensions through the standard channel ([§14.6](14-organization-and-operations.md#s14-6)) |
| **Orchestration & execution** | Test Planner ([§8.10](08-verification.md#s8-10)), runners and sharding in the mandatory pipeline, flaky registry, provisioning of test tenants and environments ([§8.8](08-verification.md#s8-8)), reporters and evidence, Test Asset Catalog ([§8.11](08-verification.md#s8-11)) | Frontend Platform team (Quality Engineering), together with the backend/infra environment owner | Do not write their own pipelines or test-running scripts ([§8.7](08-verification.md#s8-7)) |
| **Product test assets** | Test code, journeys and scenarios in `journeys.yaml`, domain fixtures, persona ↔ permission mappings, surface visual baselines, regression tests from bugs | **Vertical owner** of the surface | Write, review and maintain; fix their own failing and flaky tests; decide on quarantining their own tests ([§8.8](08-verification.md#s8-8)) |

Boundary rule: anything that recurs in ≥ 2 verticals (helpers, pack scenarios, personas) is moved up to the framework layer; anything containing the domain knowledge of a single product stays in the product repo. The platform team is accountable when tests fail because of the shared harness, runner or environment; the vertical owner is accountable when tests fail because of the product's code or tests ([§14.7](14-organization-and-operations.md#s14-7)).

<a id="s4-2"></a>

## 4.2. Contract-first

- Every service the frontend calls **MUST** publish a versioned OpenAPI (HTTP) or AsyncAPI (event) contract. This is a **prerequisite** for backend teams, recorded in an ADR in Phase 0; while the backend has not yet published, the interim `consumer-authored` path below is used.
- Each repo generates from the contract (version pinned in `factory.yaml`) with `@platform/api-codegen`: typed clients (server-side only), MSW handlers + scenarios, form schemas, Pact consumer tests. Clients are never hand-written or copied between repos.
- Calling services with hand-written `fetch` is **prohibited** (rule FE-API-001).
- **Interim path when the backend has not yet published OpenAPI.** FE may write the OpenAPI spec for that service, or derive the spec from traffic recorded on staging; the backend service owner reviews the spec, then the spec is stored in the contract registry with `authority: consumer-authored` (a contract registry entry field, distinct from `provider-published`; not the `authority` of a Knowledge Record). Clients, mocks and Pact consumer tests are generated from it like any contract, so FE-API-001 still applies. When the backend publishes its own contract, the `consumer-authored` entry is replaced (`supersededBy`), clients are regenerated, and the differences are handled as a contract change ([§11.3](11-delivery-change-management.md#s11-3)).
- The CMS content model is also a contract: schema → automatically generated types → type errors when the model changes.

<a id="s4-3"></a>

## 4.3. State ownership

| State | Source of truth | Approach |
|---|---|---|
| Server state | Backend | Client-side query cache and server-side data cache; keys include tenant/project/region (FE-CTX-001, FE-NEXT-005) |
| Long-running operation | Operation API | Tracker + polling/events, reconcile |
| URL state | Router | Context, filters, pagination — deep-linkable |
| Local UI | Component | Local state |
| Shared client state | App | Store only when truly needed |
| Form | Feature | Form engine + schema |
| Realtime | Event stream | Adapter + cache synchronization, reconnect |
| Persistent client | Browser, per policy | Do not store tokens/PII; clear on logout/tenant switch |

<a id="s4-4"></a>

## 4.4. Module architecture

```text
<repo>/src/modules/<domain>/   (Console)   or   <repo>/src/domains/<domain>/   (other apps)
├── domain/        # pure business rules, no React dependency
├── application/   # use cases, orchestration
├── server/        # server-side data access: contract-generated clients, permission checks, DTO mapping (server-only)
├── data/          # client-side query hooks (calling the app's route handlers)
├── features/      # screens, flows
├── ui/            # domain-specific components
├── AGENTS.md
└── index.ts       # the only public interface
```

Boundaries use tags (`scope:*`, `type:*`, `surface:*`) and are enforced by lint rules in the platform preset.

<a id="s4-5"></a>

## 4.5. Rule catalog

*(Each ID carries exactly one meaning, consistent with the Standard's frontend rules. The categories `NEXT`, `CTX`, `DS`, `I18N` and `CFG` are registered profile extensions, [§18.3](18-aisf-conformance.md#s18-3).)*

| ID | Rule | Enforced by |
|---|---|---|
| FE-ARCH-001 | Do not import another module's private implementation | `arch.boundaries` |
| FE-ARCH-002 | The presentation layer (`features/`, `ui/`) does not import generated clients or infrastructure adapters. Server Components go through `server/`; Client Components go through `data/` | `arch.boundaries` |
| FE-ARCH-003 | No circular dependencies | `arch.cycles` |
| FE-ARCH-004 | `domain/` does not import React or browser APIs | `arch.boundaries` |
| FE-ARCH-005 | Shared packages are Server Component compatible: interactive components are marked `'use client'`, and server-only APIs are never imported into client components | `arch.boundaries`, `build.compile` |
| FE-ARCH-006 | `@platform/*` packages contain no product or domain logic and never depend on product repos | `arch.boundaries` |
| FE-API-001 | Call services only through contract-generated clients (`@platform/api-codegen`) | `api.client-contract`, `quality.lint` |
| FE-NEXT-001 | Contract-generated clients, secrets and tokens are used only in `server-only` modules | `quality.lint`, `sec.secrets` |
| FE-NEXT-002 | Server actions and route handlers exist only through the `@platform/server` wrapper (authentication, origin/CSRF, validation, audit) | `quality.lint` |
| FE-NEXT-003 | Authorization is checked in the server-side data access layer; middleware is not an authorization layer | `authz.negative`, `sec.authz-tests` |
| FE-NEXT-004 | Only filtered DTOs cross the Server → Client Component boundary | `quality.lint`, `quality.typecheck` |
| FE-NEXT-005 | In `authenticated` rendering, the server-side data cache is used only through the `@platform/server` helper with keys and tags containing tenant, project, region (and user when needed); do not call `fetch` cache, `unstable_cache` or `'use cache'` directly for service data ([§3.7](03-target-architecture.md#s3-7)) | `quality.lint`, `sec.server-cache-scope` |
| FE-CTX-001 | Query cache keys contain tenant, project and region | `tenant.context-isolation` |
| FE-PERM-001 | Frontend permissions only reflect backend authorization, never replace it. Every permission-sensitive action has server-side enforcement and a negative test | `authz.permission-matrix`, `authz.negative` |
| FE-SEC-001 | Do not render raw HTML outside the designated sanitizer | `quality.lint`, `chat.render-sanitization` |
| FE-SEC-002 | No tokens, secrets or PII in bundles, logs or storage | `sec.secrets`, `privacy.conversation-telemetry` |
| FE-DS-001 | Tokens are used through a typed API: no dynamic token-name concatenation, no hard-coded color or spacing values | `quality.lint`, `ds.token-schema` |
| FE-STATE-001 | Server state is not duplicated into global client state without a recorded reason | `quality.lint` |
| FE-STATE-002 | Primitives that require finesse are allowed only inside platform-approved wrappers: raw `useEffect` for fetching or synchronizing data, and `setTimeout`/`requestAnimationFrame` used to mask ordering problems | `quality.lint` |
| FE-UX-001 | Interactive controls meet the keyboard and accessibility baseline and are built on the headless a11y library | `a11y.static-lint`, `a11y.keyboard-journeys` |
| FE-FLAG-001 | Flags go through the OpenFeature adapter; every flag has an owner, default, fallback, `safeState` (`off` \| `on`, [§11.4](11-delivery-change-management.md#s11-4)) and expiry; flags gating `x:security-sensitive` behavior MUST declare `safeState` explicitly | `quality.lint`, gardening |
| FE-DEP-001 | New dependencies MUST pass policy (license, vulnerabilities, maintenance, size) | `deps.policy` |
| FE-I18N-001 | No hard-coded user-facing strings | `i18n.missing-keys`, `quality.lint` |
| FE-CFG-001 | Config files generated from presets MUST NOT be edited by hand; override via `factory.config.yaml` ([§8.7](08-verification.md#s8-7)) | `meta.config-parity` |
| FE-CODE-001 | Do not write comments to justify workarounds: encode the constraint as a rule with an actionable remediation message, or fix the root cause. Starts at `observe` | `integrity.suppressions` (comment detector) |
| FE-TEST-001 | Every `critical` journey has runtime evidence (E2E) for every declared persona and scenario | `test.e2e-journeys` |
| FE-TEST-002 | Updating visual and snapshot baselines requires code owner approval; an agent never accepts its own baselines | `integrity.snapshot-churn` |
| FE-TEST-003 | From R2 upward, acceptance scenarios are defined and reviewed before implementation and are read-only for the implementer (Standard §6.4.1, §7.13) | `integrity.oracle-lock` |
| FE-TEST-004 | Generated mocks (MSW handlers) are never edited by hand; changing an existing scenario fixture counts as editing an expected value | `integrity.self-mocking`, `integrity.expected-value-edit` |

Test oracle integrity beyond these rules is governed by the Standard's integrity checks ([§8.6](08-verification.md#s8-6)). Each rule has an ID, intent, owner, enforcement, pass and fail fixtures, and a remediation message.

<a id="s4-6"></a>

## 4.6. Security baseline

- Sessions through the app's server layer ([§3.5](03-target-architecture.md#s3-5)); origin/CSRF checks for every mutation; strict CSP (nonce/hash), `frame-ancestors` blocking clickjacking for apps.
- XSS: a single sanitizer for rich text, HTML email, markdown and AI output.
- postMessage: origin + schema checks; open redirect: allowlist.
- Third-party scripts: allowlist, SRI where possible; **none** on Admin.
- Supply chain: lockfile, provenance, SBOM (CycloneDX) for released packages.
- No public source maps in production.
- The full enterprise-grade security baseline (SAST/DAST, threat model, Trusted Types, session policy, edge protection) is in [§19.1](19-nfr-enterprise.md#s19-1).

<a id="s4-7"></a>

## 4.7. Performance budget

- Public: Core Web Vitals at p75 field data against Google's "good" thresholds (LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1) — reference target.
- App: bundle budget per route, time to main interaction; virtualize long lists.

<a id="s4-8"></a>

## 4.8. Technology lifecycle

Solves problem P1 — the platform MUST NOT drift back into a fragmented, out-of-support state:
- Each foundation dependency (React, Next.js, headless a11y, test stack, Node runtime) has an **owner** and a **support window**: always on the current major or N-1 while upstream still releases security patches.
- Gardening tracks the EOL schedule: ≤ 2 quarters remaining → upgrade work item with a deadline; past EOL → `deps.policy` blocks release (waiver of at most 30 days, approved by the security owner).
- **Version skew policy:** each repo uses Next.js, React and `@platform/*` within the support window published by the platform (suggested: the Next.js and React major is the major the platform currently supports, i.e. current or N-1 as in the line above; `@platform/*` at most N-2 minor versions behind the latest). When upstream releases a new major, the window does not change immediately: it shifts to the new major only once the platform has released presets, `@platform/*` and codemods for that major, and each repo gets adoption time according to the common schedule (line below). A bot automatically opens upgrade PRs; `factory doctor` blocks release when a repo is outside the window; exceptions require an ADR and a deadline. *Monorepo option ([§3.3](03-target-architecture.md#s3-3)): every app uses one version; there is no cross-repo version-bump bot, and a major upgrade is a migration program with codemods.*
- Major upgrades (for example Next.js) follow the **common portfolio schedule**: the platform upgrades presets and `@platform/*` first and releases codemods; a bot opens `codemod.apply` PRs in each repo; vertical owners merge within the agreed window.

---

[Contents](README.md) · [← 3. Target architecture](03-target-architecture.md) · [5. Design System →](05-design-system.md)
