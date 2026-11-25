[Contents](README.md) · [← 0. Summary](00-summary.md) · [2. System scope & portfolio model →](02-portfolio-scope.md)

<a id="s1"></a>

# 1. Principles

1. **Uniformity first, local optimization later.** One language, one framework (Next.js), one repository standard, one DS. Exceptions MUST have an ADR and pay their own cost.
2. **Contract before code.** No contract, no integration.
3. **Deterministic first, probabilistic later.** Whatever a machine can check is not left to an LLM's judgment.
4. **Whoever creates or requests a change does not grade it and does not approve it alone.** This applies to agents, to CI jobs that run the PR's code ([§8.7](08-verification.md#s8-7)), and to the requester for every change above the Low tier ([§10.5](10-autonomy-policy.md#s10-5)).
5. **One source of truth per data type**; everything has an ID, a version and an owner.
6. **Verification by surface**, not by repo or app.
7. **Analyzable by construction:** write code so that impact analysis is computable (typed tokens, clients generated from contracts, tag-based boundaries).
8. **A check MUST be seen to fail** (mutation proof); its enforcement level comes from an **authoritative source**, not from how easy it is to write.
9. **Conventions are measured, not declared** (agent-legibility eval).
10. **Only `ACTIVE` records are authoritative**; approval is never inferred from merged code.
11. **One system for people and agents:** same CLI, same docs, same API.
12. **Verification before autonomy;** autonomy is per task class, versioned and revocable.
13. **Do not expand what cannot yet be maintained:** every check, kit and wave has a budget and an owner.
14. **Build intelligence, reuse infrastructure.**
15. **Small batches, measured per work item.**
16. **Verticals are autonomous within common guardrails:** the vertical owner decides the product; the platform decides standards and quality gates; neither side does the other's job.
17. **The Standard is the law; this profile tightens it.** Vocabulary, governance and evidence come from AISF Standard 4.0. Everything specific to frontend is added here as a registered extension or a stricter rule, never a looser rule. A deviation, if needed, is recorded as an ADR, approved by the Standard's owner (PC-4) and listed in the conformance declaration ([§18.1](18-aisf-conformance.md#s18-1)).

---

[Contents](README.md) · [← 0. Summary](00-summary.md) · [2. System scope & portfolio model →](02-portfolio-scope.md)
