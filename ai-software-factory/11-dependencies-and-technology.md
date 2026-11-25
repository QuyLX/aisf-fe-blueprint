# 11. Dependency engineering & technology governance

## 11.1. Dependency classification

1. The Factory MUST apply the following dependency classification table.

| Type | Example | Primary controls |
|---|---|---|
| Third-party package | npm, Maven, PyPI | Manifest/lockfile, vulnerabilities, license, maintenance level, update policy |
| Internal module | Domain/module imports | Dependency graph, public interface, layer rules |
| Service dependency | Frontend to API, service-to-service | Contract, ownership, timeout, retry, error handling policy |
| Platform/toolchain | Runtime, compiler, CI image, build system | Version policy, compatibility, upgrade plan |
| AI/model dependency | Model API, embedding model, reranker, external AI service | Provider terms, evaluation, latency, cost, fallback, data policy |

## 11.2. Lifecycle

1. The Factory SHOULD apply the following dependency lifecycle sequence, implemented by the Dependency Change workflow (§6.7, Appendix D). Regardless of sequence, the hard gates of §11.3 MUST be applied, and every added or upgraded dependency MUST have a decision record (§11.6), a locked resolved version, and an updated SBOM and inventory.

```text
Need identified
 → Search existing approved capability
 → Research candidate(s)
 → Functional and architecture fit
 → Compatibility / security / license / maintenance / cost review
 → Decision record and approval
 → Install and lock resolved version
 → Tests / SBOM / inventory update
 → Monitor vulnerability and maintenance
 → Upgrade / replace / remove
```

## 11.3. Dependency decision framework

1. A dependency decision MUST assess:
   - Functional fit.
   - Architectural fit and impact on boundaries.
   - Runtime and toolchain compatibility.
   - Security and supply chain risk.
   - License and distribution obligations.
   - Maintenance and release activity.
   - Performance and package size.
   - Ecosystem, documentation, typing and support.
   - Operating/upgrade cost.
   - Reversibility and vendor lock-in.
2. Security, license and data policy constraints MUST be hard gates.
3. A high score in another category MUST NOT compensate for a failed mandatory gate.

**Rationale.** The criteria are not equally weighted. Security, license and data policy are not categories for scoring: a dependency with good functionality, a strong ecosystem and low cost is still rejected if it fails one of those gates.

## 11.4. Policy states

1. The Factory MUST apply the following dependency policy states:
   - `APPROVED`: permitted within the defined scope/version policy.
   - `CONDITIONAL`: permitted only with specific approval or constraints.
   - `DEPRECATED`: no new use; a migration/removal plan exists.
   - `PROHIBITED`: not permitted under current policy.

## 11.5. Reuse strategy

1. The Factory SHOULD prefer mature tools for commodity capabilities (catalog, architecture views, dependency graph, dependency updates, vulnerability scanning, SBOM, browser automation, contract testing, observability).
2. The Factory SHOULD own the canonical metadata, policies, context-building logic, playbook semantics, verification contracts and evidence aggregation.
3. The Factory SHOULD NOT rebuild mature scanners or test engines.

**Example.** Reference choices:

- Catalog: Backstage or equivalent [R13, R14, R30].
- Architecture views: C4 / Structurizr [R16].
- Dependency graph for JS/TS: dependency-cruiser or equivalent [R18].
- Dependency updates: Renovate / Dependabot [R19].
- Vulnerability scanning: OSV-Scanner or an approved scanner [R20].
- SBOM: the CycloneDX ecosystem [R21].
- Browser automation: Playwright [R17].
- Contract testing: Pact where consumer-driven contracts fit.
- Observability: OpenTelemetry plus the existing telemetry backend [R12].

## 11.6. Dependency decision record

