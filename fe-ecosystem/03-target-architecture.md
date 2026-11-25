[Contents](README.md) · [← 2. System scope & portfolio model](02-portfolio-scope.md) · [4. Technical foundation →](04-technical-foundation.md)

<a id="s3"></a>

# 3. Target architecture

<a id="s3-1"></a>

## 3.1. The big picture

```text
┌───────────────────────────────── Users ─────────────────────────────┐
│ Customers · Internal staff · Editors · Public · Developers           │
└─────────────┬───────────────────────────────────────┬────────────────┘
              │ CDN / edge                             │ CDN / edge
┌─────────────▼─────────────────────────┐ ┌───────────▼────────────────┐
│ APP REPOS (one vertical owner each)   │ │ SITE REPOS (vertical owner) │
│ console · admin · desk · studio ·     │ │ web · docs                  │
│ chat · dms — Next.js full-stack       │ │ Next.js SSG/ISR             │
│  ├─ UI: Server + Client Components    │ └───────────┬────────────────┘
│  └─ Server layer = the app's own BFF  │             │ content API · webhook
│     (session · service calls ·        │ ┌───────────▼────────────────┐
│      mutations · streaming)           │ │ Headless CMS               │
│     via @platform/server              │ └────────────────────────────┘
└─────────────┬─────────────────────────┘
              │ OpenAPI / AsyncAPI — generated clients, server-side only
┌─────────────▼───────────────────────────────────────────────────────┐
│ Backend services: IAM · compute · network · billing · ticket · AI · …│
└──────────────────────────────────────────────────────────────────────┘

PLATFORM REPO   (Frontend Platform team): Design System · app shell · @platform/* · presets · generators · Factory CLI
POLICY REPO     (Frontend Platform team): mandatory pipeline · check adapters · profiles · registry · RiskMap
PORTFOLIO INDEX (generated): descriptors + lockfiles + import/token scans of every repo → consumer graph

SHARED CAPABILITIES (cut across every repo above; not separate apps or services — §3.8)
  TESTING & QUALITY ENGINEERING : @platform/testing · runners · Test Planner · test env & data · Test Asset Catalog (generated)
  DOCUMENTATION & KNOWLEDGE     : sources stay in repos / contract registry / CMS · Knowledge Index (generated, retrieval only)
                                  · publish channels: Docs Portal · Storybook · factory context/search
```

