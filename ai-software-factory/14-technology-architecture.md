# 14. Technology architecture: build vs reuse

## 14.1. Build the intelligence; reuse the infrastructure

| Capability | Recommendation | Rationale |
|---|---|---|
| Canonical metadata model | Build | Relationships and source-of-truth rules are organization-specific |
| Repository manifest/schema | Build | Small, portable, versioned contract |
| Central catalog | Reuse or build thin | Backstage or a lightweight index; decide after the pilot |
| Context builder | Build the orchestration around repository search/LSP/tools (e.g., SCIP-based code navigation [R31]) | Context selection is domain-specific |
| Architecture diagrams | Reuse C4/Structurizr or existing tooling | Avoid building a custom diagramming engine |
| Dependency graph | Reuse scanners/parsers | Avoid maintaining parsers for multiple languages |
| Rule engine | Reuse linters/policy engines; build the rules | Rules are organization-specific; the engine is a commodity |
| Agent runtime | Reuse an approved coding-agent harness | Avoid owning model execution unless required |
| Agent workflows/playbooks | Build as a schema-backed Task Workflow Specification | Specific to engineering tasks and risk policy |
| Workflow engine | Start with a task runner on issue/PR labels and CI; introduce an orchestration engine only when the pilot measures a need | Avoid building an engine before the workflow is proven |
| Knowledge dependency index | Build thin on metadata in Git; a graph service only when the thresholds in §5.9 are reached | Avoid graph infrastructure before scale demands it |
| Verification adapter | Build a thin adapter layer around existing tools per §7.16 | One common contract for many toolchains |
| Tool adapter | Build only when contract/security/normalization requires it | Avoid unnecessary abstraction |
| Verification runner | Reuse existing test engines | Keep developers' and CI's current tools |
| Evidence aggregator | Build a thin integration layer | Normalize output across tools and product profiles |
| Eval | Reuse platform features where appropriate; keep datasets portable | Avoid provider lock-in and keep the regression corpus portable |
| Observability | Reuse the existing backend; use OpenTelemetry instrumentation where appropriate | Avoid a second telemetry platform |
| Developer interface | Start with a CLI and in-repository commands | Fast path to adoption; a portal later if it proves useful |

## 14.2. Reference stack by domain

Reference technology stacks belong to domain profiles, not to the Standard. The frontend stack (TypeScript, React, Next.js, Vitest, Playwright, Storybook, MSW, Pact, OpenFeature, OpenTelemetry and other components) is defined in the AISF Frontend Profile (*Frontend Platform & AI Software Factory Blueprint v4.1*, [§4.1](../fe-ecosystem/04-technical-foundation.md#s4-1)). Every profile's stack should:

- Reuse the organization's existing toolchain unless a measured problem justifies a change.
- Avoid introducing duplicate test, telemetry or catalog systems.
- Record every default as an ADR, validated by a spike before rollout.

## 14.3. Portability requirements

- Task and evidence schemas must be provider-neutral.
- Agent prompts/configuration may be provider-specific adapters, but their versions must be recorded.
- Skills/playbooks should avoid unnecessary dependence on a single IDE where feasible.
- Tool integrations must have explicit contracts and permissions.
- Eval datasets should be portable and versioned independently of provider dashboards.

## 14.4. Platform mapping

The Standard is platform-neutral (§18 item 1), but some controls require specific Git/CI platform features. The table below is a starting point for the Pilot Charter to settle the implementation; feature names and required service tiers change between versions, so they need to be verified on the actual platform in Phase 0.

| Control | GitHub | GitLab | Azure DevOps | Bitbucket |
|---|---|---|---|---|
| Required pipeline that a PR cannot modify (§7.15 item 1) | Ruleset requiring workflows at the organization level | Pipeline execution policy / compliance framework | Branch policy build validation using a protected template | Required merge check; the central pipeline needs careful verification |
| Path-based approval (§7.13, §9.10) | CODEOWNERS + "Require review from Code Owners" | CODEOWNERS + approval rule | Path-based required reviewers | Code owners / default reviewers |
| Excluding the requester, author, pusher (§9.10) | Custom required status check | Approval rule + custom check | Branch policy + custom check | Custom merge check |
| Merge queue (§6.9) | Merge queue | Merge train | Not built in; serialize via build validation | Not built in; requires an additional solution |
| OIDC workload identity (§7.15) | OIDC for Actions | CI ID token | Workload identity federation | OIDC for Pipelines |

When the platform does not provide a control, the Factory implements it through a separate required check, or records the gap explicitly in the conformance claim; the control must not be treated as satisfied.
