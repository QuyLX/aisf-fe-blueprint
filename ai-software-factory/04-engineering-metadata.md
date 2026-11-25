# 4. Engineering Metadata & Architecture Intelligence

## 4.1. Purpose

**Engineering Metadata** is a structured map of products, software components, business domains, ownership, dependencies, contracts, verification and policies. It is a discovery and relationship layer, not a substitute for source code, runtime, API contracts or other designated sources of truth.

1. The Factory SHOULD NOT create entities for every function, hook, component or file.
2. The Factory SHOULD model at the level where ownership, interfaces, dependencies, risk or operational responsibility are meaningful.

**Rationale.** Engineering Metadata helps humans and agents quickly answer questions such as "who owns this", "what depends on it", "what does this change touch". Metadata is useful when it helps accomplish one or more of the following:

- Find ownership.
- Analyze dependencies and impact.
- Select context for agents.
- Coordinate cross-repository changes.
- Apply policies.
- Trace evidence.
- Govern the lifecycle of services and technologies.

Metadata at the level of functions, hooks, components or files quickly becomes stale, is costly to maintain and does not help answer the questions above.

## 4.2. Canonical entity taxonomy

1. Metadata MUST use the canonical entity taxonomy in the following table.
2. Every entity MUST have an `id` and an owner. The other attributes in the third column SHOULD be present when they apply.

| Entity | Description | Required or useful attributes |
|---|---|---|
| Organization / Team | Organizational unit and owner | id, name, contacts, lifecycle |
| Product / System | A coherent product or software system | id, owner, criticality, lifecycle |
| Application | Deployable application | repo, runtime, stack, environments |
| Repository | Source code and configuration boundary | URL, default branch, owners, classification |
| Domain | Business and vocabulary boundary | owner, glossary, capabilities |
| Capability | Business capability | domain, owner, lifecycle |
| Feature | User or system capability | domain, owner, requirements, verification profile |
| Service | Runtime service or platform component | API, owner, SLO, dependencies |
| API / Contract | External or internal interface | schema, version, compatibility policy, consumers |
| Event / Message | Asynchronous event contract | producer, consumers, schema, version |
| SDK / Library / Package | Reusable artifact for developers | package coordinates, languages, version policy, consumers |
| Module / Component | Logical code unit | path, public interface, layer, owner |
| Dependency | External or internal dependency | type, version, policy status, license, risk |
| Environment | Development, staging, production boundary | deployment target, data classification |
| Rule | Engineering constraint | owner, scope, detector, enforcement mode, lifecycle |
| Verification Profile | Mandatory checks and evidence for a product/task class | checks, thresholds, required artifacts |
| Agent Role | Responsibility and scope | permissions, allowed tools, evaluation profile |
| Skill | Reusable capability guidance | inputs, outputs, tools, evals, version |
| Playbook (Task Workflow Specification) | End-to-end workflow for a task type (§6.7) | trigger, inputs, steps, sub-states, checkpoints, handoffs, recovery, status |
| Knowledge Artifact | Spec, design, ADR, living document, runbook, agent instruction file, skill or playbook used as a basis for decisions (§5.8) | id, kind, status, owner, authority, describes, basedOn, validatedAgainst |
| Task / Execution | A specific requested change and its execution | goal, status, risk, trust, repo, commit |
| Evidence | Verification result or artifact | check id, status, commit, URI, timestamp, producer |
| Release / Deployment | A delivered artifact in an environment | artifact digest, version, change set, health |
| Incident / Finding | Defect, risk, review finding, operational issue | severity, root cause, recurrence, owner |

## 4.3. Relationship model

1. The metadata model MUST support at least the following relationships:
   - `ownedBy`: the entity is owned by a team/person.
   - `partOf` / `contains`: product/domain/application decomposition.
   - `implements`: a code component implements a feature or contract.
   - `dependsOn`: software or service dependency.
   - `providesApi` / `consumesApi`: API producer-consumer relationship.
   - `producesEvent` / `consumesEvent`: message relationship.
   - `usesPackage`: a repository/module uses a package.
   - `governedBy`: the entity is governed by a rule or policy.
   - `verifiedBy`: a requirement/feature is verified by a test or profile.
   - `deployedAs`: an application/service is deployed as an artifact.
   - `observedBy`: a runtime entity is observed by telemetry.
   - `supersedes`: an ADR, policy, schema, knowledge artifact or version replaces a previous one.
   - `describes`: a knowledge artifact describes a path, module, service, contract or entity (§5.9).
   - `basedOn`: the validity of a knowledge artifact depends on another artifact or contract version (§5.9).
   - `generatedFrom`: an artifact is mechanically generated from an approved source (for example, an API reference from OpenAPI).
   - `evaluatedBy`: an agent, skill or playbook is evaluated by an eval suite.
