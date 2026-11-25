# 12. Delivery, release and production operations

## 12.1. Delivery lifecycle

1. The Factory SHOULD apply the following delivery lifecycle. The merge queue (§6.9), immutable artifact with provenance (§12.3) and post-release observation (§12.2) steps are mandatory under their respective sections.

```text
PR approved
 → Merge queue: re-run required checks on latest main
 → Build immutable artifact
 → Publish artifact with provenance
 → Deploy to internal/staging environment
 → Smoke/system checks
 → Internal use first, behind flags
 → Canary, gradual rollout, or release channels (e.g. early → stable)
 → Observe technical and business signals
 → Continue rollout OR pause/rollback/forward-fix
 → Confirm health and close release
```

2. When the release is a task of its own (release train, SDK publish), the Release / Deployment workflow (Appendix D) MUST be the task's primary workflow.
3. When the release is the tail of a change task, the steps of the Release / Deployment workflow MUST be embedded into that task's workflow.
4. The Factory MUST define a release model for each platform.

**Rationale.** Release cadence is constrained by the platform: web can ship continuously, while native mobile depends on the app store review process. As agent throughput increases, the release mechanism can become a bottleneck [R28]. The release model is therefore defined per platform rather than as one common model.

## 12.2. Operational readiness for each change

1. For a meaningful production change, the Factory MUST record an operational readiness record containing:
   - The change owner and the service owner.
   - Affected services/APIs/data entities.
   - Expected success metrics.
   - Error, latency, resource and business thresholds.
   - SLO and error budget policy for the affected service or surface.
   - Alert destination and on-call ownership.
   - Rollout stages and feature flag strategy, if applicable.
   - Rollback, forward-fix or data recovery strategy.
   - Compatibility requirements.
   - Known limitations and follow-up work.
2. When the error budget of the affected service or surface is exhausted, the zero-human path MUST be closed (§9.9).
3. A destructive data-change step MUST have a compensation declared in advance, or MUST be treated as `irreversible` (§6.14).
4. The operational readiness record MUST be the subject of the `release-readiness` checkpoint (§6.12).

**Rationale.** A feature flag is not a universal rollback strategy. A flag by itself cannot reverse destructive data changes; turning off the flag only stops the new code path, while data that has been overwritten or deleted remains in that state. That is the reason for item 3.

### 12.2.1. Change Shepherd

1. For changes going to production, a **Change Shepherd** agent MAY own the operational follow-through of an approved change until the change is rolled out safely and completely.

**Responsibilities.**

2. The Change Shepherd MUST identify the change's flag and rollout plan.
3. The Change Shepherd MUST propose success and failure signals and thresholds from the operational readiness record (§12.2).
4. For Medium and High tiers, a human MUST confirm those signals and thresholds.
5. The Change Shepherd MUST create a change-specific monitoring view from existing telemetry, not a new telemetry platform.
6. The Change Shepherd MUST report status at each rollout stage to the accountable owner and on-call.
7. The Change Shepherd MUST recommend continuing, pausing or rolling back, with evidence.

**Boundaries.**

8. The Change Shepherd MUST use only approved and audited flag and rollout tools.
9. Flag-off and pause MAY be automated within policy for Low and Medium tiers; they map to `PAUSED` in §6.5.
10. Artifact rollback, revert on main and any recovery affecting data MUST be authorized by a human (rationale: §18.1).
11. The Factory SHOULD prefer fast detection and one-step rollback over gates that slow down every change (P22).

**Rationale.** The instruction for the Change Shepherd in the cited practice is "shepherd this change until it is rolled out safely and completely" [R28]. Flag-off and pause may be automated for Low and Medium tiers because they are reversible, do not change the deployed artifact or data, and are audited. In contrast, artifact rollback, revert on main and recovery affecting data all change state in ways that are harder to reverse, so they require human authorization. The preference for fast detection and one-step rollback comes from principle P22 and reported practice [R29].

## 12.3. Deployment and provenance

1. The Factory MUST record:
   - Repository and commit SHA.
   - Build image/runner/toolchain.
   - Dependency lockfile/SBOM reference.
   - Artifact digest/checksum.
   - Agent/model/skill/playbook version when agent-generated.
   - Reference to the verification bundle.
   - Approval events.
   - Target deployment environment and time.
   - Release health and rollback events.

## 12.4. Incident feedback

**Incident review.**

1. Every significant incident or revert MUST be examined for root causes in:
   - Requirements/specification.
   - Implementation.
   - Architecture or dependency boundaries.
   - Contract/consumer compatibility.
   - Oracle or verification coverage.
   - Agent context, skills, tools or permissions.
   - Release, monitoring or recovery strategy.
