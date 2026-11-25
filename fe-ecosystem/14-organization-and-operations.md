[Contents](README.md) · [← 13. Metrics](13-metrics.md) · [15. Roadmap →](15-roadmap.md)

<a id="s14"></a>

# 14. Organization & operations

<a id="s14-1"></a>

## 14.1. Ownership model

| | **Vertical owner** (each product/service) | **Frontend Platform team** |
|---|---|---|
| Owns | Product repo; modules/domains; features; journeys; product profile (may only add/tighten); app runtime, SLO, on-call; product incidents | Standards; quality gates; Factory (policy repo, registry, checks, classifiers, trust, eval); Design System; app shell; `@platform/*`; presets; generators; foundation upgrade schedule |
| Decides | Scope, priorities, UX within the DS frame, architecture within a module | Stack, repo standard, baseline, gate thresholds, autonomy policy, public API of the DS and `@platform/*` |
| Commitment to the other side | Pass the quality gates; adopt versions within the window; declare descriptors/journeys accurately; report FPs through the standard channel | Gates with mutation proof and FP rate within `fpSlo`; documentation + codemod for every breaking change; support SLA; never build features on behalf of verticals |

**The platform is an internal product** whose customers are the verticals: public roadmap, release notes, migration guides, support SLA, satisfaction measurement.

**Quality gates are guardrails, not an approval queue.** The platform team does not review individual vertical PRs; it ensures the automated gates are correct and sufficient. Platform people only participate in: policy loosening, baseline waivers, changes in the platform/policy repo, and security review when the profile requires it.

**Relationship with the Standard owner.** The Frontend Platform Owner owns this profile. Extensions are registered in the registry before use (PC-5); deviations from AISF Standard 4.0 are recorded as ADRs and require Standard owner approval (PC-4, Standard §20.2), and practices this profile proves out are proposed upstream (PC-7). Conformance claims are assessed by someone outside the team owning that scope.