2. When the source of a `basedOn` relationship changes, the dependent artifact MUST be re-evaluated (§5.9).

## 4.4. Three metadata sources

Metadata has three sources:

- **Declared** metadata is written by maintainers and managed under version control.
- **Discovered** metadata is produced by static tools and repository analysis.
- **Observed** metadata comes from CI/CD, runtime, telemetry and incidents.

1. The registry SHOULD compare these three sources and report drift.
2. The registry MUST NOT silently overwrite declared intent with inferred data.

**Rationale.** Each source has its own blind spots, so the value lies in comparing them against each other: a difference between what is declared and what is discovered or observed is often an early sign of stale documentation or drifting architecture. Inferred data must not overwrite declared intent because discovered data can be wrong (tools do not fully support a language, incorrect inference), and because a difference may be a fault of the implementation rather than of the declaration. Reporting drift for the owner to decide preserves both possibilities.

**Example.** How each source is produced, examples and its main risk:

| Source | How it is produced | Example | Main risk |
|---|---|---|---|
| Declared | Written by maintainers, managed under version control | owner, domain, criticality, public API, intended dependencies | Missing or stale declarations |
| Discovered | Static tools and repository analysis | package manifest, lockfile, AST imports, OpenAPI, event schema | Incorrect inference, incomplete language/tool support |
| Observed | CI/CD, runtime, telemetry, incidents | deployed version, consumers, latency, errors, usage | Sampling, privacy and data retention |

## 4.5. Repository metadata example

```yaml
apiVersion: factory.example/v1
kind: Repository
metadata:
  id: cloud-console
  name: Cloud Console
  owner: cloud-console-team
  lifecycle: production
  criticality: high
  tags: [frontend, nextjs, cloud-platform]
spec:
  system: cloud-platform
  product: cloud-console
  source:
    provider: github
    url: https://github.example/org/cloud-console
    defaultBranch: main
  stack:
    language: typescript
    framework: nextjs
    packageManager: pnpm
  verificationProfile: web-application-v1
  knowledge:
    entrypoint: AGENTS.md
    architecture: docs/architecture/current-state.md
    adrIndex: docs/architecture/decisions/README.md
  dependencies:
    - pipeline-api
    - auth-service
    - cicd-typescript-sdk
```

## 4.6. Feature metadata example

```yaml
apiVersion: factory.example/v1
kind: Feature
metadata:
  id: pipeline-retry
  name: Pipeline Retry
  owner: cicd-team
  lifecycle: active
  criticality: high
spec:
  domain: cicd
  repositories:
    - cloud-console
    - pipeline-service
    - cicd-typescript-sdk
  requirements:
    - BR-PIPE-014
    - AC-PIPE-031
  contracts:
    - pipeline-api-v2
  dependencies:
    - retry-policy-service
  permissions:
    - pipeline.read
    - pipeline.retry
  verification:
    profiles:
      - web-application-v1
      - backend-service-v1
      - typescript-sdk-v1
```

## 4.7. Source-of-truth rules

1. The Factory MUST treat the sources in the following table as the source of truth for each type of information.

| Information | Source of truth |
|---|---|
| Product intent and acceptance | Approved specification / product decision |
| Current implementation | Source code, schema, configuration, runtime |
| Package resolution | Manifest and lockfile |
| API contract | Designated OpenAPI/AsyncAPI/schema source |
| Consumer expectations | Consumer-driven contract tests, where used |
| Architecture history | ADR |
| Current architecture | Living architecture documentation plus validated code structure |
| Mandatory engineering rules | Versioned, enforceable rules and policies |
| Verification results | CI/runtime/eval artifacts tied to a commit |
| Deployment state | Deployment platform and artifact registry |
| Runtime behavior | Observability platform and production systems |
| Declared ownership and system relationships | Versioned catalog metadata |

2. When sources conflict, the agent MUST state the conflict explicitly.
3. The agent MUST stop if the conflict affects correctness or risk.
4. The agent MUST NOT guess which source is authoritative.

**Authority by question type.**

5. The Factory MUST determine authority by question type according to the following table; each source is authoritative only for the corresponding question type, not for every question.

