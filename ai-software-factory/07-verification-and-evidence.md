# 7. Verification architecture & evidence

## 7.1. Core principles

1. The Factory SHOULD give the implementing agent fast feedback.
2. Critical acceptance criteria SHOULD be verified by independent tests, a separate grader, consumer tests or runtime observation where appropriate.
3. Tests written by the same agent that implemented the feature MUST NOT be counted as an independent oracle for a `critical` acceptance criterion (§6.4.1, §7.7).
4. The verifier SHOULD derive expected behavior from approved requirements, contracts, invariants or independently reviewed scenarios.
5. The Factory SHOULD give the agent something runnable (a test, a build, a running app, a screenshot to compare against the design) and require the agent to **present the output**, rather than declare that the work is done.

**Rationale.** Verification is a first-class capability of the Factory, not merely a CI stage and not a final checkbox. The implementing agent needs fast feedback so the fix loop stays short; critical acceptance criteria need a verification source that is independent of that agent. Tests written by the implementing agent can be useful, but if the agent misreads the requirement, its tests usually misread it in the same way.

Practitioners describe giving the agent something runnable and demanding it present the output as perhaps the most important practice for the quality of agent output, with a self-reported estimate of 2–3× better results [R29].

## 7.2. Verification levels

1. The Factory MUST use the verification levels in the following table.

| Level | Purpose |
|---|---|
| V0 – Formatting | Consistent formatting |
| V1 – Static / Lint | Known patterns and code defects |
| V2 – Type / Schema | Structural correctness |
| V3 – Architecture | Boundary and dependency constraints |
| V4 – Unit / Component | Local behavior |
| V5 – Integration / Contract | Interactions between components |
| V6 – System / E2E | Full user/system behavior |
| V7 – Runtime / Operations | Real runtime properties |
| V8 – Security / Supply chain | Security and dependency risk |
| V9 – Performance / Accessibility | Quality for users and the system |
| V10 – Formal / Property-based | Invariants and wide input/state spaces |
| V11 – Human evaluation | Product intent, trade-offs, usability, architectural decisions |

2. The Factory SHOULD use the cheapest verifier that still provides sufficient confidence.
3. Higher levels SHOULD be added based on criticality and risk.
4. Higher levels SHOULD NOT be applied automatically to every change.

**Rationale.** Higher levels cost more in time and infrastructure, so they are added according to criticality and risk rather than applied to every change.

**Example.** Tools for each level:

| Level | Example |
|---|---|
| V0 – Formatting | Prettier, gofmt, black |
| V1 – Static / Lint | ESLint, Ruff, compiler diagnostics |
| V2 – Type / Schema | TypeScript, schema validation, OpenAPI lint |
| V3 – Architecture | dependency-cruiser, ArchUnit, custom graph tests |
| V4 – Unit / Component | Vitest, Jest, pytest, xUnit |
| V5 – Integration / Contract | Testcontainers, Pact, provider tests |
| V6 – System / E2E | Playwright, API scenarios, service orchestration |
| V7 – Runtime / Operations | logs, traces, metrics, failure injection, resource checks |
| V8 – Security / Supply chain | secret scanning, SAST, dependency vulnerabilities, SBOM |
| V9 – Performance / Accessibility | load tests, browser metrics, axe, bundle budget |
| V10 – Formal / Property-based | property-based tests, fuzzing, TLA+, Lean where there is a compelling reason |
| V11 – Human evaluation | code owner, product owner, UX/security review |

## 7.3. Verification profiles and how they combine

1. A shared verification platform MUST support multiple profiles.

A **surface** is a declared group of paths, routes or modules that share the same kind and the same criticality.

**Composition model.**

2. Verification policy MUST apply to a surface, not to the whole repository.
3. The Factory MUST determine the mandatory checks of a surface from the following composable dimensions:
   - a **baseline** that applies to every surface;
   - one or more **kinds** (an archetype such as `operational` or `public-web`, or a service type such as `event-consumer`);
   - **capability packs** for functionality that recurs across products (attachments, import/export, payments);
   - **cross-cutting profiles** (accessibility, security-sensitive, internationalization);
   - an optional **product profile**.
4. A product profile MUST only add or tighten.

**Resolution rules.**

5. The affected surfaces MUST consist of the surfaces changed directly, plus the surfaces affected through the dependency graph, plus a declared **sentinel set** whenever a shared asset changes.
6. Merge by taking the stricter choice:
   - For each check, the merged value MUST take the stricter choice: `requirement` takes the highest level, `when` takes OR, `gate` takes the earlier gate, ordered parameters take the stricter value, and set-valued parameters take the union.
   - A conflict that cannot be merged MUST cause the resolution to fail and MUST NOT be resolved implicitly.
7. Check lifecycle:
   - A check in lifecycle `shadow` MUST run with enforcement `observe`.
   - `planned` and `deprecated` checks MUST NOT run (§8.7).
8. Disposition of mandatory checks:
   - A mandatory check that does not pass MUST be satisfied only by a disposition counted as a pass under §7.5 (`not-applicable`, `assessed-elsewhere`, `baselined`, `waived`, `deferred`), each under its own conditions.
   - For a `FAIL` result, `waived` MUST be the only way, decided by a human.
9. The resolved plan and the version of every profile used MUST be recorded in the evidence (Appendix K).

**Profile layers.**

10. The Factory MUST resolve the check plan for a change from the following layers, from most general to most specific:

| # | Layer | Source |
|---|---|---|
| 1 | `core` | This Standard |
| 2 | `domain` | Domain profile (§20) |
| 3 | `organization` | Product or organization profile |
| 4 | `repository` | The repository's verification profile |
| 5 | `surface` | The per-surface composition above (baseline, kind, capability pack, cross-cutting) |
| 6 | `task-class` | The workflow spec of the task type (§6.7) |
| 7 | `risk` | Tier and reversibility (§9.5–9.6) |
| 8 | `task` | Task contract |

11. The `task` layer (the task contract) MUST only add checks or lower budgets.
12. All layers MUST be merged by the merge rule in item 6.
13. A layer MUST NOT loosen another layer (PC-3, §20.2).

**Resolving conflicts between profiles.**

14. The Factory MUST NOT silently choose a value when two profiles conflict.
15. The Factory MUST resolve conflicts according to the following table.

| Conflict type | Example | Resolution |
|---|---|---|
| Clearly ordered, stricter | Coverage thresholds of 70% and 80% | Automatically take the stricter value; record it in the trace |
| Set | Two lists of browsers to support | Take the union; record it in the trace |
| Incomparable | Two profiles pin two different runtime versions; the same check with two mutually exclusive parameters; two gates that cannot both be satisfied | Resolution fails. The task moves to `BLOCKED` with a conflict record; the owners of the profiles involved decide; the decision is recorded as a versioned resolution entry in the more specific profile, or as an ADR when it sets a precedent |
| One layer tries to loosen another | A task contract lowers the `requirement` of a check | Rejected; never merged |

