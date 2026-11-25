# 8. Engineering rules & codebase gardening

## 8.1. Rule categories

1. Each rule MUST belong to a registered category. The core categories are:

| Prefix | Domain | Example |
|---|---|---|
| CODE | Code conventions | naming, file/function complexity, formatting |
| TYPE | Type safety | unsafe casts, ignored errors, unvalidated data at boundaries |
| ARCH | Architecture | dependency direction, layer boundaries, dependency cycles |
| API | Contract | schema compatibility, versioning, deprecation |
| SEC | Security | secret exposure, authorization patterns, unsafe APIs |
| DEP | Dependency | approved packages, licenses, vulnerable versions |
| TEST | Test quality | coverage of important acceptance criteria, unstable tests, test boundaries |
| UX | Accessibility and user experience | semantic controls, keyboard, focus behavior |
| FLAG | Feature flags | owner, default value, fallback, expiry |
| PERM | Permissions in product code | client/server authorization distinction, permission tests |
| STATE | Client- and server-side state management | duplicated server state, effect usage, scheduling primitives |
| PERF | Performance | budgets, hot paths, benchmarks |
| OPS | Operational requirements | timeout, retry, idempotency, logging, recovery |
| AI | Constraints for AI applications | permitted tools, output validation, retrieval and eval requirements |
| DOC | Documentation | broken links, stale architecture documentation, missing ADR references |

2. A profile MAY register additional categories for its domain.
3. Registered categories MUST be listed in the vocabulary (Appendix J).
4. A rule MUST NOT use an unregistered category; using an unregistered category is a registry error.

**Rationale.** The core categories cover domains common to all kinds of software; profiles add categories for their own domains, e.g., the Frontend Profile registers `NEXT`, `CTX`, `DS`, `I18N` and `CFG`. Because every category is in the vocabulary, the registry immediately detects a rule using a self-invented prefix.

## 8.2. Rule definition contract

Schema of the rule definition: `Rule` in Appendix K (K.6).

1. Each rule MUST have the fields required by the `Rule` schema (Appendix K): a stable ID and version, name, category, enforcement mode, scope, detection mechanism, remediation guidance message, owner, and test fixtures (positive and negative).
2. A rule's enforcement mode MUST be one of `observe` | `warn` | `blocking`, mapped to canonical severity (§7.5).
3. A rule MUST NOT have a separate `severity` field.
4. Each rule SHOULD additionally have:
   - Purpose and rationale.
   - Detailed applicability conditions.
   - Known false positive cases.
   - Exception policy and expiry.
   - Rollout state and metrics.

**Rationale.** The required fields allow a rule to be reviewed, tested and removed like any other artifact. The enforcement mode already carries the severity level via the mapping to canonical severity (§7.5), so a separate `severity` field could only conflict with it.

**Example.** A rule definition:

```yaml
id: ARCH-001
version: 1
name: Domain public-interface boundary
category: ARCH
enforcement: blocking
scope:
  include:
    - "src/domains/**"
detection:
  type: dependency-graph
message: "Import from the owning domain's public entry point."
owner: frontend-architecture
exception:
  approvalRequired: true
  expiryRequired: true
```

## 8.3. Rule lifecycle

1. Rules MUST use the following lifecycle:

```text
PROPOSED
 → DRAFT_WITH_FIXTURES
 → OBSERVE
 → WARN
 → BLOCKING
 → MONITOR
 → REVISE / DEPRECATE / RETIRE
```

2. Legacy repositories MAY use a baseline mode so that new changes add no violations while existing debt is gradually addressed.
3. Lifecycle states MUST map to enforcement modes: `OBSERVE` → `observe`, `WARN` → `warn`, `BLOCKING` → `blocking`.
4. A rule MUST be enforced through one or more checks (§8.7).
5. A rule's state MUST always match the lifecycle of the checks that enforce it: `OBSERVE` corresponds to a check in `shadow`; `WARN` corresponds to an `active` check with an `advisory` requirement; `BLOCKING` corresponds to an `active` check with a `required` requirement in at least one profile.

## 8.4. Choosing the right detector

