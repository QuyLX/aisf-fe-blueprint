# 9. Agent evaluation, trust and governance

## 9.1. Evaluation object model

1. The Factory MUST evaluate each independently versioned component of the system:
   - Model/provider configuration.
   - Role definitions.
   - Skills.
   - Playbooks (Task Workflow Specification).
   - Tool/MCP integrations.
   - Context retrieval strategies.
   - Reviewers/graders.
   - Verification profiles and verification adapters.
2. An agent benchmark SHOULD include realistic inputs, expected behavior, reference findings or evidence, severity and a scoring rubric.
3. An agent benchmark SHOULD include both positive and negative cases: PRs with known bugs, clean PRs, ambiguous tasks, security-sensitive tasks, tool failures, outdated documentation and prompt injection attempts.

## 9.2. Evaluation lifecycle

1. The Factory MUST build a small, realistic dataset from real engineering tasks.
2. The Factory MUST establish a baseline for the current process.
3. The Factory MUST run the agent/playbook on fixed cases.
4. The Factory MUST inspect traces and failure modes.
5. The Factory SHOULD automatically score deterministically verifiable results when possible.
6. The Factory MUST use human calibration for subjective dimensions.
7. The Factory SHOULD change only one main variable at a time when feasible.
8. The Factory MUST rerun evals when the model, prompt, skill, tool or policy changes.
9. The Factory MUST promote only versions that meet the defined quality and safety thresholds.
10. The Factory MUST add escaped defects and confirmed clean examples to the dataset.
11. The dataset SHOULD be seeded from the organization's history: PRs that caused incidents or were reverted (with confirmed defect locations) and PRs known to be clean (§7.12).
12. A candidate version of a model, prompt, skill, tool or reviewer MUST NOT be promoted if it regresses on this dataset.
13. When humans score, the scoring MUST be blind.
14. Evals SHOULD be run repeatedly.
15. Every reported number MUST be explained (variance, limiting factors, errors) before acting on it.
16. The Factory SHOULD start with a right-sized dataset of representative tasks.

**Rationale.** The organization's history is the best seed source: PRs that caused incidents or were reverted show which kinds of defects actually escape, while PRs known to be clean show whether the reviewer proposes harmful changes (§7.12). Blind scoring, repeated runs and explaining every number before acting are practices recommended in [R06]: an eval number without variance easily leads to a promotion decision based on noise.

For a pilot, 20–50 cases is a practical starting point, not a universal benchmark standard. A right-sized dataset lets the team start measuring early, then grow as escaped defects and clean examples are added.

## 9.3. Evaluation metrics

1. When reporting the evaluation metrics below, the Factory MUST use the definitions in the table:

| Metric | Meaning |
|---|---|
| Task success rate | Proportion of tasks that meet all required acceptance criteria |
| Requirement coverage | Proportion of acceptance criteria with valid verification evidence |
| Reviewer precision | Proportion of reported findings confirmed as valid |
| Reviewer recall | Proportion of planted/confirmed issues that were detected |
| False-positive rate | Proportion of findings that are invalid or not actionable |
| Repair loop count | Number of repair loops before completion/escalation |
| Human intervention rate | Tasks that needed human edits or direction beyond the norm |
| Context retrieval precision | Relevant context retrieved divided by all context retrieved |
| Context retrieval recall | Relevant context found divided by known relevant context |
| Tool failure rate | Calls that failed due to schema, permission, availability or runtime |
| Cost per successful task | AI + CI + review + rework cost divided by the number of successful tasks |
| Regression rate | Eval cases that previously passed but fail after a change |
| Rounds-to-pass | Number of review/fix rounds before all findings are resolved (mean; proportion needing 5 or more rounds) |
| Finding disposition mix | Proportion of findings fixed / judged wrong / intentional / unimportant |
| Harmful-suggestion rate | Findings on a known-clean PR that would lead to a harmful change |

## 9.4. Agent Trust Profile