16. Evidence MUST record the **resolution trace** (`resolution`, Appendix K): the layers and their versions, the conflicts, and by which rule or which decision each conflict was resolved.

**Rationale.** A single set of checks does not fit every kind of software: a frontend needs browser E2E and accessibility, an SDK needs install-from-artifact tests, an AI application needs a golden evaluation dataset. Policy applies per surface rather than to the whole repository because one repository can contain both a `public-web` page and an internal `operational` tool with different criticality.

The "take the stricter choice" merge rule makes combining multiple dimensions deterministic and never accidentally loosens. When two values are incomparable, the resolution fails instead of guessing, so the conflict is surfaced for a human to decide. The resolution trace in the evidence tells the reviewer why a check is present with specific parameters, and which decision resolved the conflict.

**Example.** Each profile layer:

| # | Layer | Example |
|---|---|---|
| 1 | `core` | Core integrity sensors (§7.13) |
| 2 | `domain` | Frontend Profile |
| 3 | `organization` | A mandatory check for every payment product |
| 4 | `repository` | `backend-service-v1` |
| 5 | `surface` | A `public-web` surface plus accessibility |
| 6 | `task-class` | Database Migration requires a dry run on a production-like snapshot |
| 7 | `risk` | Tier High requires pre-merge mutation testing and Deep review |
| 8 | `task` | May only add checks or lower budgets |

Profiles A–E below are illustrative summaries. The detailed requirements for a kind of software live in its domain profile; for frontend that is the AISF Frontend Profile 1.1.

### Profile A – Frontend / Web application

**Typical technology:** React, Next.js, Vue, Angular, web applications.

Mandatory or conditional checks:

- Install/build reproducibility and lockfile checks.
- Lint, formatting, type check.
- Module boundary and dependency cycle checks.
- Unit tests and component tests.
- API contract/client integration tests.
- Browser E2E for critical user journeys.
- Runtime verification on the running application.
- Accessibility baseline and keyboard navigation for relevant controls.
- Visual regression/screenshots for changes with meaningful visual impact.
- Performance and bundle budgets for relevant areas.

**Runtime feature map:** a list of critical routes/screens, user actions, expected states, data dependencies and the verification scenarios available. This map is a navigation index for the agent and does not replace tests.

Evidence for a user journey:

```text
Open pipeline page
 → Create pipeline
 → Add repository and build configuration
 → Save
 → Verify success state
 → Reload page
 → Confirm persisted pipeline data
 → Capture screenshot/video and test report
```

### Profile B – Backend / API / Service

**Typical technology:** REST, GraphQL, RPC, event-driven services, microservices.

Mandatory or conditional checks:

- Compile, lint, type/schema checks.
- Unit tests for domain logic.
- Database tests and integration tests.
- OpenAPI/AsyncAPI schema checks.
- Consumer-driven contract tests when there are multiple consumers.
- Backward compatibility and migration checks.
- Authentication/authorization and data access tests.
- Tests for timeouts, retries, idempotency and failure modes.
- Observability checks: structured logs, traces and relevant metrics.
- Load/performance and resilience tests for critical paths.

For cross-service changes, consumer/provider verification is preferred over relying only on a full end-to-end environment. Pact is a proven option for consumer-driven contracts [R15].

### Profile C – SDK / Library / CLI

**Typical technology:** TypeScript/Java/Python SDKs, client libraries, command-line tools.

Mandatory or conditional checks:

- Checks on the public API surface and exported types.
- Consistency between source and generated declarations.
- A matrix of supported languages/runtimes/platforms.
- Package build tests and installation from the generated artifact, not only from the source workspace.
- Consumer-side integration tests against supported service/API versions.
- SemVer and breaking-change detection.
- Examples and quickstarts that compile/run in CI.
- Checks on generated documentation and migration guides.
- Package signing/provenance and release metadata where applicable.

The key question is not only whether the SDK repository's own tests pass, but whether a real consumer can install and use the published package as the documentation describes.

### Profile D – AI Application / Chat / Agent

**Typical technology:** LLM applications, RAG, tool-using agents, AI chat, model-based workflows.

Mandatory or conditional checks:

- Golden evaluation datasets for critical use cases.
- Task completion rate and workflow correctness.
- Groundedness/retrieval relevance where applicable.
- Tool selection, tool parameters and handling of tool results.
- Refusal/policy and safety behavior.
- Prompt/model regression evaluation.
- Latency, time-to-first-token, token usage and cost.
- Conversation scenarios and state management.
- Human review for subjective or high-impact outcomes.
- Trace review and evaluation of handoffs between model/tool/guardrail.

Do not rely solely on exact string matching for non-deterministic model output. Use rubric graders, deterministic checks for structured output, reference answers where appropriate, and human calibration for subjective dimensions. OpenAI's agent evaluation guidance emphasizes traces, graders, datasets and eval runs [R11].

### Profile E – Infrastructure / CI-CD / Platform Automation

For repositories that define deployments, pipelines, infrastructure or platform automation:

- Static configuration checks and policy-as-code.
- Plan/diff review before apply.
- Environment isolation and least privilege.
- Secret and identity checks.
- Integration tests in an ephemeral environment.
- Post-deploy smoke tests.
- Rollback/recovery drills.
- Change approval for production and data plane actions.

## 7.4. Verification stages

1. The Factory MUST apply the verification stages and blocking policies in the following table.

| Stage | Runs when | Typical checks | Blocking policy |
|---|---|---|---|
| Local / agent loop | During implementation | Lint, type, fast unit, focused architecture, task-specific tests | Blocks the agent from declaring completion if a mandatory check fails |
| Pre-PR | Before opening/updating a PR | The full mandatory profile, independent tests, evidence manifest | Mandatory checks MUST pass or be explicitly marked `NOT_ASSESSED` with an approved disposition |
| PR merge gate | On the PR and each relevant commit | Required CI, security, contract, risk checks, approvals | Enforce repository protection mechanisms and risk policy |
| Merge queue | After approval, before landing | The mandatory **deterministic** checks rerun on the latest main with the change applied. AI review is not rerun; its result is tied to the reviewed SHA | Blocks the merge on change-induced failures; the task returns to `REPAIRING` |
| Deep / scheduled | Nightly, weekly or release candidate | Full E2E, fuzz/property/mutation, dependency audit, documentation drift, agent regression evals, AI review on existing code | Creates findings (findings on existing code are tagged Pre-existing) and blocks a release only per defined thresholds |
| Post-release | After deploy | Smoke tests, canary metrics, business signals, error budget | Pause/rollback when predefined thresholds are exceeded |

2. At the Pre-PR stage, the `knowledge.impact` check (§5.9) MUST run together with the mandatory profile.

A **gate** is the point at which a mandatory check blocks: `merge` (before a code change lands), `publish` (before a non-PR change takes effect, §12.6), or `post-release` (after deploy, through pause and rollback thresholds).