2. Incidents MUST be handled under the Incident Response workflow (Appendix D).
3. When an incident points to a wrong spec, design, runbook or document, it MUST trigger reconciliation (§5.10).
4. The output of an incident review MUST NOT default to a new checklist.
5. An incident review MUST choose a preventive mechanism that addresses the actual root cause: code/API design, rule, regression test, contract, skill, documentation, eval case, runbook or operational control.
6. Every incident review MUST answer **which layer should have caught this defect** (guide, sensor or human gate; §3.5) and **what the agent was missing up front** (missing context, check or instruction).
7. The fix MUST be routed to the layer identified in item 6.
8. Incidents and reverts MUST be added to the reviewer and agent eval sets (§7.12, §9.2).

**Closed loop from production signals.**

9. Production signals (error tracking, failed deploys, uptime, user feedback) MAY automatically initiate a Bug Fix task, subject to the conditions in items 10–15.
10. The **requester** MUST be the accountable owner of the automation that ingests the signal (`automation-owner`, §9.10).
11. **Deduplicate before creating a task.** Signals MUST be grouped by defect fingerprint and attached to an open task or incident if one exists; one defect MUST NOT spawn multiple tasks.
12. **Thresholds and limits.** A signal MUST create a task only when it exceeds the declared frequency or impact threshold, within the area's attention budget (§13.6).
13. A signal indicating an ongoing incident MUST enter the Incident Response workflow, not Bug Fix.
14. **Reproduction is mandatory.** A task MUST proceed only when the defect is reproduced (Appendix C).
15. A signal that cannot be reproduced MUST become a finding for the owner and MUST NOT become a speculative fix.

**Lookback.**

16. In areas that allow production signals to create tasks automatically (items 10–15), the Factory MUST run the *Lookback* workflow (Appendix D) periodically; in other areas, the Factory SHOULD run it. Lookback runs over released fixes within a window (*suggested* 14–30 days) to check whether the original defect recurs, based on the signal fingerprint and the reproduction test.
17. A recurring defect MUST reopen root cause analysis, MUST be counted as an escaped defect for the corresponding task class (§9.4, §9.9) and MUST be added to the eval set (§9.2).

**Rationale.** The preventive mechanism is chosen according to the actual root cause and placed in the layer that should have caught the defect, instead of adding a checklist by default. The two questions in item 6 come from reported practice [R29].

Defects do not always arrive via human-written issues. Error tracking, failed deploys, uptime and user feedback are signal sources that can automatically initiate Bug Fix tasks [R39]. The conditions in items 10–15 prevent familiar failure modes of this automation: one defect spawning dozens of duplicate tasks, automated tasks consuming the area's entire review capacity, an ongoing incident being handled as an ordinary bug, and an agent writing a speculative fix for a defect that has not been reproduced.

Lookback closes the feedback loop after release: a merged fix has not necessarily cured the original defect. Recurrence results feed into trust (§9.4, §9.9) and eval (§9.2), so Lookback is also an early indicator for trust evaluation.

## 12.5. Production agents

1. Production agents MUST operate within the scope and autonomy boundaries of the following table.

| Agent | Scope | Autonomy boundary |
|---|---|---|
| Performance incident triage | Triage alerts, deduplicate, confirm real regressions, localize likely root cause, propose a fix | Propose only; the fix goes through the normal task flow |
| Incident assistant | Gather context, correlate signals, answer questions, propose mitigations | Propose mitigations; execute only on explicit engineer instruction |
| Change Shepherd | Per-change rollout monitoring (§12.2.1) | Within approved rollout tools and policy |

2. Humans MUST remain on call.
3. Automated handling of routine incidents MUST have its own eval, circuit breaker and approval.

**Rationale.** The scope and autonomy boundaries of the performance incident triage agent and the incident assistant are based on reported practice [R28]. Automated handling of routine incidents is a later goal, not current scope.

## 12.6. Change channels outside pull requests

1. The Factory MUST apply the following change channel table.

| Channel | Example | Evidence subject |
|---|---|---|
| `code` | Pull request, including infrastructure as code | Commit SHA |
| `content` | CMS publish, scheduled publish, documentation content | Content revision ID |
| `config` | Feature flag targeting, runtime configuration, remote configuration | Change ID in the flag or configuration system |
| `data` | Data fixes and backfills run outside deploys | Run ID of the approved script |