An **agent configuration** is a pinned model snapshot together with the playbook, skills, tools and prompt.

A **task class** is explicitly defined by: an `ACTIVE` workflow spec (`id` and major version), a scope (repository, area or surface) and a ceiling tier.

**Acceptance** means merged without the reviewer changing anything significant (nits only).

`propose-only` is not a trust level. The `allowedAutonomy` values are ordered `propose-only` < `implement-and-request-review` < `verify-and-open-pr` < `merge-on-eligible-path`.

1. Trust MUST be tied to the pair **(task class, agent configuration)**, not to a single organization-wide model score.
2. Trust earned in one task class MUST NOT be transferred to another class, even if it has the same workflow but a different scope or a higher ceiling tier.
3. The Factory MUST apply the following trust level table:

| Level | Meaning | Minimum evidence (*suggested default*) | `allowedAutonomy` |
|---|---|---|---|
| **T0** | Default; no qualifying evidence yet | — | `implement-and-request-review` |
| **T1** | Qualified in shadow or assisted mode | ≥ 30 tasks of the class, stratified by difficulty; acceptance ≥ 90%; 0 escaped blockers; 100% of critical scenarios pass; no holdout regression | `verify-and-open-pr` |
| **T2** | Qualified through a supervised production record | ≥ 50 T1 tasks merged to production, stratified by difficulty; 0 escaped defects in ≥ 8 weeks; 95% confidence upper bound of the defect rate ≤ 6% (*suggested default*); intervention rate below the agreed threshold; 100% of critical scenarios and holdout pass; no regression | `merge-on-eligible-path` |

4. A profile MAY raise the minimum evidence in the table above.
5. The policy MUST declare, for each protected category (§9.6), whether that category is subject to `propose-only`. The `auth-and-secrets` category MUST default to `propose-only`. When a category is subject to `propose-only`, it MUST be in effect regardless of trust level.
6. Logic changes, closed PRs and reverted PRs MUST NOT be counted as accepted.
7. Trust MUST NOT loosen the controls of the Medium or High tier (§9.5).
8. **Low volume.** The evidence window MUST be at most 6 months, and the Factory MUST NOT merge task classes to reach the required count.
9. **Version changes.**
   - A *major* change (model family or vendor, major version of the playbook) MUST cap the class at T1 and MUST mark it `RE-EVALUATION_REQUIRED`.
   - A *minor* change (a snapshot of the same model family, a playbook or skill patch) MUST run the regression evals, and the level MUST be kept only if they pass (§9.9).
10. **Incident.** An escaped defect attributed to the class MUST lower its trust by at least one level and MUST trigger re-evaluation; a profile MAY reset it to T0.
11. **Trust evaluation.** Trust evaluation MUST NOT merely count successful tasks. Each evaluation MUST consider all five components:
    1. *Representativeness and difficulty.* The task set MUST be stratified by difficulty (for example diff size, number of modules, whether there are non-functional acceptance criteria) with a distribution comparable to the real work of the class; each stratum MUST have a minimum sample size set by the profile; near-duplicate tasks MUST be counted only once.
    2. *Critical scenarios.* The area owner MUST define, for each class, a set of scenarios in which a failure would be severe (security-sensitive, data loss, concurrency, compatibility). T1 and T2 MUST pass the entire set.
    3. *Holdout.* A portion of the eval set MUST be kept hidden and not used to tune the configuration's prompts, skills or playbooks. A gap between the holdout score and the seen-set score that exceeds the profile's threshold is a sign of overfitting and MUST block promotion.
    4. *Regression.* There MUST be no regression on the class's eval set (§9.2), nor on the eval sets of other classes at T1 or above that share the changed component.
    5. *Reliability of the result.* The decision MUST be based on a confidence bound (for example the 95% upper bound of the defect rate by the rule of three or a Wilson interval), and MUST NOT be based on the score ratio.