1. Every dependency that is added, upgraded by major version, replaced or removed MUST have a decision record per the `DependencyDecisionRecord` schema (Appendix K), including: the dependency and locked version, the action, the approved capabilities searched and alternatives considered, the assessment per §11.3, the hard gate results (security, license, data policy), the policy state (§11.4), the decision maker and the review conditions.
2. Minor or patch version upgrades created by registered automation MAY use a batched record, provided the hard gates are assessed for each dependency.
3. The decision record is evidence of the Dependency Change task and MUST be linked from the PR.

**Rationale.** Dependencies are long-term decisions made very quickly, especially when proposed by agents. The decision record captures why this dependency was chosen over an existing capability, so successors do not have to guess again and the next upgrade has a basis. Batched records for automated minor/patch upgrades keep costs low for the most common type of change.

## 11.7. Vulnerability remediation

1. The Factory MUST scan dependencies for vulnerabilities on every manifest or lockfile change and on a schedule on main.
2. Each known vulnerability in a dependency in use MUST be remediated (upgrade, replacement, removal or verified mitigation) within an SLA by severity (`vulnerabilitySeverity`, Appendix J), *suggested default*: `critical` 7 days, `high` 30 days, `medium` 90 days, `low` at the next planned upgrade, counted from disclosure or detection, whichever is later.
3. A profile MAY shorten the SLA, and MUST NOT extend it.
4. A vulnerability past its SLA without a valid waiver (§7.5, §9.10) MUST be escalated to the scope owner and MUST block adding new dependencies to that scope until resolved.
5. A waiver for a `critical` or `high` vulnerability MUST be approved by the security owner and MUST have a recorded mitigation.

**Rationale.** A severity-based SLA turns "we will upgrade when we have time" into a measurable commitment. The suggested milestones (7/30/90 days) are close to common practice in vulnerability management programs. Blocking new dependencies while overdue vulnerabilities remain applies pressure in the right place without blocking all of the team's work.

## 11.8. License policy

1. The organization MUST maintain a centralized license list with the values `allow`, `deny` and `review` (`licensePolicy`, Appendix J), with an owner (*suggested* the legal owner together with the Factory owner).
2. A license not on the list MUST be treated as `review` and MUST have a decision from the list owner before merge.
3. A dependency with a `deny` license MUST NOT be added.
4. A profile MAY move a license toward `deny`, and MUST NOT move it toward `allow`.
5. The license check MUST run as a required check on every manifest or lockfile change.

**Rationale.** License is a hard gate (§11.3), but few teams can assess licenses themselves. A centralized list held by legal and the Factory owner enables automated checks on every manifest change. Profiles may only tighten because a domain may have stricter distribution obligations (e.g., a public SDK).

## 11.9. AI/model dependencies: deprecation and fallback

1. Every AI/model dependency, including models used in the product and models in agent configuration, MUST reference a pinned immutable snapshot (§9.9 item 8), and MUST NOT use a floating alias.
2. Each AI/model dependency MUST have an owner, and the owner MUST track the provider's deprecation notices; upon a notice, the owner MUST open a Dependency Change task early enough before the end-of-support date to allow re-evaluation (*suggested* at least 30 days).
3. Before switching to another snapshot or provider, the relevant eval set (§9.2) MUST be rerun and MUST NOT regress; for agent configuration, trust MUST be reset per §9.4 item 9 (a different model family or vendor is a major change).
4. Each AI/model dependency on a production path MUST declare fallback behavior for when the provider is unavailable or the snapshot is retired (an evaluated replacement model, degraded mode, or disabling the feature); a fallback using another model MUST meet the same data class constraints (§10.6).
5. A provider automatically switching snapshots without notice MUST be treated as a version change and triggers item 3.

**Rationale.** Models are dependencies with short lifecycles: providers retire snapshots on their schedule, not the organization's. Pinning snapshots makes behavior reproducible; tracking deprecation notices gives enough time to re-evaluate; a pre-declared fallback prevents a feature from ceasing to work on the very day the provider shuts down the model. For agent configuration, changing model family limits trust again (§9.4) because a new model has no record on that task class.
