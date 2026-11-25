# 21. References

The references below are organized by the practice they underpin. Vendor case studies are examples, not universal standards; the reported results should not be treated as guaranteed outcomes.

Before external publication, every URL and summary must be re-verified; some sources are recent and may change.

**Verification status:** R02, R03, R04, R05, R06, R07, R28 and R29 had their title, date, author and summary content cross-checked against the source through an automated fetch-and-summarize tool; R02 does not display a publication date. A human needs to check the quoted figures against the originals, and verify the remaining references, before external publication.

## 21.1. Agent-first engineering and harness design

**[R01] OpenAI – Harness engineering: leveraging Codex in an agent-first world**  
2026-02-11. Repository knowledge as the system of record, a short `AGENTS.md` acting as a map, executable architectural constraints, progressive disclosure, agent legibility, incrementally increasing autonomy, and continuous codebase cleanup.  
https://openai.com/index/harness-engineering/

**[R02] OpenAI – Codex as a platform: build on the open agent harness**  
The page carries no date (it references a July 2026 changelog; date unconfirmed). Reusable agent loops, context, tools, progress, approvals, and integrating agents into existing products/workflows.  
https://developers.openai.com/blog/codex-as-a-platform

**[R03] Anthropic – How Claude Code works in large codebases: Best practices and where to start**  
2026-05-14. Repository setup, project context, hooks, skills, plugins, MCP, LSP, subagents, and practices for enterprise-scale codebases.  
https://claude.com/resources/articles/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start

**[R04] Anthropic (Michael Segner) – Steering Claude Code: when to use CLAUDE.md, skills, hooks, and subagents**  
2026-06-18. Choosing the right mechanism for durable instructions, on-demand expertise, deterministic hooks and delegation.  
https://claude.com/resources/articles/steering-claude-code-skills-hooks-rules-subagents-and-more

**[R05] Lauren Tan (Principal Engineer, SpaceXSI) – All Code Is Slop (Except Mine), AI Engineer CODE 2026**  
2026-11-11. An inverted trust pyramid (codebase → static analysis → verification → prose → human review); the codebase is the agent's largest input; constraints as the structure of a garden trellis; banning APIs that demand finesse to use; directory structure as context; comments as "permission slips" replaced by lint diagnostics; runtime verification with a feature map; TLA+, Lean and property-based testing for UI; evidence-centered review; fixing the inner loop before automating the outer loop. The results reported by the speaker (for example ~11,000 PRs per month for the team with ~0.5% reverted) are not a controlled benchmark.  
https://ai.engineer/code/2026/talks/lauren-tan  
Video: https://www.youtube.com/watch?v=ZZ19zhYnDKY&t=3662s

**[R06] Cursor Plugins – pstack by Lauren Tan (poteto)**  
A skill plugin for Cursor under the MIT license, also installable in Claude Code. 22 playbooks (investigation, bug fixing, performance hill-climbing, refactoring, shipping, PR watching, autonomous running, session resume/pause); verification skills (TDD, writing runtime-verification and formal-verification skills, statecharts); multi-model review that keeps only findings proven by execution; blast radius analysis; `/correct`, a command that encodes lessons at the highest structural level; principles such as prove-it-works, fix-root-causes, explain-the-number and encode-lessons-in-structure. Version pinning is needed before adoption.  
https://github.com/cursor/plugins/tree/main/pstack

**[R07] Thoughtworks (Jaya Simha Reddy Nandyala, Prabina Pani) – Engineering the harness: A practical pattern for reliable coding agents**  
2026-09-29. Guides, sensors, selective human gates, progressive disclosure, error feedback, cross-repository impact analysis, and coordinated delivery.  
https://www.thoughtworks.com/insights/blog/architecture/engineering-the-harness-a-practical-pattern-for-reliable-coding-agents

**[R28] The Pragmatic Engineer (Gergely Orosz) – OpenAI's agentic software factory**  
2026-09-15. A nine-stage pipeline from human-defined outcomes to agent-driven deployment; per-change rollout agents that choose their own success signals and build their own dashboards; performance issue triage and propose-only incident bots; agent review with risk classification and opt-in auto-approval; infrastructure pressure from increased throughput; adoption patterns. Part of it is behind a paywall — the Standard uses only the public parts. The figures are reported numbers, not a controlled benchmark.  
https://newsletter.pragmaticengineer.com/p/openai-software-factory

**[R29] Anthropic (Addy Osmani, Morgan Westlee Lunt, Qing Wang) – How the Claude Code team does AI-assisted code review**  
2026-10-15. A multi-layer review pipeline; bug hunters running in parallel with voting verifiers; Important / Nit / Pre-existing findings; blocking security review; per-change risk tiers with 0/1/2 human approvals; approval bound to the commit; review convergence controls; reviewer evals derived from incidents; `CLAUDE.md` and `REVIEW.md`; "you land it, you own it". Vendor-reported results.  
https://claude.dev/blog/solving-code-review-at-anthropic/

## 21.2. Organizational AI adoption and measurement

**[R08] DORA – State of AI-assisted Software Development 2025**  
A study of nearly 5,000 technology professionals; treats AI as an amplifier of an organization's existing strengths and weaknesses.  
https://dora.dev/research/2025/dora-report/

**[R09] DORA – AI Capabilities Model**  
An accompanying guide describing seven capabilities, adoption strategy, tactics and how to measure AI-assisted software development.  
https://dora.dev/ai/capabilities-model/report/

**[R10] DORA – Publications and ROI of AI-assisted Software Development**  
Used for organization-level measurement and the economics of adoption. Confirm the report version and methodology when using specific figures.  
https://dora.dev/research/publications/

## 21.3. Agent evaluation and observability

