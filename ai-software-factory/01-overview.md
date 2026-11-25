# 1. Executive summary

## 1.1. Definition

The **AI Software Factory** is a federated engineering operating system that coordinates people, AI agents, engineering knowledge, tools, enforceable constraints, verification systems and delivery processes to build and operate software safely and efficiently.

It is not a model, IDE, agent framework, prompt library or a set of independent bots. It is the combination of:

- **Engineering intelligence:** knowledge about products, domains, architecture, ownership, dependencies, contracts and policies.
- **Repository harness:** local guidance, tools, constraints, workflows, sandboxes and verification that make each repository understandable and operable for agents.
- **Agent execution:** roles, skills, playbooks, task-specific tools and bounded autonomy levels.
- **Verification and evidence:** deterministic checks, runtime behavior checks, evals and traceable evidence tied to the exact code revision.
- **Governance:** risk, trust, permissions, human approval, provenance and recovery.
- **Feedback and economics:** learning from failures and measuring verified outcomes, reliability, cost and capability.

## 1.2. Vision

Move software engineering from a code-writing process performed mainly by humans to an **AI-assisted, evidence-based, progressively autonomous engineering system**.

A developer should be able to state a desired outcome, for example:

> Fix the pipeline retry bug. Reproduce the bug first, implement the smallest safe fix, verify the behavior, and prepare a merge-ready pull request with evidence.

The Factory should then:

1. Classify the task and determine its risk.
2. Find the relevant products, domains, repositories, architecture, rules, contracts and dependencies.
3. Select the workflow (Task Workflow Specification) and verification profile appropriate for the task type.
4. Perform the work in an isolated environment with scoped permissions.
5. Verify behavior using mechanisms appropriate for the product type.
6. Produce a traceable evidence bundle and identify which specs, designs, documents and decisions are affected by the change.
7. Request human judgment only where policy or uncertainty requires it.
8. Hand off a pull request or release candidate.
9. Feed findings and outcomes back into rules, skills, tests, documentation and evals.

## 1.3. Target outcomes

| Dimension | Desired outcome |
|---|---|
| Requirement quality | Significant changes have explicit intent, scope, acceptance criteria and constraints. |
| Agent legibility | Agents can find the relevant architecture, conventions, contracts, ownership and tools without loading the entire knowledge base. |
| Code quality | Recurring engineering failures become architectural constraints or enforceable rules where possible. |
| Verification | Changes are supported by independent, risk-appropriate evidence, not by agent claims. |
| Developer experience | Common engineering tasks can be initiated through a small set of consistent workflows. |
| Autonomy | Low-risk, well-evaluated workflows can progress with less human supervision; high-impact decisions remain gated. |
| Multi-product scale | Shared standards and policies coexist with each product's own architecture and verification. |
| Operations | Releases are observable, recoverable and linked to the changes that caused them. |
| Improvement | Recurring findings and incidents improve the Factory instead of being fixed only once. |
| Knowledge validity | Specs, designs, ADRs, documents, skills and playbooks have an explicit validity state and are reconciled when the system changes. |
| Economics | The organization can measure the cost and quality of accepted and verified changes. |

## 1.4. Out of scope

The Factory does not aim to:

- Replace all developers or remove human accountability.
- Force every task through a multi-agent workflow.
- Make every repository follow the same internal architecture or test stack.
- Build a new package manager, compiler, test runner, code index or software catalog when mature tools exist.
- Centralize all source-of-truth documentation into a single database.
- Treat AI review as a substitute for deterministic verification or mandatory human approval.
- Automatically merge high-risk changes merely because an agent has a high historical success rate.
- Store model chain-of-thought. Store trace metadata, tool actions, decisions, outputs and the evidence needed for audit and debugging.
- Optimize PR count, lines of code generated, token volume or number of agents as the primary measure of productivity.

## 1.5. Strategic positioning

The Factory should be treated as an **internal developer product and an engineering capability**, not a single centralized coding bot.

Its architecture is federated:

- Centralize reusable policies, catalogs, registries, shared skills, playbooks, evaluation standards and cross-repository knowledge.
- Distribute execution, source-of-truth product knowledge, local constraints and product-specific verification into repositories and their CI environments.
- Connect them through canonical contracts for tasks, metadata, policy and evidence.
- Publish the shared part as a **standard with domain profiles** (§20). The first profile, which is also the showcase demonstrating the Standard, is the AISF Frontend Profile (*Frontend Platform & AI Software Factory Blueprint v4.1*, [`../fe-ecosystem/`](../fe-ecosystem/README.md)). Teams in domains without a profile (backend, data, infrastructure) start with the Standard's prebuilt Core Profile (§20.7) at L1–L2.
