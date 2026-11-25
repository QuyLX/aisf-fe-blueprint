[Contents](README.md) · [← 11. Delivery & Change Management](11-delivery-change-management.md) · [13. Metrics →](13-metrics.md)

<a id="s12"></a>

# 12. Evaluation

| Layer | Content | When it runs |
|---|---|---|
| **Regression** | 5–10 cases per (archetype × task class) in the wave | Every change of model/playbook/skill/check/classifier |
| **Trust qualification** | Real tasks in shadow/assisted mode | Continuously |
| **Seeded defects** | Intentionally inserted defects to measure check recall; **dev set** (visible to the check author) and **hold-out set** (held by someone else, rotated each quarter) — published recall is measured on the hold-out | On check change, periodically |
| **Agent legibility** | Same UI prompt, different configuration (old vs new DS/AGENTS.md/docs); blind grading; no answer leakage; a new agent per prompt; measure build pass, correct component use, escape hatches, a11y, tokens, cost; degradation by conversation length | Nightly + on change of DS API, AGENTS.md, CLI or skill |
| **Reviewer quality** | PRs drawn from incidents and reverts (bug location confirmed), known-clean PRs, injection fixtures; measure precision, recall, harmful suggestion rate (Standard §7.12) and rounds-to-pass (Standard §7.11) | Every change of the reviewer's prompt, model or filter; daily |
| **Test selection** | A seeded defect is placed in a file; measure whether the TestPlan ([§8.10](08-verification.md#s8-10)) selects at least one test that catches it; compare with actual selection misses from nightly | On change of the Test Planner or its configuration; weekly |
| **Knowledge retrieval** | A standard question set with known answers (governing ADR, correct contract version, owner, suitable component), with traps: a `DRAFT` record conflicting with an `ACTIVE` one, an old contract version, an `ARCHIVED` document, a `restricted` document the asker has no access to. Measure: correct source in top-k, correct version, correct citation, rate of returning non-`ACTIVE` documents without flagging (target = 0), access leakage (target = 0) | On change of the Knowledge Index, ranking, embedding model or retrieval skill; weekly |

**History replay** (fixed bug → reassigned → graded with hidden tests) only uses cases with tests that check **observable behavior**, with a sample re-graded by humans to measure "false fails". On a new platform, this source grows over time.

Graders: deterministic first; LLM-as-judge only for qualitative rubrics, and it MUST be calibrated against humans.

**Gardening** (periodic): stale/vulnerable dependencies, boundary drift, unused components/tokens, broken docs links, documents whose source has changed but have not been re-reviewed, `ACTIVE` records past their review date, overdue flags, expiring waivers, Journey Registry entries missing tests, orphan tests and unused fixtures ([§8.11](08-verification.md#s8-11)), tests quarantined past due, foundation dependencies nearing EOL ([§4.8](04-technical-foundation.md#s4-8)) — initially reports only; PRs are created once the task class has reached trust.

---

[Contents](README.md) · [← 11. Delivery & Change Management](11-delivery-change-management.md) · [13. Metrics →](13-metrics.md)
