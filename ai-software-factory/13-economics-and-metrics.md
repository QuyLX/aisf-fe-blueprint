# 13. Factory economics and metrics

## 13.1. Metric principles

1. The Factory MUST NOT optimize raw PR count, generated lines, token volume, agent count or autonomy rate in isolation.
2. The Factory MUST measure the delivery system together with product quality and reliability guardrails.
3. The Factory MUST compare equivalent task classes and MUST use a baseline before drawing conclusions about productivity.

**Rationale.** Raw PR count, generated lines, token volume, agent count and autonomy rate can all increase without improving the delivery system. The delivery system is only evaluated correctly when measured together with product quality and reliability guardrails [R08, R10], and conclusions about productivity are only meaningful when compared against a baseline on equivalent task classes.

## 13.2. Metric catalog

1. The Factory MUST measure the metrics in the following catalog.

| Dimension | Metric | Interpretation |
|---|---|---|
| Flow | Change lead time | From task accepted to change accepted/released |
| Flow | Throughput of accepted and verified work items | Number of accepted changes with complete required evidence per period |
| Flow | Queue wait time | Time waiting for CI, review, approval or environment |
| Quality | Escaped defect rate | Defects found after mandatory gates/release |
| Quality | Change failure rate | Changes requiring rollback/hotfix or causing production defects |
| Quality | Revert rate | Changes reverted; interpret together with other failure metrics |
| Verification | AC evidence coverage | Acceptance criteria with valid evidence / mandatory AC |
| Verification | First-pass verification rate | Tasks passing mandatory checks without a repair loop |
| Verification | Flaky check rate | Number of unstable checks divided by number of check runs |
| Agent | Task success rate | Tasks fully meeting their acceptance contract |
| Agent | Human intervention rate | Tasks requiring out-of-routine human intervention |
| Agent | Repair loop rounds | Mean/P95 number of fix rounds |
| Agent | Tool error rate | Tool errors, permission errors, invalid outputs |
| Evaluation | Skill/agent regression rate | Eval cases that previously passed now failing |
| Knowledge | Metadata completeness/staleness | Coverage and freshness of important entities |
| Knowledge | Revalidation backlog | Number of knowledge artifacts in `NEEDS_REVALIDATION` and time pending, by authority (§5.8) |
| Knowledge | Knowledge conflict rate | Conflicts between implementation and intent per task, and the ratio resolved as bugs versus spec changes (§4.7) |
| Workflow | Per-workflow metrics | See §13.7 |
| Human | Attention minutes per accepted work item | Human review, approval, clarification and fix time divided by number of accepted work items; read together with the North Star (§13.6) |
| Human | People touching each task | Number of distinct people who must act (confirm, approve, answer, fix) on a work item; counts touches, not only approvals |
| Human | Review capacity utilization | Number of agent work items waiting versus declared review capacity, by area (§13.6) |
| Maintenance | Maintenance burden by owner | Number of repositories, services and public packages each team owns, and its trend (§6.12) |
| Architecture | Boundary violation rate | New violations per change or repository |
| Review | Human review time | Time spent on judgment, not just time waiting in the queue |
| Reliability | MTTR | Time to restore service as expected |
| Economics | Cost per accepted and verified work item | AI + CI + review + rework cost / number of accepted and verified work items |
| Capacity | Agent concurrency, CI queue, timeouts, retries | Factory saturation and bottlenecks |
| DevEx | Developer satisfaction / interruption rate | Whether the Factory improves actual engineering work |
| Review | Human approvals per PR | Median and P95 |
| Review | Human wait time | Time a PR waits for a person |
| Review | Rounds-to-pass | Number of review/fix rounds until all findings are resolved |
| Factory | Factory availability | Failure rate and downtime of the harness, CI integrations and agent runtime |
| Factory | Bypass rate | Share of eligible tasks done outside the Factory |
| Governance | Break-glass uses | Count, duration and post-hoc verification results |
| Governance | Conformance level by scope | Repositories and areas by L1–L4 (§20.4), and claims that fail on reassessment |

2. Human approvals per PR SHOULD decrease without guardrails degrading.

**Rationale.** The bypass rate (share of eligible tasks done outside the Factory) is the most honest adoption signal: when the Factory path is heavier than the manual path, people route around it.

## 13.3. North Star and guardrails

1. The Factory MUST use the following North Star:

```text
Cost per Verified Accepted Work Item =
  (AI cost + CI cost + human review cost + rework cost + attributable ops cost)
  / accepted work items (tasks) with required evidence
```

