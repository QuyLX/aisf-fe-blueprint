# AI Software Factory
## Standard — Engineering operating system, core requirements and profile framework

**Version:** 4.0.0  
**Date:** 2026-11-23  
**Scope:** Software engineering from product intent and requirements, through implementation, verification, delivery and production operations, to continuous improvement.  
**Canonical vocabulary:** [`aisf-vocabulary.yaml`](aisf-vocabulary.yaml) (mandatory; presented in [Appendix J](appendices/J-vocabulary.md)).  
**First profile and showcase:** AISF Frontend Profile 1.1, Part A of the *Frontend Platform & AI Software Factory Blueprint v4.1*. Domains without a profile use the prebuilt Core Profile (`aisf-core`, §20.7) at L1–L2.

> **Core thesis:** Build an engineering environment in which AI agents can understand the system, make bounded changes following a specified workflow, verify real behavior, provide evidence, keep the system's knowledge valid, and improve the environment when failures recur. Humans direct intent, consequential decisions and accountability; agents execute within explicit constraints.

> **Starting a pilot?** Read the [Pilot checklist](pilot-checklist.md) first: a single page listing exactly what to do in the first 6–8 weeks. Pilot mode is defined in §20.8.

---

## Contents

**Part I — Overview**

| § | Chapter | Content |
|---|---|---|
| 1 | [Executive summary](01-overview.md) | Definition, vision, target outcomes, out of scope |
| 2 | [Context, problem and design principles](02-context-and-principles.md) | Problem, principles P01–P28 (interpretation rule for every requirement) |
| 3 | [Target architecture](03-target-architecture.md) | Three planes, Guide/Sensor/Human gate, three loops |

**Part II — Core design** (each section: numbered requirements, followed by *Rationale* and *Example*)

| § | Chapter | Content |
|---|---|---|
| 4 | [Engineering Metadata & Architecture Intelligence](04-engineering-metadata.md) | Entities, relationships, metadata sources, authority, conflict handling |
| 5 | [Engineering knowledge and documentation governance](05-knowledge-and-documentation.md) | `AGENTS.md`, `REVIEW.md`, knowledge artifact lifecycle, dependency index, reconciliation |
| 6 | [Agent harness and operating model](06-agent-harness-and-workflow.md) | Task contract, task lifecycle, Task Workflow Specification, checkpoint, handoff, recovery, agents on developer machines |
| 7 | [Verification architecture & evidence](07-verification-and-evidence.md) | Verification profile, evidence contract, AI review, integrity, verifier separated from the change, adapter |
| 8 | [Engineering rules & codebase gardening](08-rules-and-codebase-gardening.md) | Rule, detector, gardening, Check Registry |
| 9 | [Agent evaluation, trust and governance](09-evaluation-trust-governance.md) | Eval, trust, autonomy level, floor tier, circuit breaker, separation of duties |
| 10 | [Agent security & data governance](10-security-and-data.md) | Security baseline, data classification and retention, threat model, prompt injection, security review |
| 11 | [Dependency engineering & technology governance](11-dependencies-and-technology.md) | Lifecycle, decision record, vulnerability SLA, license, model deprecation |
| 12 | [Delivery, release and production operations](12-delivery-and-operations.md) | Operational readiness, Change Shepherd, production signals, change channel, degraded mode |
| 13 | [Factory economics and metrics](13-economics-and-metrics.md) | Metric catalog, North Star, attention budget, workflow metrics |

**Part III — Implementation**

| § | Chapter | Content |
|---|---|---|
| 14 | [Technology architecture: build vs reuse](14-technology-architecture.md) | Build or reuse, platform mapping |
| 15 | [Implementation roadmap](15-implementation-roadmap.md) | Minimum Viable Harness, pilot claim, Phase 0–6 |
| 16 | [Initial pilot backlog](16-pilot-backlog.md) | Priorities P0–P3 |
| 18 | [Open architecture decisions](18-open-decisions.md) | Decisions settled through the pilot, intentionally deferred items |
| 19 | [Design stance](19-design-stance.md) | Durable assets and implementation rules |

**Part IV — Governance and conformance**

| § | Chapter | Content |
|---|---|---|
| 17 | [Standard ownership and governance](17-standard-governance.md) | Ownership model, operating cadence, versioning, change process, effective dates, registry |
| 20 | [Profiles, extension points and conformance](20-profiles-and-conformance.md) | Profile Contract, extension, conformance level, **Core Requirements Index**, Core Profile, pilot mode |