3. Every mandatory check MUST block at a gate.
4. A check that cannot run before its gate MUST have status `NOT_ASSESSED`.
5. A check that is `NOT_ASSESSED` because it cannot run before its gate MUST have a disposition.

**Rationale.** Each stage balances feedback speed against depth: the local loop must be fast, while deep checks (fuzz, mutation, dependency audit) run on a schedule so they do not slow every PR. AI review is not rerun in the merge queue because it is non-deterministic, expensive and would make the queue flaky.

The `post-release` gate is for checks that can only be evaluated after deploy, for example canary metrics; before that point the check is `NOT_ASSESSED` with a disposition such as `deferred`.

## 7.5. Evidence contract

1. Every verification result MUST conform to the canonical `VerificationRun` / `CheckResult` schema (Appendix K) and contain at least:
   - The work item (task) ID and the change channel (§12.6).
   - The subject: repository, commit SHA and base SHA; for non-PR channels, the revision ID.
   - The resolved profiles and their versions; the check ID and check version; the rules the check implements.
   - Status, enforcement, severity and gate.
   - For every non-`PASS` result: a reason and, when the result needs to count as satisfied, a disposition.
   - The command or procedure, the tool version, the runner or environment.
   - Time, duration and number of attempts.
   - Artifact URIs and digests.
   - The identity of the producer (human, agent, CI job) and, from L3 (§20.4), an attestation (§7.15).
   - The ID of the related requirement or acceptance criterion.

**Status (closed set).**

2. The status of a verification result MUST be one of the following values.

| Status | Meaning |
|---|---|
| `PASS` | The check ran and met its criteria |
| `FAIL` | The check ran and did not meet its criteria |
| `ERROR` | An infrastructure or tool error. It says nothing about the change |
| `NOT_ASSESSED` | The check did not run, or the harness could not observe the evidence layer the check requires |

3. `ERROR` MUST NOT be counted as a pass.
4. `ERROR` MUST NOT consume loop budget (§6.4).
5. On `ERROR`, the Factory MUST rerun, then escalate to the check owner.
6. `NOT_ASSESSED` MUST NOT be converted to `PASS`.

**Disposition (closed set).**

7. A non-`PASS` result MUST be counted as satisfied only when it has a valid disposition according to the following table.

| Disposition | Applies to | Who decides | Conditions |
|---|---|---|---|
| `not-applicable` | `NOT_ASSESSED` | Policy (`decidedBy: policy`, with a rule reference) | The profile declares the check's applicability conditions, and those conditions are not met. No human needed |
| `assessed-elsewhere` | `NOT_ASSESSED` | Policy | The behavior is verified in another pipeline. That run has an attestation; the version it verified is the version currently deployed, or about to be deployed, to the target environment; and the run is within the freshness window declared by the profile |
| `baselined` | `FAIL` | Policy | Every finding matches a baseline entry that CI snapshotted when the check was activated (§8.7). A baseline entry that no longer fails is stale and MUST block until a trusted mechanism removes it (§7.13) |
| `waived` | `FAIL`, `NOT_ASSESSED` | Human, per §9.10 | Records the owner, reason, scope, expiry and approver. For a `BLOCKER` check, the approver MUST be an independent code owner of the area (§9.10). A time-limited waiver is not a loosening of a control file (§7.13 item 44). For a check marked `baselineable: false`, only the security owner MAY waive, and for no longer than the profile allows |
| `deferred` | `NOT_ASSESSED` | Human, per §9.10 | States the later gate at which the check MUST be evaluated (for example `post-release`) |

8. A mandatory, applicable check that has neither `PASS` nor a valid disposition MUST block at its gate.
9. A profile MUST NOT add a status or disposition that counts as a pass.
10. A profile MAY add display labels, provided each label maps one-to-one to a status and disposition.

The canonical format of evidence is in Appendix K; an illustrative example is at the end of this section.

**Canonical enforcement and severity.**

11. Every producer MUST map its results to a single vocabulary of enforcement and severity according to the following table.

| Producer | Enforcement | Canonical severity | Effect on merge |
|---|---|---|---|
| Rule `BLOCKING` / check `active` + `required` / linter `error` | `blocking` | `BLOCKER` | Blocks at its gate |
| Rule `WARN` / check `active` + `advisory` / linter `warning` | `warn` | `WARNING` | Displayed; does not block |
| Rule `OBSERVE` / check `shadow` / informational output | `observe` | `INFO` | Recorded only |
| AI review `Important` finding (verified) | Per the area's AI review mode (§7.9): `blocking` yields `BLOCKER` while open; `advisory` yields `WARNING`; a finding in `shadow` is not posted | — | Blocks only in `blocking` mode |
| AI review `Nit` finding | `observe` | `INFO` | Never blocks; capped in number per review pass |
| AI review `Pre-existing` finding | `observe` on the PR | `INFO` | Creates a gardening finding; does not block the PR |
| Finding from security review | `blocking` | `BLOCKER` while open | Blocks until fixed or dismissed per §9.10 |

12. The severity of findings *inside* a check (for example `critical/serious/moderate/minor` from an accessibility engine) MUST be defined by the profile.
13. The severity of a finding inside a check MUST only determine whether the check's result is `FAIL`.

**ID conventions.**

14. A **rule** MUST use `<SCOPE>-<CATEGORY>-<NNN>`, where `CATEGORY` comes from §8.1 or from a category the profile has registered.
15. The scope prefix MUST be omitted for organization-wide rules (`ARCH-001`) and MUST be used for profile-specific rules (`FE-ARCH-001`, `BE-PERF-001`).
16. A **check** MUST use `<domain>.<name>` (`integrity.skip-focus`, `api.provider-contract`) and MUST list the rules it implements in `implementsRules`.
17. A rule MAY be implemented by multiple checks, and a check MAY implement multiple rules.
18. A **core requirement** MUST use `CORE-<AREA>-<NN>` (§20.6).

**Rationale.** Status, disposition, enforcement and severity are closed sets so that every producer maps to the same set of values, which means the evidence aggregator never has to guess. `ERROR` says nothing about the change; if it consumed loop budget (§6.4), the agent would have to "fix" a broken runner.

Examples and rationale for dispositions decided by policy:

- `not-applicable`: for example "the performance test applies only when a file under `src/hot-path/**` changes". No human needed, so conditional checks do not cause approval fatigue.
- `assessed-elsewhere`: the typical case is a provider contract test that runs in the provider's pipeline rather than the consumer's pipeline.
- `baselined`: allows activating a new check on code that already has violations without blocking every PR; a stale baseline entry blocks until removed, so the baseline can only shrink.

**Example.** An evidence record (illustrative; the mandatory schema is in Appendix K):

