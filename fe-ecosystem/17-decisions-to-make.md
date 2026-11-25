[Contents](README.md) · [← 16. Current state: problems and data, not constraints](16-current-state.md) · [18. AISF Standard 4.0 conformance (profile declaration) →](18-aisf-conformance.md)

<a id="s17"></a>

# 17. Decisions to make (Phase 0)

1. Confirm the default stack ([§4.1](04-technical-foundation.md#s4-1)) with Next.js as the only golden path ([§3.2](03-target-architecture.md#s3-2)); version skew window ([§4.8](04-technical-foundation.md#s4-8)).
2. Repo topology ([§3.3](03-target-architecture.md#s3-3)): polyrepo or product monorepo according to the four criteria in §3.3, with the results of the CI capability spike on a portfolio-sized repo; list of repos or projects and the vertical owner of each part; build tool (Nx/Turborepo).
3. **One** CI platform that meets the [§8.7](08-verification.md#s8-7) checklist and can apply the mandatory pipeline to every repo: OIDC, separate attest runner, signing method, license tier.
4. Headless CMS: which product; ticket backend: build or SaaS.
5. IdP, session model in the server layer (encrypted cookie or shared store), step-up auth.
6. Streaming protocol for Chat (SSE by default); who owns the AI backend and answer quality eval.
7. Backend contract-first commitment: OpenAPI publication schedule, Pact broker, staging tenant, mail sink; for services that have not published OpenAPI, use the interim `consumer-authored` path with backend owner review ([§4.2](04-technical-foundation.md#s4-2)).
8. Admin access network; which actions require four-eyes.
9. Accessibility conformance level according to the legal context.
10. Check budget per wave, trust/flaky/FP thresholds, **Frontend Platform team staffing per phase** ([§15.1](15-roadmap.md#s15-1)).
11. Retention of evidence, chat conversations and documents.
12. Platform ↔ vertical agreement: support SLA, adoption window, champions ([§14.1](14-organization-and-operations.md#s14-1)).
13. The Standard owner, and who may independently assess conformance claims (Standard §17.4, §20.5).
14. The AI reviewer product, and whether its model or vendor differs from the code-writing agent.
15. Which surfaces opt in to A0, and their owners.
16. RTO/RPO and SLO by criticality; error budget policy ([§19.3–19.4](19-nfr-enterprise.md#s19-3)).
17. Browser and device support matrix ([§19.5](19-nfr-enterprise.md#s19-5)); target SLSA level ([§19.2](19-nfr-enterprise.md#s19-2)).
18. Mapping of [§10.5](10-autonomy-policy.md#s10-5) and [§19.7](19-nfr-enterprise.md#s19-7) to compliance obligations (separation of duties (SoD), personal data law), agreed with the compliance owner; list of personal data legal instruments currently in effect, confirmed by legal.
19. Test-data API and test tenant pool on staging: which backend team owns them, staging environment SLA, seeding and cleanup mechanism ([§8.8](08-verification.md#s8-8)).
20. Per-PR preview environments: which infrastructure runs them, when to use mocks and when to use staging tenants, TTL ([§8.8](08-verification.md#s8-8)).
21. Requirement ID format and issue tracker integration for traceability ([§7.7](07-factory-metadata-registry.md#s7-7)); where visual baselines are stored ([§8.11](08-verification.md#s8-11)).
22. Owner of the Documentation & Knowledge capability: the Frontend Platform team or an organization-level Developer Experience/technical writing team ([§14.7](14-organization-and-operations.md#s14-7)); whether internal documentation uses an authenticated surface in the Docs app ([§6.8](06-product-patterns.md#s6-8)).
23. Search engine for the Docs Portal and Knowledge Index (build vs buy); LLM provider and data regions permitted to process `internal` content ([§7.8](07-factory-metadata-registry.md#s7-8), [§19.7](19-nfr-enterprise.md#s19-7)).
24. Register the profile's extensions in the registry before use (PC-5), including the `release` channel for the pre-production gate ([§8.1](08-verification.md#s8-1), [§18.3](18-aisf-conformance.md#s18-3)); choose who assesses whether extensions conform to §20.3 of the Standard.
25. Task runner and orchestration: build or buy (a dedicated workflow engine, an existing orchestration platform, or continue deriving records from issue/PR/CI events as in Slices 0–1, [§9.3](09-agent-operating-model.md#s9-3)). Owner: Agent/Eval engineer ([§14.2](14-organization-and-operations.md#s14-2)).
26. Evaluate an existing developer portal or catalog (e.g., Backstage) to host or display the Knowledge Index and Test Asset Catalog, before building them ([§0.4](00-summary.md#s0-4), [§7.8](07-factory-metadata-registry.md#s7-8), [§8.11](08-verification.md#s8-11)).


**Deadlines.** Phase 0 only MUST finalize the decisions blocking Phase 1a; the remaining decisions are finalized before the phase that needs them, and are tracked as work items with owners from Phase 0 onward.

| Deadline | Items |
|---|---|
| Phase 0 exit (blocks Phase 1a) | 1, 2, 3, 7 (only for the first service used in Slice 0), 10, 11, 12, 13, 14, 18, 19 (commitment and owner), 23 (LLM provider and data regions), 24 |
| Before enabling attestation (Phase 1a follow-up) | 17 (target SLSA level) |
| Before Phase 1b | 5, 17 (browser and device matrix), 20, 21, 22 (owner of Documentation & Knowledge) |
| Before Phase 2 | 7 (remaining services), 8, 9, 16, 19 (staging and tenant pool operational), 23 (search engine for the Knowledge Index), 26 |
| Before Phase 3 | 25 |
| Before Phase 4 | 4, 22 (internal documentation surface) |
| Before Phase 5 | 6 |
| Before Phase 6 | 15 |

---

[Contents](README.md) · [← 16. Current state: problems and data, not constraints](16-current-state.md) · [18. AISF Standard 4.0 conformance (profile declaration) →](18-aisf-conformance.md)
