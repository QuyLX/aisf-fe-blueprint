# 18. Open architecture decisions

These decisions must be made through the pilot rather than guessed in advance:

1. The Git hosting and CI platform used as the source of truth for enforcement, and how controls are implemented on that platform (§14.4). Decisions for items 1–3 are settled in Phase 0.
2. Which coding-agent runtime/provider is approved for each data class (§10.6).
3. Local, cloud or ephemeral CI runtime for each task group.
4. Whether initial metadata aggregation needs a service or only a Git-based index.
5. How repository identity, product identity and service identity map to one another.
6. Which tool/MCP integrations are approved and who owns them (§10.8).
7. Which paths/domains always require human approval.
8. Which task classes are eligible for automated PR creation or auto-merge.
9. Which product-specific verification profiles are needed first.
10. Retention periods for artifacts and traces, beyond the suggested defaults of the retention classes (§10.7).
11. How eval datasets are access-controlled and versioned.
12. Who owns the shared Factory product roadmap and adoption.
13. Which baseline thresholds justify expanding from the pilot to other repositories.
14. Which areas opt in to zero-human merge for the Low tier, and who owns that decision.
15. Circuit breaker signals, thresholds and windows (§9.9).
16. Review depth (Quick / Standard / Deep) per area.
17. Which commit-creation mechanisms are allowed to carry approval over to a new commit (§9.8).
18. Whether AI review uses a different model or provider from the implementation.
19. Separation of duties (SoD) obligations from compliance (e.g., SOC 2, SOX change management) and how they map to §9.10.
20. AI review promotion thresholds (shadow → advisory → blocking) per area.
21. Who holds the Standard owner role, and who is allowed to independently assess conformance claims (§17.4, §20.5). This is a prerequisite: no claim in `standard` mode is accepted before both are named explicitly (§17.4 item 18).
22. Availability SLOs for Factory components and the break-glass window (§12.7).
23. Which non-PR change channels are governed first, and their declared sensitivity types (§12.6).
24. Evidence windows and thresholds for T1 and T2 beyond the suggested minimums (§9.4).
25. How difficulty is stratified, the holdout ratio and the critical scenario set for each task class (§9.4).
26. Which workflows are promoted from outline to `ACTIVE` after Bug Fix, and in what order (§6.7.2).
27. Quantitative thresholds for promoting the knowledge dependency index to a graph service (§5.9).
28. Periodic review cadence for critical knowledge artifacts, and `NEEDS_REVALIDATION` resolution deadlines by authority (§5.8).
29. Declared review capacity for each area, and how automation-generated tasks are queued when capacity is exceeded (§13.6).
30. Which production signal sources are connected first, fingerprint rules for deduplication, task-creation thresholds and the Lookback window (§12.4).

## 18.1. Intentionally deferred decisions

- A full central graph database, including for knowledge dependencies (§5.9).
- A custom-built workflow/orchestration engine (§14.1).
- A full Backstage deployment.
- A custom-built agent runtime/model serving.
- Organization-wide auto-merge.
- A universal micro-agent architecture.
- A single global quality score.
- Agents automatically rolling back production (artifact rollback or revert) or recovering data. Flag-off/pause within policy limits by the Change Shepherd (§12.2.1) is explicitly allowed and is not considered a rollback.

Each deferred item requires a proven use case, an owner, a risk review and a cost/benefit analysis.