| Question | Authoritative source | Not authoritative to answer this question |
|---|---|---|
| What **must** the system do? (intent) | Specs and product decisions in state `ACTIVE`; approved API contracts | Code, existing tests, documents describing the current state |
| What **is** the system doing? (implementation) | Code, schema, configuration, runtime, telemetry | Spec, ADR |
| **Why** was the system designed this way? | ADRs in state `ACTIVE` | Code comments, descriptive documents |
| **Who** gets to decide? | Ownership metadata, §9.7, §9.10 | The content of the disputed artifact |

**Code is not presumed correct about product intent.** Code reflects the current implementation. A **divergence** is a case where an implementation differs from an `ACTIVE` spec; it has two equally weighted hypotheses: an implementation bug, or an unrecorded change of intent.

6. The agent MUST NOT modify a spec, acceptance criteria, test oracle or contract to match the code.
7. The agent MUST NOT modify code to match a spec that the current task does not cover.

**Conflict handling.**

8. The agent MUST record a **knowledge conflict** in the task record, including: the sources, the differing content, whether the conflict affects the correctness or risk of the task, and a recommendation (schema `KnowledgeConflict`, Appendix K).
9. If the conflict affects correctness or risk, the task MUST stop at `NEEDS_CLARIFICATION` (before `EXECUTING`) or `BLOCKED` (from `EXECUTING` onward).
10. A conflict under item 9 MUST be routed to the corresponding authority according to the following table.

| Conflict type | Deciding authority | Valid outcome |
|---|---|---|
| Code ≠ spec or acceptance criteria | Product/domain owner | Implementation bug → a task fixes the code. Intent has changed → the spec is modified through Specification Review (§6.12), followed by reconciliation (§5.10) |
| Code ≠ API contract | Contract owner, consulting affected consumers | Fix the provider, or open an API Contract Change workflow (Appendix D) |
| Code ≠ ADR or architecture documentation | Architect or code owner of the area | Fix the code; or a new ADR supersedes the old ADR; or fix the document if the document itself is wrong |
| Document describing the current state ≠ code | Document owner | Reconciliation: update the document after confirming the code is the desired behavior (§5.10) |
| Two `ACTIVE` artifacts conflict with each other | Owner of the artifact authoritative for that question type; if at the same level, the nearest common owner | One artifact is modified or superseded, with a record |
| Declared metadata ≠ discovered metadata | Metadata owner | Report drift, do not overwrite (§4.4) |

11. A conflict that does not affect correctness or risk MUST be recorded, MUST become a documentation-gardening finding, and MUST NOT block the task.
12. For a conflict under item 11, the task MAY continue.
13. Precedence between artifacts MUST come only from recorded `supersedes` relationships and lifecycle state (§5.8).
14. The Factory and agents MUST NOT use the most recent modification time ("newer") as a basis for authority.

**Rationale.** The main failure mode this section blocks is an agent choosing a source on its own when sources conflict, then acting as if that choice were self-evidently correct.

Code answers the question "what is the system doing" well, but cannot answer "what must the system do". If an agent modifies a spec, acceptance criteria, test oracle or contract to match the code, an implementation bug is legitimized as intent; if an agent modifies code to match a spec outside the task, it changes behavior that no one requested. The decision therefore belongs to the authority in the conflict handling table.

A more recently modified artifact is not thereby more authoritative. A minor edit (formatting, spelling) can make an old document look "new", while the actual decision was superseded long ago. Only recorded `supersedes` relationships and lifecycle state (§5.8) reflect the authority's intent.

## 4.8. Evolution of the Metadata Registry

1. The Metadata Registry SHOULD evolve through the following stages:
   - **Pilot:** local YAML/JSON metadata in the repository, validated by JSON Schema, together with a simple aggregator.
   - **Scale-out:** a central catalog/index collects repository metadata, adds discovered/observed relationships, and supports queries and impact analysis.
   - **Later:** a graph query service and dashboards, only when cross-repository queries, ownership workflows or impact analysis justify them.
2. The Factory SHOULD NOT start with a graph database merely because the target architecture contains relationships.
3. The initial source-of-truth model SHOULD remain portable and Git-friendly.
4. Dependencies between knowledge artifacts SHOULD follow the same roadmap: metadata in Git and a lightweight dependency index first, a graph service only when the conditions in §5.9 are met.

**Rationale.** The fact that the target architecture contains relationships does not mean graph infrastructure is needed from the start. Metadata in Git, validated by JSON Schema, is sufficient for the pilot, easy to review through pull requests and easy to migrate.

**Example.** At the scale-out stage, catalogs such as Backstage or Port are examples of a central catalog/index that collects repository metadata [R13, R14, R30].