The diagram depicts the polyrepo option of [§3.3](#s3-3); with the monorepo option, APP REPOS, SITE REPOS, PLATFORM REPO and `content-models` merge into a single `frontend` repo, the POLICY REPO stays separate, and the PORTFOLIO INDEX remains only for consumers outside the repo. The two shared capabilities add no boxes to the user's runtime flow; they sit across the repos and run through the Factory. Boundaries, data flows and integration points are in [§3.8](#s3-8).

<a id="s3-2"></a>

## 3.2. One framework, one golden path

**Next.js (App Router) is the only golden path** for every app and site. Differences between products are expressed through a **rendering profile** declared per surface — not through different frameworks or templates:

| Rendering profile | Used for | Rendering approach | Cache | Note |
|---|---|---|---|---|
| `authenticated` | Console, Admin, Desk, Studio, Chat, DMS | Server Components + streaming; interaction via Client Components + query cache | No cache shared between users; tenant data is `private, no-store` | Data fetched server-side via `@platform/server` ([§3.5](#s3-5)) |
| `public-static` | Website (landing, pricing, products, campaigns) | SSG/ISR | CDN; revalidate on deploy and on CMS publish | Core Web Vitals take priority |
| `docs` | Docs Portal | SSG/ISR + MDX; API reference generated from OpenAPI | CDN | Search; versioning only when needed |

The `app` generator accepts `--profile`; an app may have surfaces belonging to multiple profiles (for example `docs/account` is `authenticated` inside the docs repo). The profile determines the template, the cache/header preset and the default checks.

**Why only one golden path:** one build system, one set of kits/checks/evals, one set of skills for every team; Server Components make it possible to keep tokens and call services server-side without operating a separate BFF.

**Accepted cost:** Next.js changes significantly between major versions and has a server surface that requires discipline. The platform team owns the common upgrade schedule and codemods ([§4.8](04-technical-foundation.md#s4-8)); server discipline is enforced through `@platform/server` and the FE-NEXT rules ([§3.5](#s3-5), [§4.5](04-technical-foundation.md#s4-5)). Changing frameworks is a portfolio-level ADR.

<a id="s3-3"></a>

## 3.3. Repository topology

Repo topology is a decision **settled by ADR in Phase 0** ([§17](17-decisions-to-make.md#s17)), based on a spike on the actual CI platform. The design describes two options. The rest of the document is the same for both: **the policy repo is always separate** so that the verifier sits outside the PR author's control ([§8.7](08-verification.md#s8-7)); the module layout, standard commands, per-surface descriptors ([§7.2](07-factory-metadata-registry.md#s7-2)) and per-module CODEOWNERS do not change.

**Option A — polyrepo** (one repo per app):

| Repo | Contains | Owner |
|---|---|---|
| **platform** (the platform team's internal monorepo) | `ds-tokens`, `ds-react`, `ds-icons`, `ds-patterns`, `app-shell`, `@platform/server`, `@platform/auth`, `@platform/api-codegen`, `@platform/testing`, `i18n`, `telemetry`, `flags`, `caps/*`, lint/TS/test presets, generators, Factory CLI | Frontend Platform team |
| **policy** | Mandatory pipeline, check adapters, profiles, registry (IDs, schemas), RiskMap, trust registry | Frontend Platform team |
| **console** | The Console shell + modules following the Console Module Contract ([§3.4](#s3-4)); each module is owned by the corresponding service team via CODEOWNERS | Console owner + owner of each module |
| **admin**, **desk**, **studio**, **chat**, **dms** | One Next.js app per repo | Corresponding vertical owner |
| **web**, **docs** | One Next.js site per repo | Corresponding vertical owner |
| **content-models** | CMS content schemas → automatically generated types, used by studio/web/docs | Owner of Content Studio |

Every product repo has **the same layout and the same commands** ([§7.6](07-factory-metadata-registry.md#s7-6)), is created with a generator, uses presets and foundation packages from the platform repo as **versioned packages**, and runs the **mandatory pipeline** from the policy repo. Inside multi-module repos (Console) and the platform repo, Nx/Turborepo is used for affected builds and tag boundaries.

**Option B — product monorepo:** one **frontend** repo containing every app, site, platform package and `content-models`, plus a separate **policy** repo. Each app is an Nx/Turborepo project with tags `app:*`, `scope:*`, `type:*`, `surface:*`; boundaries between apps and between modules are enforced by lint boundaries ([§4.4](04-technical-foundation.md#s4-4)) and CODEOWNERS. Each app still has its own deployment pipeline (running only when the app is affected), owner, SLO and release owner.

**Comparison.** A monorepo provides out of the box some capabilities that a polyrepo must build itself:

| Concern | A. Polyrepo solves it with | B. Monorepo solves it with |
|---|---|---|
| Who uses which package/token | **Portfolio index** combining descriptors + lockfiles + import scans + Token Usage Index of every repo ([§5.5](05-design-system.md#s5-5)) | Project graph + Token Usage Index within one repo. The portfolio index remains only for consumers outside the repo (the `docs/` directory of service repos, SDKs) |
| Changing shared code together with every consumer | **Prerelease + consumer verification** → impact report → release → bot opens adoption PRs with codemods ([§11.2](11-delivery-change-management.md#s11-2)) | One atomic PR modifies the package and runs codemods on every consumer; resolve selects affected surfaces via the project graph |
| Version drift between apps | **Version skew policy** ([§4.8](04-technical-foundation.md#s4-8)): support window, version-bump bot, `factory doctor` blocks when the threshold is exceeded | One version for every app. A major upgrade is a migration program with codemods; when simultaneous upgrade is impossible, a time-boxed transition window per app is used |
| One place to apply policy | **Mandatory pipeline** from the policy repo applied to every repo ([§8.7](08-verification.md#s8-7)) | Mandatory pipeline on one repo |
| Scoped read access | Per repo | Every frontend engineer can read all frontend code |
| API contract changes | Bot opens regenerate PRs in each consumer repo ([§11.3](11-delivery-change-management.md#s11-3)) | One regenerate PR for every consumer |
| Main risks | Cost of building and operating compensating mechanisms; version drift; slow cross-repo changes | CI and merge queue carry the load of every team (requires affected builds, remote cache, batching and bisection); a broken main affects every team; a single PR has a larger blast radius |

**Selection criteria:**

1. **Read-access restrictions.** If there is a genuine requirement (security, external partners or contractors) to restrict code read access between verticals → A.
2. **CI capacity.** Whether the CI platform chosen in Phase 0 can run affected builds, remote cache and a merge queue with batching for a portfolio-sized repo within the time budget of [§7.6](07-factory-metadata-registry.md#s7-6). No → A. The Phase 0 spike measures this.
3. **Platform staffing.** B eliminates the cross-repo portfolio index, prerelease consumer verification, the adoption PR bot and the version skew bot ([§15.3](15-roadmap.md#s15-3)). With 6–8 FTE ([§15.1](15-roadmap.md#s15-1)), this is the largest saving in the entire design.
4. **Independent deployment cadence** is not a criterion: both options give each app its own deployment pipeline.

Criterion 1 not met and criterion 2 met → **B**. The option-dependent sections, including [§4.8](04-technical-foundation.md#s4-8), [§5.5](05-design-system.md#s5-5), [§8.10](08-verification.md#s8-10), [§11.2–11.3](11-delivery-change-management.md#s11-2), [§15.3](15-roadmap.md#s15-3) and [Appendix A](appendix-a-resolve.md#appendix-a), describe option A and note the differences for option B.

<a id="s3-4"></a>

## 3.4. Console Module Contract

The Console is a collection of many services (compute, storage, network, IAM, billing, support…). Each service is a **module** following the same contract:

```typescript
export const module = defineConsoleModule({
  id: "compute",
  owner: "team-compute",
  routes: () => import("./routes"),            // lazy
  nav: { section: "infrastructure", label: "nav.compute", icon: "server" },
  permissions: ["compute.instances.list"],     // hide the module without permission
  featureFlag: "module.compute",
  surfaces: ["console/compute"],
  assistantTools: () => import("./assistant"), // tools for the AI assistant (optional)
});
```

The app shell reads the manifest to build navigation, routes and permission gating. Benefits: adding a new service becomes repeatable work (which generators and agents do well), UX is always consistent, and the module is a clear unit of ownership and boundary. Micro Frontends are considered only when a module genuinely needs an independent deployment schedule.

<a id="s3-5"></a>

## 3.5. Server layer in each app (integrated BFF)

Each app with the `authenticated` profile is a full-stack Next.js app; **the app's server part is its BFF**, built entirely on `@platform/server` and `@platform/auth`, owned by the platform team:

| Capability | Standard |
|---|---|
| Session | OIDC with the IdP via `@platform/auth`; tokens stay on the server; the browser holds only a `__Host-` cookie with `HttpOnly; Secure; SameSite=Lax` — the BFF pattern recommended for OAuth in browser-based applications |
| Service calls | Clients generated from OpenAPI are **importable only on the server** (`server-only`); Server Components and route handlers call through them; the browser never calls services directly |
| Mutation | Server actions or route handlers wrapped by `@platform/server`: authentication, origin/CSRF checks, input validated against schema, audit correlation ID. "Bare" server actions are prohibited |
| Authorization | Checked in the **server-side data access layer** for each request, never relying on middleware. Lesson from CVE-2025-29927: Next.js middleware could once be bypassed with an internal header |
| Data to the client | Only filtered DTOs cross the Server → Client Component boundary; tokens, raw service objects or internal fields are never passed |
| Streaming | SSE route handlers for Chat AI and assistants; connection limits, idle timeout, heartbeat, cancel upstream when the client disconnects |
| Protection | CSP with nonce; security headers; rate limiting; audit |

**Operations:**
- **The vertical owner operates** its app's Next.js server: SLO, on-call, capacity — following the **platform's template** (suggested SLO: availability ≥ 99.95% for Console/Admin; health checks, metrics and traces built into `@platform/server`; sample runbooks for IdP outage, slow upstream, session errors).
- Stateless server: session lives in an encrypted cookie or a shared store; horizontal scaling; multi-zone.
- Fault isolation: per-upstream timeouts and circuit breakers, built into the generated client.
- **No business logic** in the server layer: only composition, DTO mapping, permission checks; business rules live in the backend.
- Admin has its own repo, origin and session; shorter sessions, step-up auth.
- Changes to `@platform/server`/`@platform/auth` are R3 (`x:security-sensitive`) because every app depends on them.

<a id="s3-6"></a>

## 3.6. Deployment & environments

| Component | Deployment | Environments |
|---|---|---|
| App (`authenticated`) | Immutable Next.js container image (standalone output) behind a CDN; static assets on the CDN | preview per PR · staging · production |
| Site (`public-static`, `docs`) | SSG/ISR, edge cache; **automatic revalidation on deploy and on CMS publish** | same as above |
| Feature flags | OpenFeature standard, centralized provider | per environment |

Each vertical has its own deployment pipeline but **goes through the same mandatory pipeline** ([§8.7](08-verification.md#s8-7)). Canary/staged rollout for surfaces with `criticality` `high` or `critical`; rollback = redeploy the previous image.

<a id="s3-7"></a>

## 3.7. Self-hosted Next.js at scale

Running Next.js as multiple stateless instances ([§3.6](#s3-6)) requires explicit configuration that a single-instance or managed deployment usually hides. The items below are requirements for `@platform/server` and the app template. Exact option names are confirmed against the Next.js version pinned in Phase 0.

| Concern | Risk if ignored | Standard |
|---|---|---|
| ISR and data cache | Each instance keeps its own cache, so users see different versions, and on-demand revalidation (for example on CMS publish) reaches only one instance | A **shared cache handler** (Next.js custom cache-handler configuration) backed by a shared store. Tag-based revalidation goes through it, so a single call invalidates every instance |
| Server Actions encryption | Instances encrypt action payloads with different keys, so actions fail intermittently behind the load balancer | One `NEXT_SERVER_ACTIONS_ENCRYPTION_KEY` per environment, taken from the secret manager, identical on every instance, and rotated through the release process |
| Deployment skew | Browsers still running an old build call Server Actions or load chunks that no longer exist, after a deployment or during a canary | One `deploymentId` per build. Static assets of the previous build are retained on the CDN for at least the session lifetime (*suggested* 24 h). On version mismatch, the next navigation triggers a hard reload. An unknown Server Action returns a typed error that the app shell turns into a reload prompt, never a silent failure |
| Server-side data cache | Per-tenant or per-user data enters the shared cache handler via `fetch` cache, `unstable_cache` or `'use cache'` with a key that contains neither tenant nor user, and is then served to someone else. CDN headers are still correct, so `sec.cache-headers` does not catch it | In `authenticated` rendering, data fetched from services defaults to `no-store` in the data cache. Caching goes through the `@platform/server` helper, which automatically puts tenant, project, region and (when the data depends on the user) user into the key and tags. Apps do not call the Next.js cache APIs directly; per-request `React.cache()` is allowed (FE-NEXT-005, check `sec.server-cache-scope`) |
| CDN caching | Tenant data is cached at the edge and served to other users | Authenticated responses use `Cache-Control: private, no-store`. The CDN never caches responses that set or depend on the session cookie (check `sec.cache-headers`) |
| Streaming and shutdown | SSE streams are cut on every deployment | Graceful shutdown drains connections within a timeout limit; the client resumes or shows a clear error ([§6.6](06-product-patterns.md#s6-6)) |
| Health | The load balancer routes traffic to instances whose upstreams are down | Separate liveness and readiness endpoints in `@platform/server`. Readiness includes the IdP and critical upstreams, with circuit breakers |
| Reproducibility | Instances built from different inputs behave differently | One immutable image per commit. Configuration comes only from the environment and the flag provider; nothing is built at container startup |

<a id="s3-8"></a>

## 3.8. Shared capabilities: Testing & Quality Engineering, Documentation & Knowledge

**Principle.** These two capabilities are **platform capabilities**, implemented as packages, conventions, automatically generated indexes, CLI commands and adapters in the mandatory pipeline, living in the existing repos ([§3.3](#s3-3)). They are not two applications or two microservices. A component is split out into a service only when there is a genuine operational reason (for example: a search engine, a backend test-data API, preview environments), and doing so requires an ADR. Consistent with [§0.4](00-summary.md#s0-4), the Factory does not build its own runners, search engines or catalogs: the indexes here are **generated** from sources, like the portfolio index.

**Boundary.**

| | Testing & Quality Engineering | Documentation & Knowledge |
|---|---|---|
| Purpose | Turn requirements and risks into runnable tests, run the right tests at the right time, on the right environment and data, and produce trustworthy evidence | Ensure people and agents find the right documentation, the right version, at the right level of authority, with source citations, before changing anything |
| Includes | Test frameworks and harnesses, Test Planner ([§8.10](08-verification.md#s8-10)), test asset management ([§8.11](08-verification.md#s8-11)), test data and test environments ([§8.8](08-verification.md#s8-8)), traceability from requirements to evidence ([§7.7](07-factory-metadata-registry.md#s7-7)), quality of the tests themselves ([§13](13-metrics.md#s13)) | Documentation classification and sources of truth ([§7.3](07-factory-metadata-registry.md#s7-3)), Knowledge Records, component documentation ([§5.6](05-design-system.md#s5-6)), Docs Portal ([§6.8](06-product-patterns.md#s6-8)), Knowledge Index and retrieval for agents ([§7.8](07-factory-metadata-registry.md#s7-8)), documentation freshness ([§13](13-metrics.md#s13)) |
| Excludes | Check policy and enforcement: belong to the Check Registry and profiles ([§8.2–8.3](08-verification.md#s8-2)). Approval: belongs to the autonomy policy ([§10](10-autonomy-policy.md#s10)) | Authoritative decisions: belong to `ACTIVE` records and their owners. Marketing editorial content: belongs to Content Studio ([§6.4](06-product-patterns.md#s6-4)) |
| Source of truth | Test code, scenarios and fixtures in the repos that own them; Journey Registry; Check Registry for policy; evidence store for results | The origin of each documentation type ([§7.3](07-factory-metadata-registry.md#s7-3)): Git, contract registry, SDKs, CMS, issue tracker |
| Generated components | **Test Asset Catalog**: inventory of tests, scenarios, fixtures, journeys, and their links | **Knowledge Index**: index linking records, contracts, SDKs, components, owners, versions; for retrieval only, not authoritative |
| Where it lives | `@platform/testing` and presets in the platform repo; adapters, Test Planner and flaky registry in the policy repo; test assets in product repos | Conventions, generators and the Knowledge Index in the platform repo; documentation at its source; the Docs Portal is the `docs` repo |
| Owner | Quality Engineering lead ([§14.2](14-organization-and-operations.md#s14-2)) | Documentation & Knowledge lead ([§14.2](14-organization-and-operations.md#s14-2)) |

**Data flows and integration points with the Factory.**

```text
 Requirement (issue tracker) ──► Task contract (doneWhen AC) ──► Journey Registry (journey · scenario · AC IDs)
                                         │                                  │
                                         ▼                                  ▼
 ┌──────────────── TESTING & QUALITY ENGINEERING ──────────────────────────────────────────┐
 │ Test Planner: diff + import graph + portfolio index + surfaces + journeys + risk → plan   │
 │ Runners (via check adapters): Vitest · Storybook · Playwright · Pact                     │
 │ Test env & data: mock · preview · staging tenant pool · synthetic data                    │
 │ Test Asset Catalog (generated)                                                            │
 └───────┬──────────────────────────────────────────────┬──────────────────────────────────┘
         │ CheckResult + plan + env manifest             │ coverage / orphans / staleness
         ▼                                               ▼
 Evidence store (attested) ──► PolicyDecision     factory doctor · gardening · metrics (§13)
         ▲
         │ governing records · contracts · component docs (with version + authority + citation)
 ┌───────┴──────────────── DOCUMENTATION & KNOWLEDGE ──────────────────────────────────────┐
 │ Sources: Knowledge Records · docs-as-code · OpenAPI/AsyncAPI · SDK · DS docs · CMS        │
 │ Knowledge Index (generated, access-aware, retrieval only)                                 │
 │ Publish: Docs Portal · Storybook / DS manifest · factory context / factory search         │
 └──────────────────────────────────────────────────────────────────────────────────────────┘
```

| Integration point with the Factory | Testing & Quality Engineering | Documentation & Knowledge |
|---|---|---|
| Task lifecycle ([§9.3](09-agent-operating-model.md#s9-3)) | `PLANNED`: test plan; `VERIFYING`: execute the plan; oracle locked per [§8.6](08-verification.md#s8-6) | `CONTEXT_READY`: `factory context` and `factory search` return the governing sources with citations |
| Evidence ([§8.5](08-verification.md#s8-5)) | `x-fe.testPlan`, `x-fe.environment`, journey/scenario/AC links | Citations of sources (URI@SHA) used by the agent, recorded in the task log |
| Check ([§8.4](08-verification.md#s8-4)) | `test.*`, `integrity.*`, `stream.*`… | `docs.*`, `ds.docs-api-sync`, `meta.agents-md-drift`, `knowledge.*` |
| Portfolio index | Cross-repo consumers for test selection ([§8.10](08-verification.md#s8-10)) | Graph of owners, contracts, packages used for the Knowledge Index ([§7.8](07-factory-metadata-registry.md#s7-8)) |
| Eval ([§12](12-evaluation.md#s12)) | Seeded defects, selection misses, flaky | Retrieval eval, agent-legibility eval |

**Current design status of each capability** (to know what exists, what is still missing, and which decisions must go to an ADR or spike):

| Capability | Design level | Owner | Dependencies | Decision / spike |
|---|---|---|---|---|
| Test toolchain and harness ([§4.1](04-technical-foundation.md#s4-1)) | Designed; tools are proposals | Quality Engineering lead | CI platform ([§8.7](08-verification.md#s8-7)) | Test stack ADR in Phase 0 |
| Checks, evidence, integrity ([§8.2–8.7](08-verification.md#s8-2)) | Designed | Quality & policy engineer | Standard 4.0 | — |
| Test Planner and test selection ([§8.10](08-verification.md#s8-10)) | Designed at boundary level; algorithm not settled | Quality Engineering lead | Portfolio index, Journey Registry; coverage map once enabled | v0 in Phase 1b uses only the import graph; precision spike using the coverage map when there is a measurable need ([§15.3](15-roadmap.md#s15-3)) |
| Requirement → evidence traceability ([§7.7](07-factory-metadata-registry.md#s7-7)) | Designed at schema level | Quality Engineering lead + product owner | Issue tracker | ADR: requirement ID format and tracker integration |
| Test asset lifecycle ([§8.11](08-verification.md#s8-11)) | Designed at convention level | Quality Engineering lead | Platform repo, evidence store | ADR: where visual baselines are stored |
| Test data and test environments ([§8.8](08-verification.md#s8-8)) | Designed at requirement level; backend commitment **still missing** | Quality Engineering lead + environment owner | Backend test-data/tenant API, preview infrastructure | Phase 0 ADR: mechanism for provisioning test tenants and seeding data; preview environment spike |
| Documentation sources of truth ([§7.3](07-factory-metadata-registry.md#s7-3)) | Designed | Documentation & Knowledge lead | — | — |
| Component documentation ([§5.6](05-design-system.md#s5-6)) | Designed | DS maintainer | Storybook manifest | — |
| Docs Portal ([§6.8](06-product-patterns.md#s6-8)) | Designed at information-architecture level | Owner of the Docs Portal | Contract registry, SDK releases, CMS | ADR: whether internal documentation shares the Docs app or uses a separate channel; search engine |
| Knowledge Index and retrieval for agents ([§7.8](07-factory-metadata-registry.md#s7-8)) | Designed at requirement and rule level | Documentation & Knowledge lead | Access permissions of source systems, LLM data policy ([§19.7](19-nfr-enterprise.md#s19-7)) | Evaluate an existing developer portal before building ([§17](17-decisions-to-make.md#s17)); Phase 2 spike: lexical + filter vs semantic; build vs buy ADR |

---

[Contents](README.md) · [← 2. System scope & portfolio model](02-portfolio-scope.md) · [4. Technical foundation →](04-technical-foundation.md)