```json
{
  "schemaVersion": "aisf/4.0",
  "runId": "run-9182",
  "workItemId": "TASK-2026-00124",
  "channel": "code",
  "subject": { "repo": "pipeline-service", "commitSha": "abc1234", "baseSha": "9f0e1d2" },
  "resolvedProfiles": [{ "id": "backend-service", "version": "1.0.0" }],
  "checks": [
    {
      "checkId": "api.provider-contract",
      "checkVersion": "1.2.0",
      "implementsRules": ["BE-API-001"],
      "requiredBy": ["backend-service"], "requirement": "required",
      "enforcement": "blocking", "severity": "BLOCKER", "gate": "merge",
      "status": "PASS",
      "requirementIds": ["AC-PIPE-031"],
      "command": "pact-provider-verifier --provider pipeline-service", "durationMs": 48210,
      "attempts": 1,
      "tool": { "name": "pact-provider-verifier", "version": "1.2.0" },
      "evidence": [{ "kind": "report", "uri": "ci://runs/9182/pact-report.json", "digest": "sha256:…", "retentionClass": "standard" }]
    },
    {
      "checkId": "perf.service-benchmark",
      "checkVersion": "0.4.0",
      "implementsRules": ["BE-PERF-001"],
      "requiredBy": ["backend-service"], "requirement": "advisory",
      "enforcement": "warn", "severity": "WARNING", "gate": "merge",
      "status": "NOT_ASSESSED",
      "reason": "No file under src/hot-path/** changed",
      "disposition": {
        "type": "not-applicable",
        "decidedBy": "policy",
        "decidedAt": "2026-11-20T09:55:00Z",
        "policyRef": "backend-service#perf-applicability"
      },
      "command": "applicability check (policy)", "durationMs": 0,
      "attempts": 0,
      "evidence": []
    }
  ],
  "provenance": { "factoryVersion": "4.0.0", "verifierRef": "policy@4f2a", "producer": { "ciJob": "verify/9182" }, "startedAt": "2026-11-20T09:50:00Z", "finishedAt": "2026-11-20T10:00:00Z" }
}
```

The performance check did not run because no file under `src/hot-path/**` changed; the `not-applicable` disposition is decided by policy, so the check still counts as satisfied without a human.

## 7.6. Evidence bundle and PR format

1. A pull request SHOULD be an evidence-based change package, consisting of:
   - Intent and expected outcome.
   - Spec and requirement IDs.
   - The affected repositories/domains/features/services/contracts/dependencies.
   - Architecture and migration impact.
   - Risk classification and rationale.
   - Verification profile and results.
   - Evidence links: reports, screenshots, recordings, traces, test output.
   - AI review findings and dispositions.
   - The review checkpoints passed and their approvals (§6.12).
   - Knowledge impact: affected artifacts and reconciliation decisions (§5.9–5.10).
   - Notes on rollout, monitoring and recovery.
   - The owner, the required approvers and known limitations.
2. Every PR template MUST contain the following three sections, even for small changes: **Testing** (what was run and what it returned), **Risk** (the tier and rationale), and **How to revert**. See Appendix G.
3. An agent MAY draft the Testing, Risk and How to revert sections.
4. The accountable requester MUST sign off the Testing, Risk and How to revert sections.

**Reviewer reading order.**

5. The evidence bundle SHOULD reduce review load, not increase it.
6. The PR description SHOULD be arranged for human readers in the following order:
   1. The one-line intent and the risk tier, with the tier's rationale.
   2. **Decisions needing human judgment** (open questions, trade-offs, deviations from the spec) and what the automated checks will *not* detect.
   3. Open findings and their dispositions.
   4. A diff summary by area.
   5. Evidence: one line per passing check; failures and `NOT_ASSESSED` items presented in detail.
7. A reviewer who cannot say what the checks would miss SHOULD request a genuine review pass instead of a quick approval.
8. The Factory SHOULD limit the number of open agent PRs awaiting any one reviewer (a review WIP limit); this limit is the human attention budget in §13.6.

**Rationale.** The three sections Testing, Risk and How to revert come from the cited practice [R29]: they are the minimum for the approver to understand how the change was verified, how risky it is and how to roll it back.

The reading order puts what requires human judgment first and condenses what machines have already confirmed; a reviewer who cannot say what the checks miss has probably not really reviewed [R29]. The review WIP limit exists so that the increase in agent throughput does not turn into a review queue.

When constraints are moved into the codebase and checks, human review shifts from reading diffs to reading **evidence**: recordings, screenshots, state diagrams and verification results [R05].

## 7.7. Verification independence

For critical acceptance criteria:

1. Scenario tests MUST be derived from approved requirements or contracts, not only from implementation details.
2. The implementation context and the verification context SHOULD be separated where feasible.
3. Checks MUST be rerun from a clean workspace or CI runner.
4. The Factory SHOULD use a different verifier or an independent test oracle for critical behavior when practice allows.
5. AI reviewer findings MUST be treated as advisory until precision and severity calibration are established.
6. A reviewer MUST NOT approve a change merely because another agent says it is correct.

## 7.8. Formal and property-based verification

1. Formal verification SHOULD NOT be applied by default to every feature.
2. The Factory SHOULD choose verification techniques (property-based testing, fuzzing, model checking, formal proof) based on the failure model, and apply them only where the failure model justifies the cost.

**Rationale.** Formal verification is powerful but is not a default tax on every feature. A screenshot proves one observed state; a statechart, property or model can detect missing state transitions.

Property-based testing/fuzzing fits when the input space is large, parsers/serializers are involved, or invariants can be stated as properties. Model checking or formal proof fits when state transitions are complex and the consequences of a defect justify the cost, for example distributed orchestration, money transfers, concurrency primitives, or safety-related invariants.

**Example.** From practitioners (self-reported) [R05, R06]:

- **Model checking for user flows**, for example TLA+ for notification behavior across multiple devices, together with statechart images from the user's perspective so that humans can review the model.
- **Proofs for algorithms** where a visible defect is costly, for example Lean [R34] for a list virtualization algorithm that must not flicker or stutter.
- **Property-based UI testing** that generates sequences of user interactions and checks invariants.

These are additional layers, not silver bullets.

## 7.9. AI code review pipeline

1. AI review SHOULD run continuously during the work, rather than being only a final gate.
2. Most changes SHOULD be reviewed and fixed by an agent at least once before a human looks.
3. AI review MUST tag each finding as `Important`, `Nit` or `Pre-existing`.

**Design rules.**