**Backup owner.** Every surface, module and control file has at least two eligible code owners or one named backup ([§10.5](10-autonomy-policy.md#s10-5)). Having only one owner is a finding of `factory doctor --owners`.

<a id="s14-2"></a>

## 14.2. Roles in the Frontend Platform team (may be combined)

| Role | Responsibilities |
|---|---|
| Platform engineer | Platform repo, generators, app shell, `@platform/server`/`auth` (with SLO template and runbook), presets, Next.js upgrade schedule, Factory CLI |
| Quality & policy engineer | Policy repo, mandatory pipeline, attestation, check registry, check budget, FP triage, portfolio index |
| Design System maintainer | Tokens, components, pattern contracts, visual baseline, Token Usage Index |
| **Quality Engineering lead** (owner of Testing & Quality Engineering) | Testing Platform: presets and `@platform/testing`, Playwright harness, mocks/scenarios, contract tests, Test Planner, Test Asset Catalog, flaky registry, test data tooling, preview environment templates; Testing Platform metrics; seeded defect dev set |
| **Documentation & Knowledge lead** (owner of Documentation & Knowledge) | Document classification and source-of-truth rules ([§7.3](07-factory-metadata-registry.md#s7-3)), Knowledge Record conventions, AGENTS.md template, Knowledge Index, `docs.*` and `knowledge.*` checks, the Docs Portal platform (templates, build, search); freshness and retrieval metrics. Works with technical writers |
| Security liaison | Security baseline, server layer/session, agent sandbox, `security-sensitive` review |
| Agent/Eval engineer | Playbooks, skills, eval, trust registry, classifiers; seeded defect hold-out set (separated from check authors); retrieval eval |

<a id="s14-3"></a>

## 14.3. Responsibility matrix

| Activity | Vertical owner | Platform team |
|---|---|---|
| Features, journeys, product profile | **A/R** | C |
| App runtime, on-call, SLO | **A/R** | C (templates, tooling) |
| Standards, baseline, gate thresholds | C (via champion) | **A/R** |
| Add a check / move a check to blocking | C | **A/R** |
| Waiver for a product check | **A** (tech lead) | I |
| Baseline waiver, waiver > 90 days | C | **A** |
| DS, app shell, `@platform/*` | C | **A/R** |
| New version adoption | **A/R** (within the window) | R (bot, codemod) |
| Incident caused by a platform defect | C | **A/R** |
| Incident caused by a product defect | **A/R** | C |
| Profile change (extension, tightening) | C | **A/R** (register extension in the registry; Standard owner approves deviations) |
| Conformance claim for a repo | **A/R** (provides evidence) | C (arranges an independent assessor) |
| Product test assets (tests, journeys, scenarios, baselines) | **A/R** | C |
| Testing Platform and shared test assets | C | **A/R** |
| Product documentation and its freshness | **A/R** | C |
| Documentation conventions, Knowledge Index, Docs Portal platform | C | **A/R** |

*A = accountable, R = responsible, C = consulted, I = informed.*

<a id="s14-4"></a>

## 14.4. Decision rights

| Decision | Decider |
|---|---|
| Intent, scope, business acceptance | Vertical product owner |
| Architecture within a module, boundary exceptions within a repo | Vertical tech lead |
| Stack, repo standard, baseline, quality gates, autonomy policy, registries | Frontend Platform Owner |
| Public API of the DS and `@platform/*` | DS owner / Platform owner |
| Security/privacy exceptions | Security/Privacy Owner |
| Waivers | Approver by scope (vertical tech lead for product checks; DS owner; security owner); the Frontend Platform Owner only approves baseline waivers and waivers > 90 days |
| App production rollout | Vertical release owner |
| Disputes between a vertical and the platform | Recorded as an ADR; the Frontend Platform Owner decides on standards, the product owner decides on the product |

<a id="s14-5"></a>

## 14.5. Maintenance capacity

Every kit, profile, check and playbook has a named owner and a review cadence. A check without an owner → demoted to advisory or deleted. Registry CI rejects exceeding the wave's check budget. No new wave is opened while the current wave has not met its exit gate.

<a id="s14-6"></a>

## 14.6. Change management

| Activity | Content |
|---|---|
| Blocking schedule | Notice ≥ 2 sprints ahead when a group of checks moves to blocking, with violation counts per repo and the autofix command |
| Shadow first | Check results shown in the PR (non-blocking) throughout the shadow stage |
| Feedback channel | Chat channel + issue label for FPs/waivers; the check owner triages within SLA (suggested 1 business day); overdue → temporary 7-day waiver by scope |
| Training | Golden path guides, office hours, playbook demos |
| Champion | One champion per vertical, participating in check catalog review, the blocking schedule and the platform roadmap |

<a id="s14-7"></a>

## 14.7. Ownership of Testing & Quality Engineering and Documentation & Knowledge

Both capabilities are used across all verticals, so ownership is split by **layer**: the platform owns the tooling and conventions, verticals own their own content, and backend/infra teams own the environments and data they operate.

| Question | Testing & Quality Engineering | Documentation & Knowledge |
|---|---|---|
| Who operates the tooling? | Quality Engineering lead: harness, Test Planner, runners, flaky registry, Test Asset Catalog. Infrastructure: runners and preview | Documentation & Knowledge lead: Knowledge Index, Docs Portal build and search, AGENTS.md generator |
| Who manages **shared** assets or content? | Quality Engineering lead: `@platform/testing`, scenario templates per pack, persona generator | Documentation & Knowledge lead: taxonomy, templates, record conventions; DS maintainer: component documentation |
| Who manages **product** assets or content? | Vertical owner: tests, journeys, scenarios, visual baselines | Vertical owner and service teams: product guides, runbooks, ADRs; backend teams: contracts and API reference |
| Who is responsible for **freshness**? | Vertical owner: orphan tests, overdue quarantines, ACs without tests | The owner recorded in the document or record; gardening reports, the owner fixes at the source |
| Who provides environments and data? | Backend/infra environment owners: staging, test-data API, tenant pools; Quality Engineering: frontend-side tooling | Source systems (Git, contract registry, CMS, tracker) hold access rights; the index inherits those rights |
| When the **test environment** is unreliable? | The run returns `ERROR` ([§8.8](08-verification.md#s8-8)), the PR is not penalized. The environment owner receives the incident and has an SLA; the Quality Engineering lead tracks the failure attribution rate and escalates to the Frontend Platform Owner when the threshold is exceeded | — |
| When a **knowledge source** is unreliable (incorrect, stale, conflicting)? | — | The agent escalates instead of choosing on its own ([§9.4](09-agent-operating-model.md#s9-4)); the source owner fixes at the source; the Documentation & Knowledge lead, gardening or an agent MAY move the record to `NEEDS_REVALIDATION` (the only transition non-authority people and automation may make, Standard §5.8) until the owner reconciles ([§7.3](07-factory-metadata-registry.md#s7-3)); incorrect documentation leading to an incident is classified as "incorrect or stale documentation" ([§11.5](11-delivery-change-management.md#s11-5)) |
| Who decides policy changes? | Frontend Platform Owner (flaky thresholds, planner rules, gates) | Frontend Platform Owner (retrieval rules, visibility); Security/Privacy Owner (`restricted` content, data policy for LLMs) |

Whether Documentation & Knowledge belongs to the Frontend Platform team or to an organization-level Developer Experience/technical writing team is a Phase 0 decision ([§17](17-decisions-to-make.md#s17)). This design does not depend on that choice, provided the roles in [§14.2](#s14-2) have a named person.

---

[Contents](README.md) · [← 13. Metrics](13-metrics.md) · [15. Roadmap →](15-roadmap.md)
