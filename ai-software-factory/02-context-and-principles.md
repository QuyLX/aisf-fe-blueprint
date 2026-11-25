# 2. Context, problem and design principles

The principles in §2.3 are binding as interpretation rules: when a requirement in the spec is ambiguous, it is interpreted in the way that best serves those principles.

## 2.1. Why a Factory is needed

AI reduces the cost of generating code, tests, documentation and change proposals. It does not automatically reduce the total cost of delivering reliable software. As code generation becomes faster, the bottleneck shifts to specification, context discovery, architecture, verification, review, integration, release and production operations.

The Factory must solve the following recurring problems:

1. Ambiguous requirements cause agents to invent their own assumptions.
2. Architectural knowledge is scattered across source code, documents, tickets, chat and individual people's heads.
3. Agents reproduce existing anti-patterns and spread them to new modules.
4. Implementation code and tests produced by the same agent can share the same misunderstanding.
5. Local tests pass while downstream consumers or other repositories break.
6. A successful CI run does not necessarily prove business behavior or user experience.
7. Multi-agent workflows can cause correlated errors, duplicate findings, stale context and coordination costs.
8. Tools, MCP servers, credentials and model providers introduce security and data governance risks.
9. Higher PR throughput can increase rework, review load, integration failures and operational risk.
10. Documentation, rules and eval suites become stale without owners and a maintenance process.
11. Agent throughput can grow faster than the scalability of version control, CI/CD, merge and release infrastructure [R28, R29].
12. Author-agent and reviewer-agent loops may fail to converge, consuming CI and review capacity without improving the change [R29].
13. An agent optimizing for green checks may weaken tests, rules or thresholds instead of fixing the code.
14. Every task type is forced into the same process, so type-specific steps (bug reproduction, migration design, consumer checks) are skipped or done by feel.
15. When the system changes, no one knows which specs, designs, ADRs, documents, tests, rules, skills and playbooks are still correct; agents rely on stale knowledge, or "fix" a spec on their own to match a bug in the code.

DORA research views AI as an amplifier of an organization's existing strengths and weaknesses. This Standard therefore treats technical foundations, data, platform quality, small batches and feedback loops as prerequisites for realizing the value of AI [R08, R09].

## 2.2. Three questions every workflow must answer

- **Can the agent do it?** Does it have the right context, tools, permissions, workspace and task definition?
- **Can we verify it?** Is there independent, appropriate evidence showing that the requirements and system behavior are correct?
- **Can we operate and recover it safely?** Is the change observable, owned, reversible where possible, and supported by a recovery path?

## 2.3. Design principles

| ID | Principle | Design consequence |
|---|---|---|
| P01 | Specification before implementation | Use a structured change intent for every task; use a fuller spec when complexity or risk demands it. |
| P02 | Harness matters more than model choice | Invest in context, tools, constraints, verification and feedback; keep models replaceable. |
| P03 | The repository is the primary knowledge surface | Keep authoritative product and engineering information versioned close to the code it describes. |
| P04 | Progressive disclosure | Start agents with a short map; load detailed context only when needed. |
| P05 | Constrain invariants, not every implementation detail | Enforce important boundaries, contracts, permissions and quality invariants; leave local design freedom within them. |
| P06 | The right verifier for the right problem | Use compiler/type checks for types, graph checks for boundaries, contracts for integration, runtime tests for behavior, and human judgment for product trade-offs. |
| P07 | Evidence over claims | Completion requires evidence tied to a specific revision. |
| P08 | Independent verification for important behavior | Avoid relying solely on tests or judgments produced by the implementing agent itself. |
| P09 | Small batches and fast recovery | Prefer small, isolated changes, short-lived branches, safe rollouts and tested recovery procedures. |
| P10 | Risk-based autonomy | Permissions and approvals depend on impact, reversibility, data sensitivity and policy. |
| P11 | Trust is tied to each task type | An agent trusted for one task type is not automatically trusted for another. |
| P12 | Least privilege and explicit grants | Enforce permissions outside the model; a prompt alone is not a security boundary. |
| P13 | Distributed knowledge, centralized governance | Keep source-of-truth knowledge local and distributed while centralizing schemas, policies, discoverability and quality standards. |
| P14 | Build intelligence, reuse infrastructure | Own the canonical model, policy logic, workflows and evidence contracts; reuse mature scanners, test engines, catalogs and runtimes. |
| P15 | Continuous gardening | Recurring findings should improve code, rules, tests, skills, documentation, evals or operational controls. |
| P16 | No implicit quality trade-offs | Blocking invariants cannot be offset by a weighted aggregate score. |
| P17 | Human accountability is always explicit | Humans own intent, architectural trade-offs, security decisions, exceptions and consequential approvals. |
| P18 | Measure accepted outcomes | Optimize for verified and accepted work items, escaped defects, recovery, cost and developer experience—not raw code volume. |
| P19 | The change determines the level of control, not the author | The risk tier comes from what changed (path, blast radius, semantics). Path eligibility (ownership trust plus execution trust, §9.5) can only select a lighter path within the same tier; it never bypasses mandatory gates. |
| P20 | Agent approval never fills a human's place | Automated review can pass a low-risk change where policy allows; it never replaces a mandatory human approval. |
| P21 | Bounded loops converge | Every agent–agent loop (repair, review, fix) has a round limit, reruns scoped to the diff, and escalates to humans. |
| P22 | Detect and recover fast, not just slow down | Invest in flags, staged releases, merge queues, one-step rollback and fast detection; add gates where blast radius justifies them. |
| P23 | Strongest layer first | Encode each lesson at the strongest layer that can hold it: codebase/API, then static analysis, then verification, then prose; human review is the last line, not the first (§3.5, §8.6). |
| P24 | Fix the inner loop before automating the outer loop | Automating a loop that produces poor code only produces more poor code; constraints and verification come before unsupervised automation. |
| P25 | One standard, many profiles | Domain profiles refine and tighten the core, never loosen it; conformance is declared with evidence, not by bare assertion (§20). |
| P26 | Degrade toward more human involvement, never toward fewer checks | When a Factory component is unavailable, the affected path requires more human involvement; deterministic checks are never silently skipped (§12.7). |
| P27 | Workflows are contracts | Each task type follows a specified workflow with sub-states, checkpoints and failure behavior; agents may add steps, never skip mandatory steps or approval gates (§6.7). |
| P28 | Knowledge has a lifecycle and authority | Each knowledge artifact has a validity state, an owner and dependencies. A discrepancy between code and intent is a conflict for the authority to decide, not a reason to edit the documentation on one's own (§4.7, §5.8–5.10). |