4. The reviewer MUST read the diff in the context of the whole repository and the stated intent of the change (Change Brief / spec), and assess whether the change solves the problem, fits the system and avoids breaking anything else.
5. For a bug fix, the reviewer MUST check that the root cause has been addressed.
6. A finding about a bug that the change did not cause MUST be labeled **Pre-existing**, MUST NOT be attributed to the author, and MUST become a gardening finding.
7. **Every posted finding MUST be answered**: fixed, or the thread resolved with a reason by someone with authority to dismiss it (§9.10).
8. A finding MUST be cleared automatically when a later commit fixes it.
9. **Security review MUST be a separate pass** (§10.10).
10. An open security finding MUST block merge. An `Important` bug finding from AI review blocks merge according to the area's AI review mode (§7.5): it MUST block when the area is in `blocking`.
11. **Review depth MUST be configurable by risk**: Quick, Standard or Deep.
12. The Factory SHOULD use Deep where a missed defect costs more than the extra review time.
13. AI review MUST only **post findings** and MUST NOT approve.
14. AI review MUST NOT fill a mandatory human approval slot (P20).
15. Agent review MUST run on a schedule (*suggested* nightly, §7.4) over existing code, and the results MUST be fed into codebase gardening (§8.5).
16. **Prefer findings proven by execution.** When practice allows, a finding SHOULD be accompanied by a run that proves it (a failing test, a reproduction of the bug, a trace).
17. When independence matters, the Factory SHOULD consider using a different model or provider for review than for implementation.
18. **Add-only authority.** AI review MAY block or escalate.
19. AI review MUST NOT be the sole evidence that permits a merge.
20. The zero-human path for tier Low MUST require every deterministic gate to pass on its own; on that path, "no AI findings" is a necessary condition and MUST NOT be treated as a sufficient one.
21. **The diff is untrusted input** (§10.1); the Factory MUST treat the diff as untrusted input.
22. Reviewer instructions MUST be isolated from the diff content; the diff content MUST be passed in as quoted data.
23. The Factory MUST run a **deterministic pre-scan**, outside any LLM, that flags text targeting the reviewer or agent (instruction-like phrases, role markers, hidden or zero-width characters) in files that are **not** control files (§7.13) or declared prompt assets (`factory.yaml` → `promptAssets`, for example prompts in an AI application repository and prompt-injection test fixtures).
24. A pre-scan hit MUST be a security finding, regardless of whether the LLM reviewer recognizes the text.
25. Control files and declared prompt assets, which are exempt from the pre-scan, MUST be protected by independent code owner approval (§7.13).
26. An LLM review pass that returns no findings on a change with a pre-scan hit MUST be treated as invalid.
27. **Who may dismiss a finding** MUST follow the definition in §9.10.
28. The implementer (the implementing agent, or the developer driving an agent locally, §6.15) MAY fix a finding but MUST NOT dismiss it.
29. On zero-human paths, every dismissal MUST require a human and MUST move the change to the 1-approval path.
30. **Per-area enforcement rollout.** Every area MUST start in `shadow` (findings recorded, not posted), move to `advisory` (posted, not blocking), then `blocking`.
31. Promoting an area's AI review mode MUST require that the measurements in §7.12 meet thresholds agreed with the area owner.
32. **Budget.** Every review depth MUST have a per-PR cost ceiling and a latency SLO set in the pilot.
33. Scheduled scans MUST have their own budget.
34. When the budget is exceeded on a **tier Low** change, the review MAY fall to the next lighter depth and MUST record that it did so.
35. On **tier Medium/High**, the review MUST NOT be downgraded when the budget is exceeded: the review waits in the queue or the task is escalated.

**Rationale.** AI review runs continuously, like a spell checker while writing. In the cited practice, most changes are reviewed and fixed by an agent at least once before a human looks [R29].

- **Pre-existing** (item 6): the author is not blamed for bugs the change did not cause; those bugs are still handled through codebase gardening.
- **Findings proven by execution** (item 16): ensembling multiple models to attack the same diff and keeping only findings that have been proven reduces noise better than voting alone [R06].
- **Add-only authority** (items 18–20): AI review can raise the level of control (block, escalate) but does not replace other controls.
- **The diff is untrusted input** (items 21–26): comments, strings or documents in a change can try to steer the reviewer ("nothing to see here"). The pre-scan sits outside any LLM because the reviewer is itself the target of the injection: the pre-scan result cannot depend on whether the LLM reviewer recognizes the text. Control files and declared prompt assets are exempt from the pre-scan because their purpose is to address the agent.
- **Per-area rollout** (items 30–31): the measurements for promoting a mode may include precision from disposition data and the rate of harmful suggestions on PRs known to be clean (§7.12). The `shadow` → `advisory` → `blocking` roadmap reconciles §7.7 (AI findings are advisory until calibrated) with the severity mapping in §7.5.
- **Budget** (items 34–35): at tier Medium/High, cost must not decide the level of risk control.

**Example.** The reference architecture of the pipeline:

```text
Diff + whole repository + stated intent (Change Brief / spec)
   ↓
Parallel bug hunters (independent contexts; file+line evidence required)
   ↓
Finding verifiers (separate agents vote; post only if enough agree it is real)
   ↓
Filter (drop style-only, diagnostics-only, cleanup-only findings; filters A/B-tested)
   ↓
Severity ranking and tagging: Important | Nit | Pre-existing
   ↓
Inline comments sorted by severity + summary that starts with a count
```

Reported results, for reference only (vendor-reported, not a benchmark): the share of internal PRs with substantive review comments rose from 16% to 54%; most PRs over 1,000 lines receive findings while most small PRs come back clean [R29].

## 7.10. Pre-PR self-review

Before opening a PR, the implementing agent, or the developer driving it:

1. MUST run the mandatory fast checks (format, lint, type, focused tests, build).
2. SHOULD run a review pass on the diff using an agent session with fresh context (for example a subagent).
3. MAY run optional skills: bug hunting, diff simplification, end-to-end behavior checks, and design guideline checks for UI changes.

**Rationale.** Fresh context matters: a reviewer that shares context with the implementer tends to share its misunderstandings too (§7.7). Self-review is a recommendation, not a gate, because the Factory cannot observe what happens on the developer's machine; what is actually guaranteed lies in the centralized required checks and the review on the PR (§6.15, §7.15).

## 7.11. Review convergence control

1. After 3 review/fix rounds, the task MUST stop and move to `ESCALATED` so the accountable person can reconsider the approach.
2. After the first full review pass, re-review MUST consider only the diff portion.
3. The reviewer SHOULD flag the class of bug, not just each instance, so the author fixes every occurrence in one pass.
4. The reviewer MUST NOT repost a finding that has already been posted or has a disposition.
5. The Factory SHOULD NOT run multiple review rounds by default; it runs additional rounds only when there are new commits worth it.
6. The Factory MUST track the number of rounds to pass together with recall, prioritizing catching real defects first and reducing the number of rounds afterward.

**Rationale.** The author agent and the reviewer agent can loop forever without converging. In one reported case, a PR went through more than 50 rounds overnight; about 59% of the repeated findings were in code that had existed since the reviewer's first pass, meaning each round the reviewer raised a few more old issues rather than new ones [R29]. Noise is also more expensive when the author is an agent: a human skips a nit in a few seconds, while an agent fixes it and triggers another round.

