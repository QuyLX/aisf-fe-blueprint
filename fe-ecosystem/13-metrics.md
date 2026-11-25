[Contents](README.md) · [← 12. Evaluation](12-evaluation.md) · [14. Organization & operations →](14-organization-and-operations.md)

<a id="s13"></a>

# 13. Metrics

| Group | Metric |
|---|---|
| Flow | Lead time per work item (p50/p85), queue time |
| Quality | Escaped defects, change failure, revert |
| Platform | Number of repos/modules by L1–L4, % created by generator, drift, **version lag** of Next.js/`@platform/*` per repo, adoption time after release |
| Platform as a product | Vertical owner satisfaction, support response time, number of escape hatches/golden path exceptions |
| Check health | Checks by lifecycle vs budget, FP rate (owner-confirmed), dispute rate, waivers per check, recall on hold-out |
| Verification | Evidence coverage, first-pass rate, flaky rate (Testing Platform details in the table below) |
| A11y / Perf | Critical violations; CWV field data (public), route budget (app) |
| Agent | Task success, intervention, repair loop, task classes by trust level |
| Content | Blocked publishes, post-publish errors |
| DX | Developer satisfaction, time to first successful task |
| Economics | **Cost per Verified Accepted Work Item** by change class (AI + CI + review + rework) |
| Operations | Error rate, MTTR, rollback time |
| Review & governance | Human approvals per work item (median, p95), human wait time, rounds-to-pass, finding disposition distribution, waivers per approver, break-glass uses, bypass rate |
| Conformance | Repos and surfaces by claimed level, expired or failed claims, showcase scenarios passing ([§20](20-showcase.md#s20)) |

**Thresholds.** Each metric has one threshold type: **Baseline** = measure first, no target set yet; **Suggested** = proposed target, finalized once ≥ 1 quarter of data exists; **Invariant** = always 0 because it is a safety failure (e.g., data leakage). No hard targets are set for metrics without data. The numbers in the roadmap exits ([§15](15-roadmap.md#s15)) also follow this convention.

**Quality of the Testing Platform itself:**

| Metric | Definition | Threshold |
|---|---|---|
| Run time per command | p50/p95 of `verify:fast`, `verify`, `verify:runtime` against the budgets in [§7.6](07-factory-metadata-registry.md#s7-6) | Suggested: [§7.6](07-factory-metadata-registry.md#s7-6) budgets |
| Runner wait time | From job queued to job start, p50/p95 | Baseline |
| Failure attribution | `FAIL`/`ERROR` rate by `product`, `test`, `environment`, `test-data`, `harness` ([§8.8](08-verification.md#s8-8)) | Baseline; the non-`product` share is tracked downward |
| Environment availability | % of time preview and staging are healthy; number of environment incidents | Baseline |
| Flaky rate | Fail-then-pass runs / total runs, per surface; average age of quarantined tests | Suggested ≤ 2% (L3 criterion, finalized after baseline) |
| Selection miss rate | Selection misses / number of PRs ([§8.10](08-verification.md#s8-10)); separately for `critical` journeys | Baseline; selection misses on `critical` are handled as findings |
| Critical journey evidence coverage | % of combinations (`critical` journey × persona × scenario) with `PASS` evidence in the last 30 days | Suggested 100% for the L3 criterion |
| Traceability coverage | % of `critical` journey ACs with tests and evidence | Baseline |
| Test asset maintenance cost | Person-hours fixing tests not caused by product defects; ratio of test changes per product change | Baseline |
| Test recall | Seeded defect recall on hold-out ([§12](12-evaluation.md#s12)) | Suggested per archetype |

**Quality of Documentation & Knowledge:**

| Metric | Definition | Threshold |
|---|---|---|
| Staleness | % of documents whose source (contract, package, code) changed after the latest review; % of `ACTIVE` records past their review date | Baseline |
| Docs update time | From contract or feature release to publication of the corresponding docs, p50/p85 | Baseline |
| Zero-result searches | % of queries on the Docs Portal and Knowledge Index returning nothing | Baseline |
| Correct document retrieval | Retrieval eval: correct source in top-k, correct version ([§12](12-evaluation.md#s12)) | Suggested, finalized after baseline |
| Runnable code samples | % of samples passing `docs.code-samples` | Suggested 100% for published samples |
| Broken links | Number of broken links on the Docs Portal and in internal documentation | Suggested 0 on the Docs Portal |
| Agent uses a non-`ACTIVE` source without flagging | Occurrences in task logs | Invariant = 0 |
| Access leakage in retrieval | Results returned to someone without access at the source | Invariant = 0 |

**Workflow quality (Standard §13.7):** measured per playbook `id@version` and per task class: success rate, failure rate by step or sub-state, rework rate, retry count (separating `ERROR`/timeout from repair loops), rate of human intervention outside checkpoints, wait time per checkpoint, escalation rate by cause, rejected handoff rate by role pair, handoff cost, lead time by sub-state and cost per task. A new playbook version is compared against the previous version on the same task mix. A metric improvement achieved by removing a `mandatory` step, checkpoint or gate does not count as an improvement; that change is a loosening ([§9.2](09-agent-operating-model.md#s9-2)) and is logged as an event affecting the metrics.

**North Star:** Cost per Verified Accepted Work Item (Standard §13.3), reported per task class — the denominator is the **work item**, not the PR. Mandatory guardrails: escaped defects, change failure, security/a11y blockers, MTTR, developer satisfaction. Never collapsed into a single score.

**Per product (example):** Admin — unaudited actions (= 0), overdue impersonation sessions (= 0); Desk — internal note exposure (= 0), SLA errors; Chat — UI-side TTFT, stream error rate, conversation content leaking into telemetry (= 0); Website — CWV field data, blocked publishes; Docs — broken links, code sample pass rate; DS — adoption, breaking changes.

---

[Contents](README.md) · [← 12. Evaluation](12-evaluation.md) · [14. Organization & operations →](14-organization-and-operations.md)
