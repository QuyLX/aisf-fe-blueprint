[Contents](README.md) · [← 1. Principles](01-principles.md) · [3. Target architecture →](03-target-architecture.md)

<a id="s2"></a>

# 2. System scope & portfolio model

<a id="s2-1"></a>

## 2.1. Systems in scope

| System | Description | Users |
|---|---|---|
| **Console** (User Portal) | Customers manage cloud resources, IAM, billing, support | Customers |
| **Admin Portal** | Internal operations: tenants, users, support, configuration | Internal staff |
| **Service Desk** | Tickets: customer side (in the Console) + agent workspace | Customers + support agents |
| **Content Studio** | CMS for website content, docs and announcements | Editors |
| **Public Website** | Landing, pricing, products, campaigns | General public |
| **Docs Portal** | Publishing channel for external-facing documentation: guides, API reference, SDKs, changelog. Content comes from multiple sources of truth; the Docs Portal does not own them ([§6.8](06-product-patterns.md#s6-8), [§7.3](07-factory-metadata-registry.md#s7-3)) | Developers, customers |
| **Chat AI** | Standalone chat application + assistant embedded in the Console | Customers, internal |
| **Document Management** | Business documents: storage, approval, signing (if any) | Customers, internal |
| **Design System** | Shared tokens, components, patterns, icons | All frontend teams |

<a id="s2-2"></a>

## 2.2. Four-level model

```text
Portfolio
 └── Product            (Cloud Platform)
      └── Application   (console, admin, desk, studio, web, docs, chat, dms)
           └── Surface  (console/compute, console/iam, desk/queue, web/pricing, ...)
                └── Archetype + Capability Packs + Cross-cutting profiles
```

A **surface** is a group of routes/modules with a consistent archetype and criticality — the unit to which policies and checks are applied. An application has multiple surfaces; for example, a developer portal has `docs/guides` (docs), `docs/account` (operational) and `docs/api-explorer` (embedded).

<a id="s2-3"></a>

## 2.3. Archetypes (experience types)

| ID | Used for | Main risks |
|---|---|---|
| `operational` | Console, Admin, Service Desk, Document Management | Wrong permissions, tenant data leakage, wrong state, destructive operations |
| `content-studio` | Content Studio | Wrong publish, lost revisions, preview ≠ published |
| `public-web` | Public Website | SEO, performance, consent, conversion |
| `docs-portal` | Docs Portal | Docs drifting from the API, broken code samples, wrong version |
| `conversational-ai` | Chat AI (standalone app) | XSS from model output, streaming, conversation leakage, AI-initiated actions |
| `embedded-ui` | Embedded assistants, widgets, API explorer | CSS leak, lifecycle, postMessage |
| `design-system` | Design System | Widespread breaking changes, a11y regression |

<a id="s2-4"></a>

## 2.4. Capability Packs (shared functional blocks)

An archetype describes a type of experience; a **pack** describes a function that recurs across multiple products. Each pack consists of components/patterns in the DS, hooks/adapters, checks, risk signals and scenario templates for tests.

| ID | Function | Main risks |
|---|---|---|
| `cap:workflow-state` | State machine, role-based transitions | Unauthorized transitions, concurrent updates |
| `cap:threaded-conversation` | Threads, mentions, **public reply vs internal note** | Internal notes exposed to customers |
| `cap:attachments` | Upload/download/preview | Malware, ACL, executable SVG/HTML |
| `cap:sla-timers` | SLA, business hours, time zones | Incorrect SLA display |
| `cap:assignment-queue` | Queues, claim, collisions | Two people handling the same item |
| `cap:notifications` | In-app, email/push, preferences | Content leaked in previews |
| `cap:search-filter` | Search, facets, saved views, URL state | Cross-tenant results |
| `cap:bulk-actions` | Multi-select, dry-run, partial failure | Wrong operation scope |
| `cap:audit-trail` | Activity log: who, when, acting as whom | Lost trace of impersonation |
| `cap:rich-text` | Editor, paste, markdown | XSS via paste |
| `cap:realtime-presence` | Live updates, presence | Event ordering/duplication |
| `cap:import-export` | CSV/Excel | **CSV/formula injection**, export beyond permissions |
| `cap:dashboard-analytics` | Charts, KPIs | Stale data without notice, inaccessible charts |
| `cap:forms-wizard` | Multi-step forms, drafts | Double submit, lost drafts |
| `cap:billing-payments` | Pricing, invoices, payments | Card data touching our DOM (PCI), double charging |
| `cap:streaming-response` | SSE/WebSocket/fetch stream | Mid-stream interruption, cannot be stopped |
| `cap:records-retention` | Versions, restore, retention, legal hold, policy-based deletion | Wrong deletion or failure to delete, lost versions, retention violations |

<a id="s2-5"></a>

## 2.5. Cross-cutting profiles

| ID | Attached when |
|---|---|
| `x:a11y` | Every surface with a UI (WCAG 2.2 AA) |
| `x:security-sensitive` | Auth, IAM, billing, secrets, CSP/headers |
| `x:privileged-admin` | Internal surface with permissions crossing tenant boundaries |
| `x:ai-surface` | Displays LLM output or lets AI perform actions |
| `x:i18n` | Multiple locales |
| `x:perf-public` | Public surface (Core Web Vitals) |
| `x:perf-app` | App surface with critical routes |
| `x:theming` | Multiple themes/modes |

<a id="s2-6"></a>

## 2.6. Composition recipes

| System | Archetype | Packs | Cross-cutting |
|---|---|---|---|
| Console | `operational` | workflow-state, search-filter, bulk-actions, forms-wizard, notifications, realtime-presence, dashboard-analytics, import-export; billing-payments for `console/billing` | a11y, perf-app, i18n, theming; security-sensitive for iam/billing |
| Admin | `operational` | search-filter, bulk-actions, audit-trail, workflow-state, import-export, dashboard-analytics | **privileged-admin**, security-sensitive, a11y |
| Desk — customer side (in the Console) | `operational` | forms-wizard, threaded-conversation (public), attachments, notifications, sla-timers, workflow-state | a11y, i18n |
| Desk — agent workspace | `operational` | assignment-queue, threaded-conversation (public + internal), sla-timers, workflow-state, attachments, rich-text, search-filter, bulk-actions, realtime-presence, audit-trail | privileged-admin, security-sensitive, a11y |
| Content Studio | `content-studio` | workflow-state, rich-text, attachments, audit-trail, realtime-presence | a11y (+ATAG), i18n |
| Public Website | `public-web` | forms-wizard | perf-public, i18n, a11y |
| Docs Portal | `docs-portal` (+ `embedded-ui` for the API explorer) | search-filter | perf-public, i18n, a11y |
| Chat AI | `conversational-ai` | streaming-response, attachments, search-filter, rich-text | ai-surface, security-sensitive, a11y, i18n |
| Embedded assistant | `embedded-ui` | streaming-response | ai-surface, a11y |
| Document Management | `operational` | attachments, workflow-state, audit-trail, threaded-conversation, records-retention | security-sensitive, a11y |

<a id="s2-7"></a>

## 2.7. Coverage and extension

"Fully covered" is a **machine-checkable** condition:
1. Every surface has ≥ 1 archetype.
2. Every critical journey has ≥ 1 check beyond the baseline.
3. `factory doctor --coverage` detects missing packs from signals in the code (upload → `attachments`; `EventSource`/WebSocket → `streaming-response`; editor → `rich-text`; CSV generation → `import-export`; payment SDK → `billing-payments`; deletion/retention API → `records-retention`; admin route → `x:privileged-admin`).
4. Coverage reliability is measured with seeded defects ([§12](12-evaluation.md#s12)).

A new product type: **compose** from what already exists → if something is missing, **add a pack** → add an **archetype** only when there are ≥ 2 risk groups that no pack covers, together with an owner and eval cases.

---

[Contents](README.md) · [← 1. Principles](01-principles.md) · [3. Target architecture →](03-target-architecture.md)