Reference results after applying these controls (vendor-reported): pass after one round 65% → 78%; PRs needing five or more rounds 8.9% → 1.5%; average number of review passes to pass 2.08 → 1.32 [R29].

## 7.12. Measuring reviewer quality

1. The Factory MUST classify every reviewer comment by the author's response: **fixed**, **wrong**, **intentional behavior** or **didn't matter**.
2. The Factory MUST run the reviewer on **PRs known to be clean** and count the findings that would have led to a *harmful* change.
3. The Factory MUST maintain an **incident-derived eval set**: past PRs that caused an incident or had to be reverted, with the bug location confirmed by engineers who understand the code.
4. The incident-derived eval set MUST be run on a schedule (for example daily) and MUST be supplemented after every incident.
5. Every change to the reviewer's prompt, tools, model or filters MUST be scored on the full eval set, and MUST be kept only if the score improves (§9.2).

**Rationale.** Classifying comments by the author's response yields a precision signal without manual labeling. Running the reviewer on PRs known to be clean measures the other side: whether the reviewer pushes the author toward a harmful change. The incident-derived eval set keeps the recall measurement tied to defects that actually happened.

## 7.13. Verification integrity: preventing check gaming

1. Weakening of checks MUST be detected mechanically, not left to the reviewer's attention.
2. The Factory MUST implement the following integrity sensors.

| Integrity sensor | Detects |
|---|---|
| Test count delta | Fewer tests or assertions in the affected areas compared to the base commit |
| Skip/focus markers | New `.skip`, `.only`, `xit`, `@Disabled`, `pytest.mark.skip` |
| Suppression directives | New `eslint-disable`, `@ts-ignore`, `@ts-expect-error`, `# noqa`, `as any`, `nolint` |
| Snapshot churn | Snapshot/golden file updates without a stated reason tied to an acceptance criterion |
| Threshold change | Lowering coverage, bundle, performance or quality thresholds |
| Control file change | Edits to any control file (defined below) |
| Expected-value edit | Changes to expected values, assertions or golden output in pre-existing tests |
| Self-mocking | Mocking or stubbing the very unit under test, or the code path that the acceptance criterion exercises |
| Vacuous test | Early return, unconditional pass, assertions removed or made always true |
| Tolerance inflation | Increasing timeouts, retries, test sleeps or numeric tolerances |
| Broad catch | New catch-all exception handlers or swallowed errors in changed production code |
| Surviving mutant | Mutation testing on changed lines: a surviving mutant shows the tests do not constrain the change |

3. Mutation testing MUST run before merge for tier Medium/High; in other cases it MUST run on a schedule.

**Control files.** A **control file** is any file that steers the agent or is an input to risk classification and review. Control files are protected paths.

4. The Factory MUST treat the following categories as control files:
   - *Harness files:* `engineering/rules/**`, `engineering/verification/**`, `.factory/evaluations/**`, lint/architecture/test configuration, CI workflow files, and merge queue and branch protection configuration stored as code.
   - *Agent-steering files:* agent instruction files at every level, including `AGENTS.md` and any equivalent file of the runtime in use (for example `CLAUDE.md`), `REVIEW.md` at every level, `.factory/skills/**`, `.factory/playbooks/**`, and hook, agent and MCP/tool configuration.
   - *Risk classification inputs:* `CODEOWNERS`, `factory.yaml` (for example `verificationProfile`, `promptAssets`), `catalog/**` (for example `criticality`, ownership, protected categories).
   - *Check execution inputs:* any file that a mandatory check executes or reads as configuration, including scripts and actions invoked by CI workflows, package manager script definitions (for example `scripts` in `package.json`, targets in a Makefile), test runner configuration and setup files, and the implementation of integrity sensors and separation of duties (SoD) checks.
5. **By field, not by whole file.** For a file that contains both configuration read by a mandatory check and other content (for example `package.json`), only the fields the check reads (for example `scripts`, test runner configuration) MUST be treated as control.
6. A change only to non-control fields MUST NOT be treated as a control file change.
7. Classification by field MUST use a content-analyzing check, not rely only on per-file CODEOWNERS.
8. Updates to the **lifecycle metadata of knowledge artifacts** (`status`, `validatedAgainst`, `supersedes` links) made by a registered trusted mechanism after reconciliation (§5.10) MUST NOT be treated as a control file change requiring approval; any other change to agent guidance files MUST still be treated as control.
9. **Derived, not manually listed.** The set of control files MUST be generated from the definitions of the mandatory checks (what each check runs and reads) plus the categories declared above.
10. A Factory diagnostic command (for example `factory doctor`) MUST report files that a mandatory check uses but the control file set does not yet cover.

**Approval of control file changes.**

11. Every change to a control file, whether written by an agent or a human, MUST be approved by an independent code owner of that file (§9.10).
12. A change to a control file MUST make the whole change at least tier Medium.
13. A change to a control file MUST NOT be eligible for zero-human merge.
14. The only exception to items 11–13 is a change produced by a registered trusted mechanism under the rules on change direction below.

**Sensor enforcement modes.**

15. Every integrity sensor MUST have an enforcement mode (`observe` | `warn` | `blocking`).
16. Every integrity sensor MUST follow the rule lifecycle (§8.3) and MAY be introduced at `observe`, then promoted based on false-positive data, until it reaches the target mode below.

Target enforcement mode of the integrity sensors (the minimum for L2):

- `blocking`: test count delta, skip/focus markers, suppression directives, control file change, expected-value edit, self-mocking, vacuous test.
- `warn`: tolerance inflation, broad catch, snapshot churn, threshold changes outside control files.
- Mutation testing: `observe` (items 17–18).

17. Mutation testing MUST be scored by the **mutation score on changed lines** against a threshold set after a baseline exists, and MUST NOT consider individual mutants.
18. Mutation testing MUST start at `observe`.
19. A profile MAY run any sensor at `observe` while onboarding a repository.
20. An L2 conformance claim (§20.4) MUST meet at least the target modes above.
21. A hit from a `blocking` sensor MUST have an explanation accepted by an independent person (§9.10).
22. A `warn` hit MUST be displayed to the approver; a `warn` hit does not need separate acceptance.
23. **Intentional expected-value edits.** An `integrity.expected-value-edit` hit MUST be lowered to `warn` for that change alone when all three conditions hold: the edited assertion is in a test linked to an acceptance criterion of the task; that criterion has passed `requirement-review` and describes new behavior that contradicts the old value; and the PR shows the old/new value pair to the approver.
24. Every other expected-value edit MUST follow the sensor's mode.
25. **Pilot mode.** In pilot mode (§20.8), integrity sensors MAY run at `warn` until false-positive data exists for promotion under §8.3, except `integrity.skip-focus` and `integrity.control-file-change`.
26. In pilot mode, `integrity.skip-focus` and `integrity.control-file-change` MUST run at `blocking`.
27. Pilot mode MUST NOT be considered sufficient for CORE-VER-08 (L2).
28. A behavior change and the weakening of the check that would have caught it MUST NOT land in the same unreviewed PR.
29. Reproduction tests and acceptance tests written by the verifier for a task MUST be read-only to the implementer of that task.