1. The Factory SHOULD choose a detector by problem type:
   - Formatter for formatting.
   - AST lint for syntax and import/call patterns.
   - Type-aware lint for rules that need type information.
   - Compiler/type checker for type correctness.
   - Dependency graph for module boundaries and dependency cycles.
   - Schema validator for metadata and contracts.
   - Static analysis for supported security/data-flow checks.
   - Runtime tests for behavior.
   - AI/human review for intent and judgment.
2. A rule SHOULD NOT use regex when the problem requires understanding the AST, types or graph.
3. Subjective judgment SHOULD NOT be forced into brittle blocking rules.

**Rationale.** AST lint suits syntax and import/call patterns [R32], while type-aware lint is needed for rules that depend on types [R33]. Regex does not understand code structure, so it easily yields both false positives and false negatives when the problem actually requires the AST, types or graph. A subjective judgment forced into a blocking rule breaks on edge cases and erodes trust in the gate; such judgments belong to AI/human review.

## 8.5. Codebase gardening

**Codebase gardening** is a continuous maintenance capability, not a one-time cleanup project.

1. Codebase gardening SHOULD take input from:
   - Recurring review findings.
   - New architecture violations.
   - Flaky tests and reliability issues.
   - Dependency drift.
   - Documentation drift and knowledge artifacts in `NEEDS_REVALIDATION` (§5.8).
   - Dead code and outdated patterns.
   - Production incidents and support tickets.
   - AI review findings tagged Pre-existing, and nightly AI review of existing code (§7.9).
2. Codebase gardening MUST follow this process:
   1. Scan and identify candidates.
   2. Classify root cause and impact.
   3. Prefer prevention through code/API/architecture design where possible.
   4. Otherwise, propose a change to a rule, test, skill, documentation or eval.
   5. Create small, isolated maintenance PRs.
   6. Run the relevant verification profile.
   7. Measure recurrence frequency and false positives.
3. The Factory SHOULD NOT create a rule for every review comment.
4. A recurring review comment SHOULD be promoted along the path: entry in `REVIEW.md` → lint rule, architecture constraint or skill when machine-detectable → removal from `REVIEW.md`.

**Rationale.** The goal is to eliminate recurring classes of defects, not to accumulate an unmaintainable rule set. The promotion path in item 4 lets a convention start as cheap text, move to a machine mechanism once it has proven its value, and then be removed from `REVIEW.md` so the text does not bloat.

## 8.6. Encoding lessons into structure

1. When a defect recurs, the Factory SHOULD fix it at the highest effective level, in order:
   1. **Architecture/API:** eliminate the wrong choice.
   2. **Types:** make invalid states unrepresentable.
   3. **Lint/CI:** forbid the pattern with a diagnostic message that explains the correct fix.
   4. **Tests/verification:** catch the behavior when it cannot be prevented.
   5. **Text:** `REVIEW.md`, skills or `AGENTS.md`, only for what requires judgment.
2. An API that requires finesse to use well SHOULD only be used in product code through approved framework wrappers.
3. When restricting a pattern, the Factory SHOULD block new instances first, then clean up existing instances through gardening PRs.

**Rationale.** The codebase is one of the largest inputs to every agent. Workarounds, anti-patterns and deprecated code get copied because they look like accepted patterns [R05]. Treat the codebase as a garden: constraints are the structure new code grows around, and weeds left alone spread.

The order in item 1 is based on [R05, R06]. A lint message delivers guidance at exactly the moment the agent violates the rule, so it needs to explain the correct fix rather than merely report an error. Experience shows that an API requiring finesse to use well often produces poor code at scale (item 2). Blocking new instances first and gradually cleaning up old ones (item 3) keeps the violation count from growing while old debt is paid down.

**Example.**

- **Architecture/API:** separate top-level directories for processes or layers that must not import each other, enforced by a dependency-graph check.
- **Framework wrapper:** a team forbids bare `useEffect` in product code and allows it only inside their framework's APIs [R05].

## 8.7. Check Registry

A **check** is the unit of execution that enforces the purpose stated by a rule (§8.2). Schema of the check definition: `Check` in Appendix K (K.6). In a check definition:

- `fpSlo` is the ceiling threshold for the owner-confirmed false positive rate.
- `baselineable: false` means the check is never ratcheted with a baseline.
- `evidenceLayer` is the layer the check needs to observe; layers are defined by profiles.
- `implementsRules` lists the IDs of the rules the check enforces; core sensors are defined by §7.13.