**[R11] OpenAI – Evaluate agent workflows**  
Traces, graders, datasets, eval runs and workflow-level regression analysis.  
https://developers.openai.com/api/docs/guides/agent-evals

**[R12] OpenTelemetry – GenAI semantic conventions**  
Semantic conventions for GenAI spans, tool execution, retrieval, agent invocations, events and metrics. The relevant GenAI conventions are currently marked as Development; confirm the current status and the privacy implications before standardizing on them.  
https://github.com/open-telemetry/semantic-conventions-genai

## 21.4. Catalog, metadata, architecture and contracts

**[R13] Backstage – Software Catalog**  
Catalog entities and ownership metadata stored alongside the code and collected for ecosystem discovery.  
https://backstage.io/docs/features/software-catalog/

**[R14] Backstage – Descriptor Format**  
Definitions of entities, owners, systems, domains and relationships. Note that Backstage's ownership metadata is for accountability in the catalog and should not be treated as a runtime authorization engine.  
https://backstage.io/docs/features/software-catalog/descriptor-format/

**[R15] Pact – Introduction and Consumer-Driven Contracts**  
Contract testing for interactions between consumers and providers.  
https://docs.pact.io/

**[R16] Structurizr / C4 Model**  
References for architecture as code and visual architecture models.  
https://docs.structurizr.com/  
https://c4model.com/

## 21.5. Verification, delivery and security tooling

**[R17] Playwright – Documentation**  
Browser automation, end-to-end testing and the trace viewer.  
https://playwright.dev/docs/intro

**[R18] dependency-cruiser**  
JavaScript/TypeScript dependency graphs and rule enforcement.  
https://github.com/sverweij/dependency-cruiser

**[R19] Renovate**  
Automated dependency update workflow.  
https://docs.renovatebot.com/

**[R20] OSV-Scanner**  
An open-source vulnerability scanning tool.  
https://google.github.io/osv-scanner/

**[R21] CycloneDX**  
Software Bill of Materials standards and the accompanying tool ecosystem.  
https://cyclonedx.org/

**[R22] OWASP GenAI Security Project – Top 10 for LLM Applications 2026**  
A threat taxonomy for applications that use LLMs.  
https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/

**[R23] OWASP GenAI Security Project – Top 10 for Agentic Applications**  
Risks and mitigations specific to autonomous/agentic applications.  
https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/

**[R24] NIST – Secure Software Development Framework (SSDF)**  
Secure software development practices and a process baseline.  
https://csrc.nist.gov/projects/ssdf

## 21.6. Technical documentation and architecture decision practices

**[R25] Diátaxis**  
A documentation system organized around tutorials, how-to guides, explanation and reference documentation.  
https://diataxis.fr/

**[R26] arc42**  
A practical template for software architecture documentation.  
https://arc42.org/

**[R27] Architecture Decision Records**  
ADR practices and templates.  
https://adr.github.io/

## 21.7. Technology foundation references

**[R30] Port – Developer Portal / Catalog documentation**  
An internal developer portal and alternative software catalog; compare with Backstage when choosing a central catalog.  
https://docs.port.io/

**[R31] Sourcegraph – Code Navigation / SCIP**  
Precise cross-repository code navigation and the SCIP index format; a candidate input for the context builder.  
https://sourcegraph.com/docs/code-navigation

**[R32] ESLint – Custom Rules**  
Writing AST-based lint rules with actionable messages and fixers.  
https://eslint.org/docs/latest/extend/custom-rules

**[R33] typescript-eslint – Custom Rules**  
Type-aware custom lint rules for TypeScript codebases.  
https://typescript-eslint.io/developers/custom-rules/

**[R34] Lean**  
A theorem prover and programming language for machine-checked proofs about algorithms and invariants.  
https://lean-lang.org/

## 21.8. Talks and case studies on the software factory

The talks below are the presenters' opinions and experience, not standards or benchmarks. The descriptions and figures have not been cross-checked against the original content; verify before quoting externally.

**[R35] Armin Ronacher (Earendil) – Build the Workshop Before the Software Factory, AI Engineer**  
Human attention remains the root limit as agent runtime grows; creating many new repositories does not make maintenance easier; build the "workshop" (open standards, durable workflows) before running the factory at scale.  
https://www.youtube.com/watch?v=JNexz3Tbiu8

**[R36] Zach Lloyd (Warp) – How to Build a Software Factory That Improves Itself, AI Engineer**  
The software factory as team infrastructure defined in code: sandboxes, evals on real data, and an improvement loop for handling recurring failures.  
https://www.youtube.com/watch?v=nKjPBAw_N_0

**[R37] ZazenCodes – Build Your Own AI Software Factory with Claude Code**  
A hands-on guide: role-based sub-agents, state tracking through Markdown files, parallel runs with git worktrees, and the cost of passing context between sub-agents. Its example uses an agent as an "approver"; this Standard does not allow that (P20).  
https://www.youtube.com/watch?v=ctoaIC4LHmI

**[R38] Google Cloud Tech – Build a "software factory" that doesn't produce AI Slop**  
Read-only sub-agents used to audit and clean up technical debt against platform standards, with a code freshness report.  
https://www.youtube.com/watch?v=EFejzwBLAVo

**[R39] Steve (Builder.io) – Build an agentic software factory: deep dive**  
A closed loop from production signals (error tracking, failed deploys, uptime, feedback) to reproduction, a fix in a worktree, a PR, PR watching, and periodic watchdog/lookback tasks to catch recurring defects. Its example uses an always-on machine holding a logged-in session; this Standard does not recommend that approach (§10.1).  
https://www.youtube.com/watch?v=pNmfMi-yjZk