**Direction of control file changes.**

30. Control files SHOULD be machine-readable (structured configuration, override files validated by schema), so the Factory can classify every change to them by direction.
31. The Factory MUST classify control file changes by direction and apply the minimum controls in the following table.

| Direction | Example | Minimum control |
|---|---|---|
| **Loosening** | Adding a baseline entry. Adding a waiver with no expiry (item 44). Permanently lowering a requirement, threshold, criticality, required check or AI review mode. Disabling or downgrading a rule or check. Removing an owner. Extending the skip list in `REVIEW.md`. Adding a configuration-as-code escape hatch | Tier High (A3), plus an independent code owner of the file |
| **Tightening or neutral** | Removing a baseline entry. Adding a rule or check. Raising a threshold. Adding an owner. Documentation-only edits in agent-steering files. Adding an exclusion for a path declared as generated or vendored in the catalog (the declaration is itself a control file change) | At least tier Medium (A2), plus an independent code owner of the file |
| **Trusted mechanism** | A registered automation whose output only tightens or is byte-identical to a pinned template: shrinking the baseline after an entry becomes stale, the baseline snapshot CI takes when a check is activated (§8.7), or generator output identical to its template version | MAY merge without human approval, provided the mechanism is registered, the output is verified to be of exactly that kind, and the change contains nothing else |

32. A change that the Factory cannot classify MUST be treated as loosening.

An **escape hatch** is configuration-as-code that the Factory cannot analyze.

33. An escape hatch MUST be tier High.
34. An escape hatch MUST be verified by a **dual run**: a signed result with the base configuration and an unsigned reference result with the change's configuration.
35. After merge, an escape hatch MUST be re-verified on main and MUST be reverted automatically if it fails.
36. A trusted mechanism MUST be registered in the policy registry with an owner.
37. Registering or changing a trusted mechanism MUST be treated as a control file change.

**Integrity check IDs.**

38. Integrity sensors MUST be implemented as checks with the following stable IDs: `integrity.test-count-delta`, `integrity.skip-focus`, `integrity.suppressions`, `integrity.snapshot-churn`, `integrity.threshold-change`, `integrity.control-file-change`, `integrity.expected-value-edit`, `integrity.self-mocking`, `integrity.vacuous-test`, `integrity.tolerance-inflation`, `integrity.broad-catch`, `integrity.mutation-score`.
39. A profile MUST provide detectors for its stack (for example which test framework markers mean "skip").
40. A profile MAY add sensors.
41. A profile MUST NOT remove a core sensor or set its target enforcement lower than the default.

**Comments as licenses.**

42. The Factory MUST treat new comments that justify workarounds and suppressions as an integrity signal.
43. The Factory SHOULD prefer an executable check with an actionable message over an explanatory comment.

**Time-limited waivers.**

44. A waiver with an expiry (disposition `waived`, §7.5) MUST be approved under §7.5 and §9.10, and MUST NOT be classified as loosening under item 31, even when it is stored in a control file. Only permanent loosening of a rule, threshold or required check, including a waiver with no expiry, MUST follow the loosening path (tier High).

**Rationale.** An agent optimizing for green checks can weaken the check instead of fixing the code. This is a primary failure mode, and reviewer attention is not reliable enough to catch it.

A **surviving mutant** is the strongest sensor for tampering with the test oracle: if a mutant on a changed line survives, the tests do not actually constrain that change. Mutation testing is scored by score rather than per mutant (item 17) because equivalent mutants always survive.

**Intentional expected-value edits** (item 23). A bug fix often has to modify an old test that asserts the wrong behavior. This exception lets that legitimate case through when it is anchored to a reviewed acceptance criterion, without opening the door to arbitrary expected-value edits.

**Direction of control file changes** (items 30–35). Classification by direction means that tightening changes do not bear the same control as loosening changes. A change that cannot be classified is treated as loosening under the safe-default principle. The dual run for escape hatches allows comparing results under the old and new configuration without letting the new configuration sign its own results.

**Time-limited waivers** (item 44) do not follow the loosening path because they already have an owner, scope, expiry and an independent code owner's approval (§7.5, §9.10), and they expire on their own. Separating the two paths this way avoids requiring two approvers for every short-term waiver.

**Stable integrity check IDs** let every profile report sensors the same way, so metrics are comparable across repositories and stacks.

**Comments as licenses** (items 42–43). A comment that justifies a workaround ("using a timer here because…") is read by the agent as a precedent, and the workaround spreads. Such comments are also not executable and quickly become stale. One team reported banning about 99% of comments in code and replacing them with lint errors whose messages point to the root-cause fix [R05]. The Factory does not mandate that comment-ban policy; item 42 only treats new comments that justify workarounds and suppressions as an integrity signal.

**Example.** Control files by field (items 5–6): changing `"test"` to `vitest --passWithNoTests` changes the meaning of a check, even though it looks like a minor edit in `package.json`. Conversely, merely bumping a dependency version in `package.json` does not touch a field the check reads, so it is not a control file change.

Scenario prevented: a "docs" change that adds `src/billing/**` to the skip list of `REVIEW.md`, or lowers `criticality` in the catalog, is auto-merged and silently weakens every later review and classification.

## 7.14. Flaky check policy

1. **Only pre-existing instability counts as flaky.** A check MUST be counted as flaky only if it is in the flaky registry or has failed intermittently on the base commit or in recent history on main.
2. **Non-determinism is `FAIL`.** A check that fails and then passes on rerun, and is not yet known to be flaky, MUST be treated as a failure (`FAIL`).
3. Changes in time-sensitive areas MUST rerun the affected tests repeatedly (stress mode) before review.
4. **Quarantine MUST be decided by the check's owner**; the agent working on the task MUST NOT quarantine a check.
5. A quarantined check MUST still run and report, be excluded from blocking and from triggering fixes, and have an owner and a deadline to fix it.
6. A quarantined check that protects a critical acceptance criterion MUST have a replacement verifier or an independent human disposition (§9.10).
7. Reruns MUST be recorded with both results.
8. Flaky findings MUST be fed into codebase gardening and the flaky check rate metric (§13.2).
9. A retry that ends in `ERROR` MUST be treated as an infrastructure event, not as flaky, and MUST be reported to the check's owner.

**Rationale.** Flaky checks poison the fix loop: the agent "fixes" errors that do not exist until its budget runs out. However, a sloppy flaky policy is even worse: it hides non-determinism introduced by the change itself, which is common when working with timeouts, retries, concurrency and caching, especially when the check covers code the PR touches.

## 7.15. Trusted evidence: verifier separated from the change

A change MUST NOT influence how it is verified or how the results are recorded. The Factory MUST meet the following requirements (numbering is kept stable for references of the form "§7.15 item N").

