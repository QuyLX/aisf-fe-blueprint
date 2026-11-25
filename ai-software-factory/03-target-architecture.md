# 3. Target architecture

## 3.1. Architecture overview

The architecture uses three interacting planes. The three planes are an implementation-oriented organization of the eight capability layers in §3.6.

```text
                         HUMAN INTENT & GOVERNANCE
                 Product Owner · Domain Owner · Architect
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    FACTORY CONTROL PLANE                             │
│                                                                      │
│  Engineering Catalog      Knowledge & Policy Registry                │
│  Agent / Skill Registry   Workflow (Playbook) Registry               │
│  Verification Profiles    Verification Adapters                      │
│  Evaluation Registry      Knowledge Dependency Index                 │
│  Risk & Trust Policy      Factory Analytics                          │
│                                                                      │
│  Cross-repository discovery · dependency graph · impact analysis     │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ contracts / policies / context
             ┌─────────────────┼──────────────────┐
             ▼                 ▼                  ▼
┌────────────────────┐ ┌──────────────────┐ ┌────────────────────┐
│ Repository Harness │ │ Repository       │ │ Repository Harness │
│ Product A          │ │ Harness Product B│ │ SDK / Service C    │
│                    │ │                  │ │                    │
│ Local knowledge    │ │ Local knowledge  │ │ Local knowledge    │
│ Local rules        │ │ Local rules      │ │ Local rules        │
│ Agent workflows    │ │ Agent workflows  │ │ Agent workflows    │
│ Verification       │ │ Verification     │ │ Verification       │
│ Sandbox / tools    │ │ Sandbox / tools  │ │ Sandbox / tools    │
└──────────┬─────────┘ └────────┬─────────┘ └──────────┬─────────┘
           └────────────────────┼──────────────────────┘
                                ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    FEEDBACK & EVIDENCE PLANE                         │
│                                                                      │
│ Task traces · Verification bundles · PR evidence · CI results        │
│ Release provenance · Production telemetry · Incidents                │
│ Rule/skill/eval updates · Knowledge reconciliation · Doc gardening   │
│ Workflow metrics · Factory quality, reliability, capacity, economics │
└──────────────────────────────────────────────────────────────────────┘
```

## 3.2. Plane A – Factory Control Plane

The Control Plane manages the capabilities that should be reused across many repositories and products.

| Component | Responsibility | Initial implementation |
|---|---|---|
| Engineering Catalog | Find products, repos, domains, services, SDKs, owners and their relationships | Versioned metadata files plus a lightweight index |
| Knowledge Registry | Index links to authoritative architecture, rules, specs and ADRs, along with lifecycle states and dependencies between knowledge artifacts (§5.8–5.9) | Frontmatter in the repository plus a CI-generated dependency index |
| Policy Registry | Publish shared policies on security, risk, trust and approval | Versioned YAML/JSON policies |
| Agent Registry | Define approved agent roles, model/provider constraints, tool scope and evaluation status | Configuration files, and a registry API only when needed |
| Skill Registry | Publish reusable skill packages and their versions | Git-based package repository or plugin-style distribution |
| Workflow (Playbook) Registry | Publish a Task Workflow Specification for each task type: trigger, input, step, sub-state, checkpoint, handoff, recovery (§6.7) | Versioned playbooks in Git, validated by schema (Appendix C) |
| Verification Profile Registry | Define the mandatory checks and evidence for each product type, and the profile composition layers (§7.3) | Versioned profile manifests |
| Verification Adapter Registry | Register adapters that execute checks for each toolchain under a common contract (§7.16) | Versioned adapters in the central pipeline (§7.15) |
| Evaluation Registry | Store eval datasets, graders, baselines and regression history | Datasets stored in Git plus an evaluation runner |
| Policy Decision Service | Evaluate task risk, agent trust, permission requests and approval requirements | Start as a library/CLI; become a service only when cross-system use demands it |
| Factory Analytics | Aggregate data on flow, quality, reliability, cost and capability | CI artifacts and structured events before any dashboard |

**Do not build every component as a separate service in the pilot.** The Control Plane is a logical boundary. The initial implementation can be a Git repository, a CLI, schemas and a small indexing process.

## 3.3. Plane B – Repository-level Engineering Harness

Each repository owns an execution environment that can understand and verify that repository.

| Harness component | Responsibility |
|---|---|
| Entry-point instructions | A short map of the repository and its operational constraints (`AGENTS.md` or the tool-specific equivalent). |
| Context resolver | Find the relevant specs, modules, ADRs, rules, contracts, ownership and dependencies; return only artifacts currently in effect, along with their states (§5.8). |
| Architecture model | Describe domains, package/module layers, allowed dependency directions and public interfaces. |
| Enforceable constraints | Lint rules, type checks, dependency graph rules, schema checks, security policies. |
| Task runner | Execute the steps of the Task Workflow Specification; record state, sub-state, checkpoint, handoff, basis and recovery events in the task record (§6.5, §6.7, §6.12–6.14). |
| Agent adapter | Integrate approved coding agents and their tool interfaces without hard-wiring the Factory to one model. |
| Workspace manager | Create an isolated branch/worktree/container/workspace for each task. |
| Verification runner | Invoke the verification adapters (§7.16) for the resolved check plan and collect artifacts. |
| Knowledge impact check | Identify knowledge artifacts affected by a change and require a reconciliation decision (§5.9–5.10). |
| Evidence collector | Normalize reports, screenshots, traces, test results and provenance into an evidence bundle. |
| Local policy enforcement | Apply the repository's own protected paths, commands, approvals and data rules. |

