[Contents](README.md) · [1. Principles →](01-principles.md)

<a id="s0"></a>

# 0. Summary

<a id="s0-1"></a>

## 0.1. The problem

The current ecosystem does not constrain the design, but it clearly shows what needs to be solved:

| # | Problem | Source | Consequence | Solution |
|---|---|---|---|---|
| P1 | **Framework fragmentation**: part uses modern React/Next.js, part uses Vue 2/Nuxt 2 + webpack 4, which are out of support (Vue 2 EOL 2023-12-31, Nuxt 2 EOL 2024-06-30) | Observed | No more security patches; each stack needs its own kit, checks, golden path and evals; components and skills cannot move between teams | One framework — Next.js ([§3.2](03-target-architecture.md#s3-2)); technology lifecycle ([§4.8](04-technical-foundation.md#s4-8)) |
| P2 | **One repo per product with different tooling** (pnpm/yarn/npm, Vitest/Jest, separate lint configurations) | Observed | Nobody knows who uses what; cross-product changes need multiple coordinated PRs; agents must relearn each repo | One repository standard, foundation packages and presets released by the platform, standard commands, portfolio index ([§3.3](03-target-architecture.md#s3-3), [§7.6](07-factory-metadata-registry.md#s7-6), [§8.7](08-verification.md#s8-7)) |
| P3 | **Fragmented CI** across multiple platforms and multiple ways of writing pipelines | Observed | No single point to apply policy; evidence cannot be signed consistently | One CI platform that meets the separated-verifier checklist ([§8.7](08-verification.md#s8-7)) |
| P4 | **Home-built CMS** for the website | Observed | Cost of maintaining an engine; lacks standard revision/preview/workflow; published content is never verified | Headless CMS, content model as contract, verify-on-publish ([§6.4](06-product-patterns.md#s6-4), [§11.4](11-delivery-change-management.md#s11-4)) |
| P5 | The same functions (attachment, workflow, search, export) are rebuilt in each product | Typical | Repeated security defects (XSS, CSV injection, tenant data leakage) | Shared capability packs + DS + checks ([§2.4](02-portfolio-scope.md#s2-4), [§8.4](08-verification.md#s8-4)) |
| P6 | AI agents are used without boundaries, evidence or policy | Typical | Quality cannot be measured; nobody dares to raise the autonomy level | Factory: verification, evidence, autonomy policy ([§8–§10](08-verification.md#s8)) |

<a id="s0-2"></a>

## 0.2. Definition

The system consists of two tightly coupled parts:

1. **Frontend Platform** — a unified technical foundation (one language, one framework — Next.js, one repository standard, one Design System, one app shell, API-first contracts) on which every frontend product is built.
2. **AI Software Factory** — the operating system that helps people and AI agents develop, verify and release changes on that platform, with the autonomy level decided by **policy that lives outside the model**, based on evidence.

> **One platform — one standard — many products — verification by surface — autonomy by evidence.**

The two parts are designed together for one simple reason: **the Factory is only strong when the platform is uniform.** Every additional framework, every additional way of organizing repos or calling APIs multiplies kits, checks, golden paths and evals.

**Relationship to the Standard.** The Factory part of this design is **AISF Frontend Profile 1.1**: the first profile of AI Software Factory Standard 4.0 and the showcase that demonstrates the Standard. The Standard owns the common law: vocabulary, risk and autonomy policy, separation of duties, evidence schema, integrity rules and conformance levels. This profile adds what is specific to frontend: surfaces, archetypes, capability packs, rendering profiles, the check catalog, the frontend stack and stricter thresholds. It does not loosen the Standard; the single deviation is declared and pending approval by the Standard's owner ([§18.1](18-aisf-conformance.md#s18-1)).

**Ownership model.** Each product/service is owned by a **vertical owner**: repo, domain, features, operations. The **Frontend Platform team** owns what is shared and what ensures quality: standards, quality gates, the Factory, the Design System, the app shell and foundation packages. The platform team does not build product features; it provides the **paved road** and the **quality gates** that every vertical passes through ([§14](14-organization-and-operations.md#s14)).

**Two shared capabilities.** **Testing & Quality Engineering** and **Documentation & Knowledge** are designed as **platform capabilities** shared by every vertical and every agent, not as two separate applications or services. Each capability has its own boundary, owner, source of truth, lifecycle and integration points with the Factory ([§3.8](03-target-architecture.md#s3-8)). The Docs Portal is only one publishing channel of Documentation & Knowledge; checks in the pipeline are only one consumption point of Testing & Quality Engineering.

<a id="s0-3"></a>

## 0.3. Goals

1. Every frontend product shares the platform, Design System and app shell; each vertical is autonomous in its product, and uniform in ways of working and quality gates.
2. Every interaction with the backend goes through a **machine-readable contract** (OpenAPI/AsyncAPI/schema), from which clients, types, mocks and contract tests are generated.
3. Agents understand the product, surface, contracts, ownership and policy **before** changing anything, and can verify both code and runtime behavior themselves.
4. Every change — code **and content** — carries evidence appropriate to the affected surface.
5. The impact of shared changes (components, tokens, APIs, content models) is computed from the graph, not guessed.
6. Agent autonomy levels are granted per task class based on measured evidence, and are revoked when an incident occurs.
7. Quality, flow, cost and developer experience are measured against a baseline.

<a id="s0-4"></a>

## 0.4. Out of scope

- No parallel UI framework in the target state.
- No Micro Frontends by default; only when there is a genuine independent-deployment requirement (ADR).
- No home-built browser automation, test runner, catalog, scanner or CMS engine when mature tools exist. The Factory's portfolio index, registries (Journey, Check, trust), Test Asset Catalog and Knowledge Index are **files or indexes generated in Git and CI**, not catalog services ([§7.1](07-factory-metadata-registry.md#s7-1)); before building the Knowledge Index or Test Asset Catalog, an existing developer portal or catalog (for example Backstage) is evaluated ([§17](17-decisions-to-make.md#s17)).
- No use of AI review in place of deterministic checks or human decisions where those are required.
- No auto-merge merely because CI passes; no collapsing quality into a single score.
- No measuring by number of PRs, lines of code or number of agents.
- Out of scope: native mobile, desktop apps.

<a id="s0-5"></a>

## 0.5. Definition of success

1. Creating a new app/site repo or module with a generator → immediately reaches L2 ([§7.5](07-factory-metadata-registry.md#s7-5)), with the mandatory pipeline, checks, AGENTS.md, telemetry, i18n and auth.
2. An engineer assigns a task as **outcome + done checks**; the Factory determines the surface, context, playbook and profile itself; the agent works in a sandbox; evidence is complete; policy decides approval.
3. A token/component/API change is traced to every consumer **across repos** via the portfolio index and selectively verified, with no regression escaping in known consumers.
4. Content published from the CMS is verified like code.
5. At least one task class reaches auto-merge (A0) without the escaped defect rate rising above baseline.
6. No foundation dependency is past EOL; no frontend product sits outside the platform and the common policy.
7. Every product repo publishes a conformance claim against AISF Standard 4.0: Slice 0 is the AISF pilot with a pilot claim (L1 plus the pilot's list of CORE-IDs, [§20.2](20-showcase.md#s20-2)); generator-created repos reach L2 from creation, starting at the end of Phase 1b; Wave 1 reaches L3 at Phase 2 exit. The showcase scenarios in [§20](20-showcase.md#s20) pass on real repositories.

---

[Contents](README.md) · [1. Principles →](01-principles.md)