1. **Central definition.**
   - Mandatory checks MUST run from a single pipeline definition, adapter set and configuration version that the change cannot modify or bypass.
   - That definition MUST be pinned by reference and enforced by central policy across every governed repository.
2. **Repository-defined entry points only through control files.**
   - An adapter MAY call tools directly with centrally pinned configuration.
   - An adapter MAY call a repository-defined command (for example `npm test`, a Makefile target, an nx, turbo, bazel or gradle task) if and only if: (a) the command's definition is a control file classified by field (§7.13), so a change cannot modify it outside the control file approval path; (b) the adapter records the command and the digest of the definition in the evidence; and (c) the adapter verifies that the definition is unchanged relative to the base, or that the change has approval by direction under §7.13.
   - An adapter MUST NOT execute a repository-defined entry point that does not satisfy (a)–(c).
   - A repository's local overrides MUST be read only from a schema-validated override file, and changes to that file MUST be classified by direction (§7.13).
3. **Job separation (L3).**
   - The job that executes the change's code MUST NOT sign results or write to the evidence store.
   - A separate attestation job MUST run on a trusted runner.
   - The attestation job MUST NOT check out or execute the change's code.
   - The attestation job MUST receive results by digest, verify `verifierRef`, and sign them: keyless via OIDC workload identity when the platform supports it, otherwise with a key usable only by that runner's identity.
4. **Evidence store (L3).**
   - Evidence MUST be written only by the attestation job, to storage outside the repository, with retention classes (§10.7).
   - Policy decisions MUST accept only runs that have an attestation.
5. **Verifier changes.**
   - Changes to the central pipeline MUST go through their own review pipeline.
   - Before taking effect, a verifier change MUST run in shadow mode on governed repositories, then be rolled out as a canary.
   - A broken verifier MUST NOT silently block the whole organization (P26).
6. **Residual risk.**
   - The Factory SHOULD layer mitigations for the residual risk: independent review, seeded defects (§9.2), staging smoke tests and staged release.
7. **Requirements for the CI platform.** The CI platform MUST meet the following requirements; the Factory MAY use any platform that meets them:
   - A centrally enforced pipeline, pinned by ref, that a change cannot modify or bypass.
   - OIDC workload identity for jobs.
   - A dedicated trusted runner for attestation.
   - Attestation signing.
   - An evidence store with identity-scoped write permissions and retention.
8. Pilot mode (§20.8) MUST meet items 1–2.
9. Items 3–4 MUST be met for L3.
10. A profile MAY require items 3–4 earlier than L3.

**Rationale.** §7.13 protects the files that define verification; this section protects the **process** that produces evidence. Mandatory checks execute the very code under review, so if a change can influence how it is verified or how the results are recorded, the evidence loses its value.

**Repository-defined entry points** (item 2) are allowed to avoid drift between local and CI, and so that teams outside frontend do not have to rewrite how every tool is invoked. Integrity still holds because the command's definition is a control file: a change cannot redefine how it is verified without going through control file approval by direction, and the digest of the definition is in the evidence.

**Residual risk** (item 6): tests still execute the change's code, and that code can detect the test environment; therefore the mitigation is layered.

Why the CI platform needs each capability in item 7:

| Requirement | Why |
|---|---|
| Centrally enforced pipeline, pinned by ref | The verifier is not under the author's control |
| OIDC workload identity for jobs | Keyless signing; no static secrets |
| Dedicated trusted runner for attestation | Attestation never shares a machine with the change's code |
| Attestation signing | Evidence cannot be forged |
| Evidence store with identity-scoped writes and retention | Only the attestation job can write |

## 7.16. Verification Adapter Contract

A **verification adapter** is a component that executes one or more checks for a toolchain (language, framework, testing framework) and returns results in the canonical schema.

1. The Control Plane and the task runner MUST invoke verification only through registered adapters.
2. Workflow specs and profiles MUST reference checks by ID and MUST NOT define their own way of invoking a verifier.
3. Every verification adapter MUST satisfy the following table.

| Component | Requirement |
|---|---|
| Identity | `adapterId`, `version`, `owner`; registered in the Verification Adapter Registry (§3.2) and centrally defined (§7.15 item 1) |
| Input | A `VerificationRequest` (Appendix K): the subject (repository and commit/base SHA, or the channel's revision ID), the resolved plan (check ID, check version, parameters), work item, requirement ID, gate |
| Toolchain | Pinned tool version and image digest. Repository-defined entry points may be called only under the conditions of §7.15 item 2; repository overrides may be read only from a schema-validated override file |
| Execution environment | `executor` (`ci`, `agent-sandbox`, `local`), container image digest, resource limits, clean workspace |
| Sandbox and permissions | Network egress blocked by default, opened only by allowlist; no secrets by default, secrets granted only by scope and approval; source code read-only outside the declared output directory; no production credentials (§10.1) |
| Checks | Declares the check IDs the adapter implements and the evidence layers it can observe; a check that needs a layer it cannot observe returns `NOT_ASSESSED` |
| Output | One `CheckResult` per check, conforming to Appendix K; never adds a status or disposition |
| Evidence | Artifacts with URI, digest and retention class, written only to the declared output; attestation per §7.15 from L3 |
| Timeout | Per check and for the whole run; a timeout is `ERROR`, not `FAIL` |
| Error handling | Tool crash, infrastructure error or unavailable service → `ERROR`. A failing check → `FAIL` with a structured diagnostic (silent success, actionable failure, §3.5). Partial results are allowed; a check that did not run is `NOT_ASSESSED` with a reason |
| Determinism | The same input yields the same result, except for checks in the flaky registry (§7.14); safe to rerun |

4. **Adapters are verified like checks.** Every adapter MUST have a repository fixture; the registry's CI MUST prove that each check the adapter declares fails on a violating fixture (mutation proof, §8.7).
5. **An adapter change is a verifier change.** An adapter change MUST go through its own review pipeline, run in shadow, then canary (§7.15 item 5).
6. **The `local` executor does not satisfy required checks.** A result with `executor: local` (a developer machine) MUST be used only for the fast loop and MUST NOT satisfy a required check at the `merge` or `publish` gate.
7. A result from `agent-sandbox` MUST be counted only when the sandbox is managed by the central pipeline (§7.15 item 1).
8. **There is no other way to invoke a verifier.** A workflow, playbook or skill that needs a new verification MUST register the check (and the adapter if needed) in the registry and MUST NOT embed the command in the workflow spec.

**Rationale.** Because the Control Plane and the task runner invoke verification only through registered adapters, and workflow specs and profiles reference checks only by ID, the same workflow can run across many repositories, languages and testing frameworks. Adding support for a new toolchain means adding an adapter, not modifying workflows. Distinguishing `ERROR` from `FAIL` in the adapter keeps the agent from spending loop budget to "fix" infrastructure errors.