**References**

| | Document | Content |
|---|---|---|
| 21 | [References](21-references.md) | R01–R39 |
| A | [Suggested Factory CLI](appendices/A-factory-cli.md) | Command-line golden path |
| B | [Minimal `AGENTS.md` example](appendices/B-agents-md.md) | Minimal entry point file |
| C | [Task Workflow Specification](appendices/C-task-workflow-spec.md) | Format, reference and pilot Bug Fix, `generic` workflow |
| D | [Workflow catalog by task type](appendices/D-catalog-workflow.md) | Outlines of workflows by task type |
| E | [Knowledge artifact metadata](appendices/E-knowledge-artifact.md) | Required fields, frontmatter, index, reconciliation |
| F | [`REVIEW.md` example](appendices/F-review-md.md) | Owner-written review guidance |
| G | [PR template](appendices/G-pr-template.md) | Intent, Risk, Testing, Revert, Checkpoint, Knowledge impact |
| H | [Pilot Charter template](appendices/H-pilot-charter.md) | Pilot scope and criteria |
| I | [End-to-end walkthrough: bug fix](appendices/I-walkthrough-bug-fix.md) | Full flow from issue to reconciliation |
| J | [Canonical vocabulary](appendices/J-vocabulary.md) | Presentation of `aisf-vocabulary.yaml` |
| K | [Canonical schemas](appendices/K-schema.md) | TypeScript schemas |

---

## Reading paths

1. **Idea:** §1 → §2 → §3 → §19.
2. **Getting started:** [Pilot checklist](pilot-checklist.md) → §15 → Appendix I → Appendix C.
3. **Conformance:** §20 (level, Index, Core Profile, pilot mode).
4. **Detail by topic:** §6 → §7 → §9 are the core; then §4, §5, §8, §10–§13, §17.

## Reading by role

| You are | Read first | Then |
|---|---|---|
| Pilot team | [Pilot checklist](pilot-checklist.md) | §20.8, §15, Appendix C, I |
| Non-frontend team without a domain profile | §20.7 (Core Profile), [Pilot checklist](pilot-checklist.md) | §20.6, §20.8, Appendix C.5 |
| Engineering leadership | §1, §2.3, §3.1, §15.0 | §13, §19 |
| Factory / platform builder | §6, §7, §20, Appendix K | §3, Appendix C, D, E |
| Repository owner looking to onboard | §5.2, §5.8, §6.7 | Appendix B, F, G, H |
| Reviewer / approver | §6.12, §9.5, §9.10 | §7.6, Appendix G, I |
| Conformance assessor | §20, then the sections referenced by the Core Requirements Index | — |
| Profile owner | §20, §7.3, §17 | Appendix J, K |

---

## Conventions

- **Requirements** are numbered items containing the uppercase keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT** or **MAY**, interpreted as in RFC 2119 and RFC 8174. Definitions and canonical tables brought in by a requirement are also normative. **Rationale** and **Example** paragraphs, together with §1–3, §14–16, §17.1–17.3, §18, §19, §21 and Appendices A, B, D, F–I, do not set requirements.
- **References** take the form "§7.11 item 2". The Core Requirements Index (§20.6) groups requirements into `CORE-*` codes and is the sole conformance surface: a conformance claim is assessed only against CORE-IDs. A MUST that is not in the Index is a design requirement for harness and profile builders. The cited section determines the meaning of the Index entry, and the Index entry must not be stricter than that section.
- A requirement marked *from <date>* in the Level column of §20.6 is assessed as SHOULD before that date and as MUST from that date onward. A claim is assessed against the Standard version it pins (§17.4 item 9).
- Numbers marked *suggested* or *suggested default* MAY be changed by a profile or pilot within the stated limits. Most default thresholds come from the published practice of a small number of vendors (mainly R05, R28, R29); they are starting points for the pilot to recalibrate.
- Enumerated terms (tier, level, state, disposition, sub-state, channel, checkpoint) are defined only in `aisf-vocabulary.yaml`. New terms are registered as a profile extension (§20.3).
- The principles in §2.3 are interpretation rules: when a requirement is ambiguous, it is interpreted in the way that best serves those principles.
- Content marked as a **pilot decision** must be confirmed on the selected repository. The loosenings granted to pilots are only those listed in §20.8.