Registry rules:

1. **Mutation proof.**
   - Every check MUST have violating fixtures.
   - The registry CI MUST prove the check fails on each violating fixture.
   - A check that has never been seen to fail MUST NOT become `active`.
2. **Enforcement level follows basis.**
   - A check MUST only be `required` when its basis is a mandatory standard, a contract, a decision record in the `ACTIVE` state (§5.8), or a policy of this Standard or of a profile.
   - Ease of implementation MUST NOT be used as a basis.
3. **Honest evidence layer.** If the harness cannot observe the layer the check needs, the result MUST be `NOT_ASSESSED`, and MUST NOT be a simulated `PASS`.
4. **Lifecycle.**
   - The check lifecycle is `planned → shadow → active → deprecated`.
   - A check MUST run in `shadow` (enforcement mode `observe`) for at least the period set by the profile (*suggested* 2–4 weeks) before it can become `active`.
   - A check not in `active` MUST NOT be `required`.
5. **False positives.**
   - False positives MUST only be counted when the check owner, or a delegated person who is not the author of the change, confirms them; disputes MUST be counted separately.
   - When the confirmed rate exceeds `fpSlo` and `autoDemote: true`, the Factory MUST automatically open a proposal to demote the check to `advisory`. Demotion is a loosening change: it MUST only take effect after approval per the loosening row of §7.13. While pending, the check MUST remain blocking and waivers MUST have limited scope and duration.
   - When the confirmed rate exceeds `fpSlo` and `autoDemote: false`, the check MUST remain blocking, the Factory MUST open a priority work item, and waivers MUST have limited scope and duration.
   - `autoDemote` MUST be `false` for every core check (`integrity.*`, `knowledge.impact`, separation of duties checks) and for every check whose `basis.class` is `security`, `privacy`, `legal` or `contract`.
6. **Budget.**
   - A profile MUST set a budget for the number of `active` checks per rollout wave.
   - The registry CI MUST reject anything exceeding the budget.
   - Every check MUST have a specific owner; a check without an owner MUST be demoted or removed.
7. **Ratchet on activation.**
   - When a `baselineable: true` check becomes `active`, CI MUST snapshot the existing violations on the target branch as a baseline; this mechanism is a trusted mechanism (§7.13).
   - New violations MUST be blocked, and the baseline MUST only shrink.
   - A `baselineable: false` check MUST only become `active` when its violation count reaches zero.
8. **Cross-team checks.** A `cross-team-contract` check is a check detected in one domain whose cause lies in another domain. It MUST be implemented in the provider's pipeline and MUST be reported to the consumer via the `assessed-elsewhere` disposition (§7.5).
9. Every check MUST be defined once, in a registry that profiles reference by ID.

**Rationale.** A rule states the purpose; a check is the unit of execution. Separating the two concepts allows one rule to be enforced by multiple checks (e.g., a static check and a runtime check), and each check has its own lifecycle, owner and false positive budget.

**Example.** A check definition:

```yaml
kind: Check
id: integrity.skip-focus
version: 1.0.0
owner: factory-verification
fixOwner: factory-verification
lifecycle: active               # planned | shadow | active | deprecated
ownership: local                # local | cross-team-contract
implementsRules: []             # core sensor defined by §7.13; profile checks list rule IDs
basis: { class: policy, source: "AISF Standard §7.13" }   # standard | contract | security | privacy | legal | policy | record
evidenceLayer: static           # layers are profile-defined; NOT_ASSESSED when the layer cannot be observed
baselineable: false             # false: never ratcheted
fpSlo: 0.02                     # ceiling for owner-confirmed false positives
autoDemote: false               # must be false for core checks and when basis.class ∈ {security, privacy, legal, contract} (§8.7 rule 5)
fixtures:
  conforming: [fixtures/skip-focus/pass]
  violating:  [fixtures/skip-focus/only-marker, fixtures/skip-focus/skip-marker]
remediation: "Remove the marker, or open a separate PR with a justification for an independent approver (§7.13)"
```

Cross-team check (item 8): a provider's API must never return an internal field. The violation is detected on the consumer side, but the cause and fix lie with the provider, so the check runs in the provider's pipeline and the consumer only receives the `assessed-elsewhere` disposition.
