# 17. Standard ownership and governance

## 17.1. Ownership model

| Asset | Accountable owner | Change approval |
|---|---|---|
| Canonical metadata schema | Factory architecture owner | Architecture review |
| Repository metadata | Repository/service owner | Repository CODEOWNERS or equivalent |
| Shared engineering rules | Engineering standards owner | Rule owner + owners of affected repositories |
| Local architecture rules | Repository architect/code owner | Repository owner |
| Skills and playbooks (Task Workflow Specification) | Skill/playbook owner | Eval gate + designated reviewer; control file rules (§7.13) |
| Knowledge artifact | Owner declared in the artifact's metadata | Authority per §5.10 |
| Verification adapter | Verification platform owner | Verifier pipeline review (§7.15) |
| Verification profile | Verification platform owner | Profile owner and service/repository owner |
| Agent/tool permission policy | Security/platform owner | Security approval |
| Eval dataset and graders | Eval owner | Human calibration for subjective graders |
| Release and production policy | SRE/service owner | Operations/security policy |
| AISF Standard, vocabulary and schemas | Standard owner (Factory owner) | Standard change process (§17.4) |
| Domain profile, e.g., Frontend Profile | Profile owner | Profile owner; Standard owner for extensions and deviations (§20.2) |
| Conformance claim | Owner of the claimed scope | Independent assessor (§20.5) |

## 17.2. Operating cadence

**Per task:** task intake, execution trace, verification, evidence, review, delivery.

**Per PR:** review code/evidence/risk; record confirmed findings and dispositions.

**Weekly during the pilot:** review failed checks, repair loops, false positives, agent interventions, tool failures, review rounds-to-pass, finding disposition mix, circuit breaker events, basis changes, rejected handoffs and knowledge conflicts.

**Monthly:** review rule health, skill/playbook evals, workflow metrics (§13.7), the `NEEDS_REVALIDATION` backlog, documentation/architecture drift, cost per accepted and verified work item, verification profile changes, bypass rate and developer feedback.

**After a significant incident:** root cause analysis and owned preventive actions; update datasets and recovery procedures.

Do not create a meeting for every automation. Use dashboards and exception-based review where possible.

## 17.3. Adoption and change management

The Factory is an internal product: adoption must be earned, not mandated. Observed patterns [R28]:

- Usage spreads mainly by word of mouth, and the bottleneck shifts from model capability to people not knowing what the system can do. Invest in showcases, hands-on examples and an easily browsable skill catalog.
- "You can't just give people an empty box": provide role-based plugins, skills and playbooks so that first use succeeds.
- Agents connected to internal systems (source, chat, documentation, telemetry, data) are far more useful than generic agents. Integration is a capability investment, subject to the constraints of §6.10 and §10.5.
- Pairing domain experts with Factory builders improves skills in areas where developers lack expertise.

Factory practices:

- Publish a golden path for each task class and measure the first-time success rate.
- Train developers to steer agents, read evidence and sign off with accountability. Understanding the change remains the requester's responsibility.
- Track the bypass rate and interview developers who bypass the Factory.
- Guard against skill atrophy: keep engineers directly debugging and designing patterns.
- Clarify the changing engineer role: more on defining outcomes, judgment and verification; less on manual implementation [R28].

## 17.4. Standard governance

**Versioning.**

1. The Standard MUST use semantic versioning as follows:
   - *Major:* a requirement is added, removed or tightened in a way that may invalidate an existing conformance claim, or a vocabulary term changes meaning.
   - *Minor:* a new optional extension point, new informative guidance, or a new requirement that takes effect only from a future date. A minor version's new requirements bind only claims pinning that version or later (item 9), so they do not invalidate existing claims.
   - *Patch:* editorial fixes.
2. Profiles and conformance claims MUST pin an exact Standard version.
3. A new major version MUST give profiles a transition window (*suggested* two quarters); within that window, claims against the previous major remain valid.

**Change process.**

4. Changes MUST be proposed as a pull request to the Standard repository, stating the problem, evidence (pilot data, incidents or conformance assessments), affected requirement IDs and transition impact.
5. The Standard owner MUST approve changes; profile owners MUST be consulted. Upstream proposals from profiles (§20.2, PC-7) MUST follow the same process.
6. The Standard MUST change requirements only on the basis of evidence from pilots, incidents or conformance assessments. Editorial findings SHOULD be batched into patch versions.
7. A new requirement originating from review without pilot data MUST have a future effective date and SHOULD be validated through the end-to-end flow of pilot mode (§20.8 item 2) before that date. If the evidence refutes it, the requirement MUST be revised in a minor version before it takes effect.

**Effective dates.**

8. The Core Requirements Index (§20.6) MUST record an effective date for each requirement that does not take effect immediately. Before that date, the requirement is assessed as SHOULD; from that date, as MUST.
9. A conformance claim MUST be assessed against the Standard version it pins, using that version's own effective dates, as of the assessment date. Requirements added in a later version bind a scope only when that scope's claim is renewed or reassessed against the later version.

**Transition impact.**

10. Every change that may affect mandatory requirements, the Core Requirements Index, the vocabulary, schemas or conformance levels MUST state in its pull request: the affected requirement IDs, the claims that may become invalid, the version selected per item 1, and the transition steps.
11. A change MUST NOT alter the meaning of a requirement, or add a mandatory requirement that takes effect immediately, while keeping the same version.

**Synchronization and machine-readable sources.**

12. Every concept, state, enum, lifecycle and requirement MUST be changed simultaneously in the Standard chapters, the Core Requirements Index (§20.6), the vocabulary (`aisf-vocabulary.yaml`, Appendix J) and the schemas (Appendix K) in the same change.
13. The vocabulary (`aisf-vocabulary.yaml`, presented in Appendix J) and the schemas (Appendix K) in the Standard repository are the canonical source. Machine-readable copies in the policy repository (vocabulary, JSON Schema) MUST be generated from this canonical source or checked by CI to match it.
14. The Standard repository's CI MUST check that every enumerated value and every ID used in the Standard and in registered profiles exists in the vocabulary or a registry, and that every type in the schemas uses the vocabulary's exact values.
15. The Standard repository's CI SHOULD check that **Rationale**/**Example** paragraphs and informative parts (§1–3, §14–16, §17.1–17.3, §18, §19, §21, Appendices A, B, D, F–I) contain no uppercase RFC 2119 keywords, and that every referenced item number exists.

**Release, registry and prerequisites.**

16. Each version MUST be released with release notes sent to profile owners and registered adopters, stating the affected requirement IDs, effective dates and transition steps.
17. The Standard owner MUST maintain a registry of registered profiles (`ProfileRegistryEntry`, Appendix K), adopters and conformance claims (scope, pinned Standard and profile versions, level, mode, expiry, assessor).
18. Before any conformance claim in `standard` mode (§20.5 item 7) is accepted, there MUST be a named Standard owner and a named group of independent assessors (§20.5 item 3) recorded in the registry. Claims in `pilot` mode (§20.8) MAY be recorded before then.
19. The Standard owner MAY delegate deviation approval (PC-4) to profile owners within declared limits (which requirements, maximum duration, count). The delegation MUST be recorded in the registry; deviations outside the limits, and every deviation touching separation of duties (§9.10) or data governance (§10.2), MUST be approved by the Standard owner.

**Rationale.** The Standard changes only on evidence because review rounds without pilot data are the documentation version of the non-converging review loop described in §7.11: each round adds a few requirements without proving their value. Setting future effective dates for requirements coming from review makes it possible to keep good ideas while still forcing them to be validated through the end-to-end pilot flow (§15.0, §20.8) before they become mandatory.