12. The result of a trust evaluation MUST be recorded as a `TrustEvaluation` (Appendix K).
13. Trust MUST be based on: the evaluation history for the same task class; the task success rate and the escaped defect rate; the human intervention rate and the repair loop rate; verification quality and evidence completeness; security and policy violations; and the recency and representativeness of the evaluation data.
14. Execution trust T2 (§9.5, rule 3) is a necessary condition for the zero-human path but MUST NOT by itself be considered sufficient: the other conditions in §9.5 and §9.10 MUST still apply.

**Rationale.** An agent that does one kind of task well is not thereby trusted for every kind of work (P11); therefore trust belongs to the pair (task class, agent configuration) and does not transfer between classes.

It is necessary to know what the numbers prove. Zero failures in *n* trials bounds the true failure rate at about 3/*n* with 95% confidence (the rule of three), so 50 clean tasks support only the conclusion "below about 6%" — that is the origin of the T2 6% upper-bound threshold. That is enough for zero-human merge at the Low tier and no more, so trust does not loosen the Medium or High tier (item 7). For the same statistical reason, the decision rests on a confidence bound rather than a score ratio, the evidence window is limited and task classes are not merged: merging different classes makes the sample size look sufficient but no longer says anything about the specific class.

**Example.** A trust profile:

```yaml
agent: frontend-implementer
configuration: { model: provider/model-2026-09-15, playbook: bug-fix@1.4, skills: [repro@2.1, verify@1.3] }
trust:
  docs.link-fix:             { level: T2 }
  web.copy-change:           { level: T1 }
  ordinary-feature-change:   { level: T0 }
  auth-or-permission-change: { level: T0, policy: propose-only }
```

The same agent configuration has a different trust level for each task class, and the class `auth-or-permission-change` is subject to `propose-only` by policy regardless of level.

## 9.5. Risk, trust, reversibility and autonomy level

The autonomy decision MUST consider three dimensions:

- **Task risk:** the impact if the change is wrong (tier, §9.6).
- **Agent/workflow trust:** evidence that the agent configuration performs this task class reliably (§9.4).
- **Reversibility:** how easily the change can be undone or recovered from.

The decision rules (items 1–5) and the following requirements:

1. **Start from the risk tier** of the change (§9.6). The Factory MUST use the tier as the basis for determining the lightest permitted autonomy level.
2. **Reversibility can only tighten.** Reversibility MUST NOT loosen controls. If reversibility is LOW (destructive data changes, irreversible external side effects, released artifacts that cannot be recalled, messages already sent), the Factory MUST apply the controls of the next higher tier.
3. **Trust can only loosen within a tier, never beyond the tier.** Trust MUST NOT loosen controls beyond the tier. A change is **eligible** for the lighter path of the tier if and only if both of the following are true:
   - *Ownership trust:* the requester belongs to the team that owns every affected area.
   - *Execution trust:* the agent configuration has trust level T2 for this task class (§9.4).
4. **Unknown means worst case.** The Factory MUST treat: unknown risk → High; unknown reversibility → LOW; no trust evidence → T0; unknown requester → not eligible.
5. **Protected categories override everything.** The security, legal, data, public API, CI/release workflow, production operations and control file (§7.13) categories MUST override all other rules. Each category has a floor tier in the table in §9.6 (`protectedCategory` in the vocabulary).
6. **Autonomy level.** Each level is shorthand for a set of approval slots; §9.10 is the authoritative source on who may fill a slot. The Factory MUST apply the following table:

| Level | Approval slots | Typical use case |
|---|---|---|
| **A0** | None. Permitted only when all zero-human conditions below are true | Eligible Low-tier changes in opted-in areas |
| **A1** | One slot. The requester's sign-off fills this slot only if the tier is Low, the requester belongs to the team that owns every affected area, no circuit breaker for the area or task class is tripped (§9.9), and there is no `blocking` integrity sensor hit; otherwise an independent code owner of each affected area fills the slot | Low tier, not eligible |
| **A2** | One **independent** slot. The requester signs off but does not count. A code owner fills the slot when the area is listed by the owner as sensitive or the profile requires it | Medium tier |
| **A3** | A human-approved plan before implementation, plus two **independent** slots, at least one filled by a code owner of each affected area; staged rollout and a recovery plan | High tier |

7. The Factory MUST add extra slots to the level by union (§9.10): a code owner for each control file touched, the security owner, a business owner for declared sensitive content, and any domain owner the profile designates (for example a design-system owner).
8. **Lightest permitted level.** The Factory MUST NOT use a level lighter than the level in the following table:

| Risk tier | Reversibility | Eligible | Lightest level |
|---|---|---|---|
| Low | HIGH / MEDIUM | Yes, and all zero-human conditions are true | A0 |
| Low | HIGH / MEDIUM | No | A1 |
| Low | LOW | Any | A2 (treated as Medium) |
| Medium | HIGH / MEDIUM | Any | A2 |
| Medium | LOW | Any | A3 (treated as High) |
| High | Any | Any | A3 |

9. A profile MAY require a heavier level in any cell.
10. A profile MUST NOT permit a lighter level.
11. **Zero-human conditions (A0).** A change MUST take the A0 path only when all of the following are true; if any is not met, the change MUST fall to the lightest level that the matrix above allows when not eligible (A1 for Low tier with HIGH or MEDIUM reversibility; heavier when the tier or reversibility requires):
    1. The change is Low tier, reversibility is HIGH or MEDIUM, and it is eligible (rule 3).
    2. The area has opted in to zero-human merge (§9.6), and the opt-in is recorded in a control file.
    3. AI review runs in `blocking` mode for the area and reports no open findings, and there are no open security findings.
    4. All deterministic gates pass on their own. A clean AI review is a necessary condition, never a sufficient one (§7.9).
    5. No control file is touched, except by a trusted mechanism (§7.13), and no integrity sensor in `blocking` mode is triggered.
    6. No circuit breaker for the area or task class is tripped (§9.9), the area's error budget is not exhausted, and no release freeze is in effect.
12. High agent trust MUST NOT override mandatory approval rules for security, legal, data, public API or production.

## 9.6. Risk tier

1. The Factory MUST apply the following mandatory controls by tier:

| Tier | Examples | Mandatory controls |
|---|---|---|
| Low | Documentation, copy, internal additive changes, isolated styling | Quick checks, mandatory evidence; zero-human merge only on the eligible path, otherwise 1 approval (§9.10) |
| Medium | CRUD, ordinary business logic, API integrations, non-breaking SDK changes | Full profile, independent targeted tests, 1 independent human approval (§9.10) |
| High | Authentication, authorization, billing, payments, data migrations, breaking public API changes, release/security pipelines | Deep verification, threat/impact analysis, 2 independent approvals, with at least one code owner of each affected area (security owner where applicable), recovery plan, staged rollout |

2. The Factory MUST infer risk from the change path, metadata, data sensitivity, permissions, dependency blast radius and semantic analysis.
3. A classifier MAY raise risk but MUST NOT lower protected categories.
4. **The change determines the tier, not the author** (P19). The tier MUST be determined from the change, MUST NOT from the author.
5. The Factory MUST use two independent decisions: a **tier classifier**, and an **override check** that asks whether this specific change needs a human even if its tier says it does not. Either MAY raise the requirement; neither MAY lower a protected category. In pilot mode, the tier classifier MAY be replaced by a deterministic path → tier mapping table per §20.8 item 4.
6. Sensitive paths listed by owners and **all control files** (§7.13), including CI/CD workflow files, MUST always require one independent human approval, except for control file changes produced by a registered trusted mechanism per §7.13. This requirement MUST be enforced through CODEOWNERS and branch protection, MUST NOT through prompts.
7. The lighter path MUST require eligibility as defined in §9.5 rule 3 (ownership trust plus execution trust).
8. A change whose requester is outside the owning team of an affected area MUST require additional approvers per the corresponding rows of the §9.10 table.
9. Zero-human merge for the Low tier MUST exist only in areas that have **opted in**; zero-human merge MUST NOT be the organization-wide default.
10. Opt-in MUST require AI review in `blocking` mode for the area (§7.9).
11. **An open security finding MUST raise the change to at least Medium** until it is fixed or dismissed per §9.10.
12. The Factory SHOULD **start strict and loosen based on evidence.**
13. The Factory MUST apply the following default autonomy level and number of approvals by tier:

| Tier | Lightest autonomy level (§9.5) | Default number of human approvals |
|---|---|---|
| Low | A0 when all zero-human conditions are true; otherwise A1 | 0 or 1 (§9.10) |
| Medium | A2 | 1 independent |
| High, and paths that always require two people | A3 | 2 independent, with at least one code owner of each affected area |

14. **Floor tier of protected categories.** A classifier MAY raise the tier above the floor, MUST NOT lower it below the floor. The Factory MUST apply the following table:

| Protected category | Floor tier |
|---|---|
| Authentication, authorization, secret or key management | High |
| Billing, payments, pricing | High |
| Legal: terms, privacy, legally obligated content | High |
| Data: migrations, destructive deletion or transformation of data, personal or regulated data | High |
| Public API or contract: breaking changes | High |
| Public API or contract: additive changes | Medium |
| CI/CD workflow files, merge queue configuration and branch protection (`ci-release-and-branch-protection`) | Medium when tightening or neutral; High when loosening (classified by direction like control files, §7.13) |
| Production operations and data plane, including executing a release to production | High |
| Control files (§7.13) | Medium when tightening or neutral; High when loosening |

15. **Sub-tiers.** A profile MAY subdivide tiers (for example R0 and R1 within Low) to attach different checks or autonomy levels. Each sub-tier MUST map to exactly one standard tier, and the controls of that tier MUST be the floor.

The authoritative approval rules are in §9.10.

**Rationale.** The tier assignment rules are based on practices published in [R29]. The additional approver for changes from outside the owning team (item 8) exists so that someone asks "should we be doing this at all?". The §9.10 table defines that person for each tier: at the Low tier it is a code owner of the area, at the Medium tier it is the second independent approver, at the High tier it is already in the two-person rule.

CI/CD workflow files, merge queue configuration and branch protection have a floor tier that follows the same direction as control files, because a fixed High floor for every CI change (even adding a check) discourages teams from improving the pipeline; executing a release to production and data plane operations remain High.

Per-area opt-in to zero-human merge (item 9) follows the practices in [R28, R29].

## 9.7. Human accountability

1. The Factory MUST assign accountability to roles according to the following table:

| Role | Accountability |
|---|---|
| Product/Domain Owner | Intent, scope, business rules, acceptance, prioritization |
| Engineering Architect | Architecture boundaries, design trade-offs, exception approval |
| Repository/Code Owner | Local standards, code review, approval of protected paths |
| Factory Owner | Shared schemas, policy, workflows, platform reliability |
| Skill/Playbook Owner | Skill quality, workflow maintenance, evaluation |
| Security/Privacy Owner | Data classification, tool permissions, threat model, security gates |
| SRE/Operations Owner | Release readiness, SLOs, recovery, incident response |
| Agent Operator / Developer | Task supervision, understanding the change, accountable handoff |

2. Humans MUST remain in the loop to:
   - Decide **whether to build at all** (direction, taste, scope).
   - Review **risky code** (billing, authentication, updater, CI/release workflows).
   - Review **architecture that sets a pattern** that others will copy.
   - **Read the evidence**, not just the diff.
   - **Take responsibility for the outcome** of every released change, regardless of who typed it.
3. Disagreement about direction SHOULD be raised early in the task thread, SHOULD NOT be raised as a blocking comment on a finished PR.

**Rationale.** The list of what humans keep in the loop is based on [R29]. Disagreement about direction should be raised early because agent-written code is very cheap: closing a PR is not a loss, while a blocking comment on a finished PR wastes both the implementation and the review effort.

## 9.8. Approval rules

A **policy-approved commit-producing mechanism** is a commit-producing mechanism whose result the system can guarantee (for example a conflict-free rebase in a merge queue that reruns checks). This concept is distinct from the trusted mechanism of §7.13, which is automation that may only tighten control files.

1. An agent's approval MUST NOT be counted as a human approval (P20).
2. The accountable requester's sign-off MUST count as an approval only when §9.10 allows it (Low tier).
3. **An approval MUST be bound to a commit SHA.**
4. New commits that the system cannot guarantee (an agent push after approval, a manually resolved merge) MUST invalidate the approval.
5. Commits produced by a policy-approved commit-producing mechanism MAY carry the approval over if the policy allows.
6. A stale approval MUST be automatically dismissed on a new commit through branch protection.

## 9.9. Circuit breakers for autonomy

1. The Factory MUST put the circuit breakers below into operation, and rehearse them, before enabling the A0 path in any area. In an area where A0 is not enabled, the Factory SHOULD operate them.
2. The Factory MUST apply the following automatic responses:

| Signal (per area, sliding window) | Automatic response |
|---|---|
| Revert rate or change-failure rate exceeds the agreed threshold | Pause zero-human merge for the area; Low tier requires 1 **independent** approval (the requester's sign-off no longer fills the A1 slot) |
| Escaped defect or incident related to an agent task class | Lower the trust of that task class by one level; trigger re-evaluation |
| Rate of harmful suggestions or wrong reviewer findings rises | Pin the reviewer to the most recent good version; disable auto-fix for its findings |
| Agent violates policy or security | Revoke the agent's elevated scopes; restore only after a security review |
| Version change of the model, prompt, tool or skill | Mark the affected trust profiles `RE-EVALUATION_REQUIRED` until evals pass |
| The area's error budget is exhausted (§12.2) | Close the A0 path for the area until the budget recovers |
| Break-glass used in the area (§12.7) | Close the A0 path until post-incident verification is complete |
| Workflow spec changes major version | Cap the task classes using that workflow at T1 and mark `RE-EVALUATION_REQUIRED` |

3. A breaker MUST tighten automatically and MUST be loosened only by a human decision with evidence.
4. Breaker thresholds MUST be set after a baseline exists (§13.4).
5. **Minimum sample size.** A rate MUST be evaluated only when the number of changes in the window exceeds a minimum. In low-volume areas, the breaker MUST use an absolute trigger instead (for example, any revert of a zero-human merge pauses that path for the area until a human reviews).
6. **Signal latency.** A breaker MUST combine reverts and incidents with early indicators: a hotfix within 48 hours after merge, post-merge check failures, Change Shepherd pause events, and recurring defects detected by Lookback (§12.4).
7. **Attribution.**
   - Every change MUST link to its task record, so that a revert automatically maps to its task class.
   - Escaped defects MUST be attributed in the incident review and confirmed by the area owner.
   - Defects that cannot be attributed MUST be counted against the area, MUST NOT against the task class.
8. **Model pinning.**
   - The agent configuration MUST reference an immutable model snapshot, MUST NOT use a floating alias.
   - Every run MUST record the model version reported by the provider.

**Rationale.** Published practice points to a gap: detecting a bad merge still depends on humans and dashboards, and nothing automatically tightens approval when quality signals change [R29]. Circuit breakers fill this gap, so they are needed before zero-human merge is expanded.

- **Minimum sample size** (item 5): one revert in five PRs is not a 20% rate signal.
- **Signal latency** (item 6): reverts and incidents lag the merge by days, so relying on them alone would make the breaker trip too late; early indicators shorten that lag.
- **Attribution** (item 7): linking every change to its task record lets a revert map to its task class automatically without manual investigation.
- **Model pinning** (item 8): with a floating alias, a provider-side model update would never fire the "version change" trigger, and the trust profile would continue to apply to a model that has never been evaluated.

## 9.10. Separation of duties

The table below is the single normative source on who may approve or dismiss what. Other sections refer to it. The autonomy level (§9.5) is shorthand for the slot sets defined here.

**Independent** means a person who is not the requester, not the PR author identity, and not the person who pushed any of the reviewed commits.

The **implementer** of a change is the task's implementing agent, or the developer driving a local agent that produced the change (§6.15 rule 1).

1. An agent MUST NOT be an approver.
2. When the autonomy level (§9.5) and this table differ, this table MUST apply.
3. The Factory MUST apply the following table:

| Decision | The requester may supply | Independent person needed |
|---|---|---|
| Merge a Low-tier change on the zero-human path (eligible, area opted in, AI review in `blocking` mode, deterministic gates all green, no open findings, no control file touched) | — (0 approvals) | No |
| Merge a Low-tier change, requester belongs to the team owning every affected area, no breaker tripped and no `blocking` integrity sensor hit | Yes; the sign-off is 1 approval | No |
| Merge a Low-tier change, requester belongs to the owning team but a breaker is tripped or there is a `blocking` integrity sensor hit | Sign-off required but not counted | 1 code owner of each affected area |
| Merge a Low-tier change, requester is outside the owning team of any affected area | Sign-off required but not counted | 1 code owner of each affected area |
| Merge a Medium-tier change | Sign-off required but not counted | 1 |
| Merge a Medium-tier change, requester is outside the owning team of an affected area (§9.6 item 8) | Sign-off required but not counted | 2, with at least one code owner of each area whose owning team the requester does not belong to. At the High tier, the High-tier row already includes this requirement |
| High tier and paths that always require two people | Sign-off required but not counted | 2, with at least one code owner of each affected area |
| Change to a control file (§7.13) | No | 1 code owner of the changed control file |
| Explanation for an integrity sensor (§7.13) | No | 1 |
| `waived` (with expiry) or `deferred` disposition on a `BLOCKER` check (§7.5, §7.13 item 44) | No | 1 independent code owner of the area (security owner for `baselineable: false` checks) |
| `waived` or `deferred` disposition on a `WARNING`/`INFO` check | Yes | No |
| `not-applicable`, `assessed-elsewhere` or `baselined` disposition (§7.5) | — (decided by policy) | No |
| Dismissing an Important-level AI finding | Yes on Medium/High PRs (visible to the independent approver), only if the requester is not the implementer; a developer driving a local agent (§6.15) is the implementer and may not dismiss | At Low tier: someone other than the implementer must dismiss, and the zero-human change moves to the 1-approval path. At Medium/High tier when the requester is the implementer: an independent person |
| Dismissing a security finding | No | Security owner or an independent person; the change is at least Medium while the finding is open (§9.6) |
| Quarantining a flaky check | No | The check's owner |
| Raising a trust level or loosening a circuit breaker | No | The area owner and the Factory owner |
| Change to a control file produced by a registered trusted mechanism (tightening only or identical to the template, §7.13) | — | No; audited |
| Change to a declared sensitive category on a non-PR change channel, for example pricing or legal terms (§12.6) | No | The business owner of that category |
| Activating break-glass (§12.7) | No | Incident commander and an independent code owner of the area |
| Registering or changing a trusted mechanism | No | Code owner of the policy registry and the Factory owner |
| `requirement-review` (§6.12) | Yes, when the tier is Low or Medium and it is not a new user-facing capability | Product/domain owner when it is a new user-facing capability or the tier is High |
| `specification-review`: moving a spec to `ACTIVE` | No | Product/domain owner; contract owner when the spec defines a contract |
| `design-review`: moving an ADR or design to `ACTIVE` | No | Architect or code owner of the area |
| `plan-review` | No | 1 independent code owner (`plan-approval` slot) |
| `release-readiness` | No | Service owner or SRE; security owner where applicable. Medium tier: the person filling the `implementation-review` slot. Mitigation during an open incident: incident commander |
| Moving a knowledge artifact to `SUPERSEDED` or `DEPRECATED` (§5.8) | No | The artifact's authority per §5.10 |
| Decision when the basis changes (§6.5) | `continue` only, when the tier is Low and acceptance criteria and scope are not touched | The authority that approved the affected checkpoint |
| Resolving an incomparable profile conflict (§7.3) | No | The owners of the profiles involved |
| Resolving a knowledge conflict affecting correctness or risk (§4.7) | No | The authority per the table in §4.7 |

Implementation notes:

4. **Do not rely solely on the platform's author rules.** The Factory MUST NOT rely solely on the platform's author rules to prevent self-approval.
5. The Factory MUST enforce the exclusion with a required status check that compares the approver with the requester, the PR author and the person who pushed the reviewed commits.
6. Branch protection settings such as requiring approval of the most recent push MUST be enabled, but MUST NOT be considered sufficient.
7. **Requester identity is authenticated, not self-declared.**
   - The source of truth for the requester MUST be the authenticated identity that triggered the task, recorded by the Factory's task runner from the trigger event: the actor of the CI dispatch, the authenticated author of the issue or command, or the CLI login session.
   - The agent MUST NOT write this value.
   - Commit trailers such as `Requested-by:` are for display only; checks MUST NOT read them.
   - An agent PR with no task runner record MUST be treated as having an unknown requester; an unknown requester is the worst case, so it MUST require independent approval.
8. **Work initiated by automation.** Scheduled or event-driven automations (dependency bots, gardening, codemod rollouts) MUST record the **accountable owner of the automation** as the requester (`source: automation-owner`). Ownership trust MUST be evaluated against that owner.
9. **Approval slots combine by union.**
   - A PR's requirement MUST be the union of the slots by tier, one code owner for each control file touched, one code owner for each affected area where required above, and the security owner where required.
   - One qualified person MAY fill multiple slots if they are qualified for each slot.
   - The number of distinct independent approvers MUST be at least the largest number required by any single rule (for example 2 for High).
10. **Backup owner.**
    - Every area and control file MUST have at least two qualified code owners, or a designated backup.
    - If the only owner is the requester, the backup MUST approve; if there is no backup, the change MUST be escalated to the area's architect or the Factory owner.
    - Ownership with a single point of failure MUST be recorded as a Phase 1 finding.
11. **Small teams.**
    - The number of independent people a rule requires MUST NOT be reduced by team size.
    - A team with few qualified people SHOULD pre-designate fallback reviewers from another team (for example an architect or a code owner of a neighboring area) for the `independent` slots, the second A3 slot and break-glass.
    - Fallback reviewers MUST be recorded in CODEOWNERS or policy, MUST NOT be chosen ad hoc when needed.
    - The pilot (§20.8) SHOULD choose an area with at least three people qualified to review (for example: Appendix H).
12. The Factory MUST enable the mandatory code owner review mechanism on the platform in use (for example "Require review from Code Owners" on GitHub; platform mapping: §14.4) for all governed paths, so that the platform enforces the area owner slots alongside status checks.
13. If the organization has separation-of-duties obligations (for example change management under SOC 2 or SOX), the organization MUST confirm this table with the compliance owner before starting pilot mode (§20.8).

**Rationale.**

- **Do not rely solely on the platform's author rules** (items 4–6): on GitHub, a PR author cannot approve their own PR, but when a bot opens the PR the requester can approve it — exactly the self-approval path that the table above prohibits for Medium and above.
- **Requester identity** (item 7): the value is recorded by the task runner from the trigger event, so an agent cannot self-declare a requester favorable to itself; commit trailers such as `Requested-by:` are easy to forge.
- **Small teams** (item 11): fallback reviewers are pre-designated rather than chosen when needed, so the choice is not influenced by time pressure.

**Example.** An automation owned by the platform team opens a PR in a product team's repository and is not ownership-eligible there, because ownership trust is evaluated against the automation's owner (item 8). The result is that a code owner of the product area fills the A1 slot.