## 3.4. Plane C – Feedback & Evidence Plane

The Feedback Plane connects evidence from tasks, repositories, CI/CD and production.

It must support:

- Traceability from task to commit, PR, build, release and runtime outcome.
- Aggregation of verification results across repositories.
- Finding deduplication and root-cause classification.
- Generation of improvement proposals for rules, skills, specs, documentation, eval datasets and runbooks.
- Storage of reconciliation records and knowledge conflicts, to know which knowledge has been reconciled against which change (§5.10).
- Measurement of the quality of each workflow by version (§13.7).
- Measurement of whether an improvement reduces recurrence.
- Analysis of Factory economics and capability.

## 3.5. Harness execution model: Guide, Sensor and Human Gate

A reliable harness has three enforcement mechanisms [R07]:

| Mechanism | Timing | Example | Primary owner |
|---|---|---|---|
| Guide | Before action | Scoped guidance, architecture maps, policies, skills, playbooks, tool descriptions | Engineering owner / repository owner |
| Sensor | After action | Lint, type check, tests, architecture checks, runtime verification, security scans, evaluation graders | Harness owner / CI |
| Human gate | At consequential decisions | Public API changes, multi-repo scope, schema migrations, security-sensitive changes, production actions | Product/domain/code/security owner |

Design rule: do not require human approval for every trivial action. Reserve human gates for decisions with significant uncertainty, large blast radius, sensitive data or poor reversibility.

**Where to place constraints: the trust pyramid.** A prose guide is the weakest form of enforcement. Encode each lesson at the strongest layer that can hold it [R05, R06]:

| Order | Layer | Mechanism | Effect |
|---|---|---|---|
| 1 | Codebase | API design, architecture, directory structure, refactors that eliminate wrong choices | Deterministic; the error becomes impossible |
| 2 | Static analysis | Types, compiler, lint rules with actionable guidance messages, dependency-graph checks, tests | Deterministic; the error is caught |
| 3 | Verification | Runtime checks, browser/device drivers, property-based and formal verification | Confirms behavior and that the intent was correctly understood |
| 4 | Prose | `AGENTS.md`, `REVIEW.md`, rules, skills | For issues requiring judgment |
| 5 | Human review | Review of evidence and consequential decisions | The last line of defense, not the first |

When an agent repeats a mistake, the default response is not to add a line to `AGENTS.md`; it is a change at the highest effective layer (§8.6).

A useful sensor behavior is **silent success, actionable failure**:

- On success, return a brief status and a link to the evidence.
- On failure, return a structured diagnosis with file, location, check ID, severity and actionable remediation context.
- Never turn `NOT_ASSESSED` into `PASS`.

**Defense in depth.** No single layer is expected to catch everything. Pre-PR verification, CI, AI review, security review, risk-based approval, merge queue, staged release and rollback each catch some classes of problems and miss others. Very few defects get past all of them, and fast rollback handles the rest [R29]. When an incident occurs, ask *which layer should have caught it* and strengthen that layer instead of adding a generic checklist.

## 3.6. Eight capability layers

| Capability layer | Where it is implemented |
|---|---|
| 1. Product & Specification Intelligence | Control Plane standards + specifications in the repository + task intake |
| 2. Engineering Knowledge & Context | Catalog, knowledge registry, knowledge artifact lifecycle and dependency index, context resolver, reconciliation |
| 3. Agent Orchestration & Execution | Repository harness, task runner, Task Workflow Specification, agent adapter, workspace manager |
| 4. Verification Platform | Verification profiles, sensors, CI, runtime verification |
| 5. AI Evaluation & Review | Evaluation registry, reviewer workflows, trace grading |
| 6. Governance & Provenance | Policy registry, risk/trust decisions, approval, audit and provenance |
| 7. Delivery & Production Operations | PR workflow, merge controls, release pipeline, telemetry and recovery |
| 8. Continuous Improvement & Economics | Feedback plane, workflow gardening, metrics, capability and cost |

## 3.7. Three core loops

### Delivery loop

```text
Intent
 → Workflow selection (task class)
 → Change Brief / Specification  (Requirement / Specification Review)
 → Context Discovery
 → Impact & Risk Analysis
 → Plan / Design  (Design / Plan Review when required)
 → Implementation
 → Local Verification
 → Independent Verification
 → Review
 → Knowledge Impact & Reconciliation
 → Evidence Package
 → Human Approval (per risk tier, bound to commit)
 → Merge Queue (re-verify on latest main)
 → Release Readiness Review (when required)
 → Staged Release (internal use → early channel → general)
 → Monitoring
```

### Engineering improvement loop

```text
Finding / Repeated Pattern / Review Feedback
 → Root-cause classification
 → Select prevention mechanism
 → Code/API/Architecture change OR Rule/Test/Skill/Spec/ADR/Eval update
 → Test the improvement
 → Enforce or publish
 → Measure recurrence
```

### Production improvement loop

```text
Release
 → Observe technical and business signals
 → Detect regression
 → Alert / Pause / Rollback / Forward-fix
 → Incident analysis
 → Regression test / Runbook / Rule / Architecture improvement
 → Validate recovery mechanism
```