2. Every channel in use MUST define:
   1. **Risk classification** from declared inputs (content type, flag ownership, data class per §10.6); risk classification MUST NOT rely on free-text matching. Declared sensitive types (pricing, legal terms, security settings) MUST have their own approval slot (§9.10).
   2. **Pre-change verification**, producing a `VerificationRun` whose subject is the channel revision, at the `publish` gate, within the declared time budget.
   3. **Approval** per §9.5 and §9.10. An agent acting on a channel MUST be subject to the same autonomy policy as on code.
   4. **Mitigation actions**, including unpublish, flag-off, rollback to a previously verified revision, and content takedown for legal or incident reasons. These actions MUST NOT be blocked; they proceed directly and MUST be audited, if and only if:
      - the subject is declared in advance as having a safe state (`safe-off`): a flag whose disabling does not weaken a security or legal control (for example, disabling a flag that enforces MFA is not `safe-off`), or a verified revision within the freshness window declared by the channel;
      - the action is tied to an open incident or alert;
      - the actor is an authenticated human with operational permission on that subject, or the Change Shepherd within policy (§12.2.1);
      - the action is reviewed weekly.

      Actions that do not satisfy these conditions MUST follow the channel's normal path or break-glass (§12.7).
   5. **Hotfix path, if any.** A channel MAY define a hotfix path (publish first, verify afterward). If defined, that path MUST apply only to changes with HIGH reversibility, MUST verify immediately after publishing, MUST roll back automatically on `FAIL`, and MUST alert the owner.
   6. **Evidence and provenance**, retained like code evidence and linked to the work item.
3. Non-PR channels MUST use a shortened lifecycle: `REQUESTED → VERIFYING → AWAITING_APPROVAL` (only when approval is required) `→ PUBLISHED → OBSERVING → COMPLETED`, with `ROLLED_BACK` reachable from any state after `PUBLISHED`, and `BLOCKED` when verification fails.

**Rationale.** Not every production change is a code change. Content published from a CMS, feature flag targeting, runtime configuration and operational data fixes all change what users see without going through a pull request. A Factory that governs only PRs leaves these paths unverified.

Mitigation actions (unpublish, flag-off, rollback to a verified revision) proceed directly because blocking them during an incident prolongs the damage. The `safe-off` condition prevents abuse of this path: disabling a flag that enforces MFA looks like a mitigation action but in fact weakens a security control.

## 12.7. Factory degraded mode and break-glass

1. A failure of the Factory itself MUST NOT halt emergency fixes.
2. A failure of the Factory itself MUST NOT become a way to bypass gates without audit (P26).
3. Every Factory component (policy decision, separation of duties check, verifier pipeline, attestation, evidence store, AI reviewer, merge queue) MUST have an availability SLO, an owner and a declared degraded behavior.
4. The Factory MUST apply the following degraded behaviors.

| Unavailable component | Degraded behavior |
|---|---|
| AI reviewer | The zero-human path is closed (A0 becomes A1). Medium and High changes wait in the queue; review for Medium and High is never downgraded because of cost or incidents (§7.9) |
| Deterministic CI or verifier pipeline | No merge on the normal path |
| Attestation or evidence store | No merge on the normal path. Unattested results are retained and attested after recovery |
| Policy decision or separation of duties check | No merge on the normal path |
| Merge queue | Merges must wait; no direct push to main |

**Break-glass** is the only exception, and it exists to restore service.

5. Break-glass MUST be used only to fix or mitigate a production incident, a security vulnerability, or a legally mandated change.
6. Break-glass MUST NOT be used for feature development, for control files, for the policy repository, or for any loosening change.
7. Break-glass MUST be activated by two people: the incident commander and an independent code owner of the affected area (§9.10).
8. Break-glass MUST be time-limited (*suggested* ≤ 24 hours) and MUST be logged with reason and scope.
9. During break-glass, every check that can still run MUST still run.
10. Skipped checks MUST be recorded as `NOT_ASSESSED` with disposition `deferred` to a specifically named post-hoc gate.
11. After recovery, the change MUST be fully verified and attested within a declared period (*suggested* 24 hours).
12. If post-hoc verification fails, the Factory MUST open an incident leading to a revert or forward fix.
13. Use of break-glass MUST close the zero-human path for that area until post-hoc verification is complete (§9.9).
14. Each use of break-glass MUST be reviewed in the weekly cadence and MUST be reported as a metric (§13.2).

**Rationale.** The Factory sits on the critical path of delivery. When a Factory component fails, there are two failure modes to avoid: emergency fixes getting stuck because the Factory is unavailable, and a Factory incident becoming an excuse to skip gates without leaving a trace. Principle P26 addresses both: degrade toward more human involvement, never toward fewer checks. Break-glass is the controlled escape hatch for restoring service.