2. The North Star MUST be read together with:
   - Human attention minutes per accepted work item (§13.6).
   - Escaped defects.
   - Change failure/revert rate.
   - MTTR.
   - Security/policy violations.
   - Developer satisfaction.
3. A lower cost per change MUST NOT be treated as an improvement if defects, rework or operational incidents increase.

**Anti-gaming.**

4. Verification profile changes MUST be logged as metric-affecting events.
5. Verification profile changes MUST be reviewed monthly.

**Anti-splitting.**

6. The North Star MUST count **accepted tasks (outcomes)** instead of PRs; a task split into multiple PRs MUST be counted only once.
7. The North Star MUST be reported **per task class** and MUST NOT be aggregated into a single number.
8. The Factory MUST track batch size and PRs per task alongside the North Star.

**Factory cost.**

9. The cost of building and operating the Factory itself (team, infrastructure, licenses) MUST be reported alongside the North Star and MUST NOT be hidden inside it.

**Rationale.** Human time has two kinds of cost: review time, and time spent fixing what bad changes break. Reducing the first is meaningless if that time reappears in the second [R29].

Reference target pattern: most changes are merged with at most one person in the loop, without increasing reverts or incidents. The cited practice allows that person to be the author [R29]. The target of this Standard is stricter: no person required for eligible Low-tier changes, and **at most one independent approver** (plus the requester's sign-off) for Medium.

*Metric gaming.* The North Star denominator counts changes "with complete required evidence." Weakening a verification profile would improve the metric without improving quality; therefore profile changes are logged and reviewed (items 4–5).

*Splitting.* Cost per change decreases if work is split into many smaller changes, and the metric would weigh a documentation fix the same as a feature. Items 6–8 block these two effects.

*Factory cost.* If the cost of building and operating the Factory were folded into the North Star, it would be diluted across the number of work items and obscure the question of whether the Factory is worth the investment; reporting it alongside keeps that question visible.

## 13.4. Measurement design for the pilot

1. Before enabling the Factory, the Factory MUST collect a baseline for representative task groups:
   - Bug fixes.
   - Small features.
   - Refactoring.
   - Dependency upgrades.
   - API/contract changes where possible.
2. Measurement MUST include both elapsed time and human effort.
3. Measurement MUST separate coding time from queue wait time, review time and rework.
4. Measurement MUST compare task complexity and risk so that changes in task mix do not skew the results.
5. Human effort MUST be measured from system events (PR open, review and approval timestamps; number of review comments; CI wait time) plus periodic sampled surveys.
6. The Factory SHOULD NOT use manual self-reported timekeeping to measure human effort.

**Rationale.** Manual self-reported timekeeping distorts behavior and is rarely accurate.

## 13.5. Suggested pilot gates (require calibration, not to be treated as industry standards)

1. Pilot gates MUST be agreed with the repository owner after the baseline is collected.
2. The Factory MUST NOT choose numeric thresholds for task success rate, precision or defects before measuring the repository baseline and validating the dataset.

**Example.** Suggested starting gates (initial hypotheses, to be agreed with the repository owner after the baseline is collected):

- 100% of mandatory checks have a clear `PASS` or `FAIL` status, or an approved `NOT_ASSESSED`.
- 100% of protected-path changes and high-risk changes have human approval as configured.
- No unresolved blocker is auto-merged.
- Important acceptance criteria have traceable verification evidence.
- Agent/playbook evals show no regression on the critical eval set before promotion.
- Auto-merge starts only with a narrow, low-risk task group, after the pilot shows stable results on a representative sample.

This list is a starting point for calibration, not an industry standard.

## 13.6. Factory capacity planning

1. The Factory MUST plan for:
   - CI capacity, queue wait time and cost per run, as first-class Factory metrics.
   - Merge queue throughput and version control limits.
   - Concurrency limits per repository and per model provider.
   - Harness efficiency: fast, cached checks in the inner loop; deep checks on a schedule.
   - Compute and token resources for verification.
   - Agent run time.
2. The Factory MUST set an explicit budget for compute and token resources dedicated to verification.
3. Timeouts, checkpoints (§6.14) and runner capacity MUST be set according to the measured run time distribution (e.g. P90), not the mean.

**Human attention budget.**

4. The Factory SHOULD regulate by human attention rather than only measure it.
5. Each area SHOULD declare the review capacity of its owners; once declared, items 6–8 MUST apply (*suggested*: the number of agent work items that may wait on each reviewer at once). This capacity is the review WIP limit of §7.6.
6. When an area's review capacity is exhausted, the task runner MUST NOT initiate additional automation-created tasks for the area (§12.4) and MUST queue new tasks.
7. When an area's review capacity is exhausted, human-requested tasks MUST still be accepted and MUST clearly display the expected wait time.
8. The Factory MUST NOT lower checkpoints or approvals to clear the queue.
9. Instead, the Factory SHOULD reduce intake, batch related changes, or move recurring constraints to a stronger layer (from instruction to deterministic sensor; §3.5) so that each review requires less attention.
10. *Reference target:* a Low/Medium bug fix needs no more than two human touches outside exception handling: confirming the Change Brief and approving the PR. When this target is exceeded, the workflow MUST be revisited (§13.7).
11. The metric *human attention minutes per accepted work item* (§13.2) MUST be read together with the North Star.

**Rationale.** Agent throughput can grow faster than infrastructure plans assume. Reported figures:

- One organization reported some systems under roughly 10× load within about six months, with version control, CI/CD and the release process all overloaded [R28].
- Another organization reported that changes landed per day in a monorepo grew from about 2,500 to more than 30,000 in four months [R29].
- A third organization reported a team shipping about 11,000 PRs in one month with a revert rate of about 0.5% [R05].

These are signals of possible scale, not forecasts.

Practitioners report that most tokens consumed by agents can go to verification rather than code generation [R05]; this is why the verification budget is set separately. As models get stronger, each run gets longer and produces more work [R35]; the mean hides the tail of the distribution, so timeouts, checkpoints and runner capacity are set by percentile.

Machines can scale; the attention of reviewers, approvers and maintainers cannot [R35]. Without regulating intake by review capacity, increased throughput turns into review queues, and pressure to clear the queue easily leads to lowering checkpoints or approvals. Attention minutes are read together with the North Star because falling cost with rising attention minutes is a sign that the Factory is shifting the burden onto humans. Similarly, a Low/Medium bug fix needing more than two human touches outside exceptions is a ceremony signal.

## 13.7. Measuring workflow quality

1. The Factory MUST measure the quality of each workflow, by workflow spec `id` and version, and by task class.
2. The Factory MUST measure the following workflow metrics.

| Metric | Interpretation |
|---|---|
| Success rate | Workflow tasks reaching `COMPLETED` with all mandatory outputs |
| Failure rate | Tasks ending in `CANCELLED`, `REJECTED` or reverted, classified by cause and by the step/sub-state where failure occurred |
| Rework rate | Commits after the first review round, reopened tasks, changes reverted or requiring a forward fix |
| Retry count | Per step, separating `ERROR`/timeout from `FAIL`-driven repair loops (§6.14) |
| Human intervention rate | Tasks requiring human editing or steering outside declared checkpoints |
| People touching | Per task and per checkpoint (§13.2) |
| Approval wait time | Per checkpoint (§6.12) |
| Escalation rate | By cause: budget exhausted, basis changed, handoff rejected, profile conflict, knowledge conflict |
| Handoff rejection rate | By role pair (§6.13) |
| Handoff cost | Tokens, time and amount of context passed through each handoff |
| Lead time by sub-state | Time in each sub-state, to find workflow bottlenecks |
| Cost | AI + CI + human time per completed workflow task |

**Comparison by version.**

3. A new workflow version MUST be evaluated like a skill (§9.2): it MUST run on the task class eval set before being promoted to `ACTIVE`, and MUST be compared with the previous version on the same task mix.

**Improvement must not weaken controls.** A workflow spec is a control file (§7.13). A change that removes or downgrades a `mandatory` step, a checkpoint or a gate, or that raises a budget, is a loosening change.

4. A metric improved by removing a gate MUST NOT be counted as an improvement.
5. The change removing that gate MUST be logged as a metric-affecting event (§13.3).

**Read together with guardrails.**

6. Workflow metrics MUST be read together with escaped defects, change failure rate and revert rate.
7. Increased speed with worsening guardrails MUST be treated as degradation.

**Review cadence.**

8. Workflow metrics MUST be reviewed monthly.
9. Workflow metrics MUST be an input for promoting a workflow from outline to `ACTIVE` (§6.7.2).

**Rationale.** Measuring only the agent's final outcome is not enough: two workflows with the same success rate can differ greatly in retries, approval wait time or the number of times humans must intervene. Measuring by `id`, version and task class shows which workflow, at which step or sub-state, is creating a bottleneck. Costly handoffs that do not increase independence are a sign that roles should be merged (§6.8).
