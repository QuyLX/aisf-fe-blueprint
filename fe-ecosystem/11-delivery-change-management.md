[Contents](README.md) · [← 10. Autonomy Policy](10-autonomy-policy.md) · [12. Evaluation →](12-evaluation.md)

<a id="s11"></a>

# 11. Delivery & Change Management

<a id="s11-1"></a>

## 11.1. A PR is a change package with evidence

Generated automatically from `VerificationRun` + `PolicyDecision`: intent, work item, surface, affected contracts/components, risk + rationale, checks + evidence, visual/a11y/perf artifacts, rollout/recovery, approval slots, and the non-`PASS` checks with their disposition. Every PR contains the Standard's mandatory **Testing / Risk / How to revert** sections (Standard §7.6 and Appendix G): the agent drafts them and the requester signs off. The description is ordered to serve the reviewer: intent and sub-tier first; then the decisions that require human judgment and what the checks did *not* catch; then open findings, the diff by surface, and evidence.

<a id="s11-2"></a>

## 11.2. Design System / token

Classify the diff (breaking / non-breaking / semantic) → consumers from the portfolio index + Token Usage Index + sentinel → DS checks in the platform repo → publish a **prerelease** → the mandatory pipeline of each affected consumer repo runs verification against the prerelease (no merge needed) → impact report → the DS owner approves; consumer code owners only approve the visual baseline of surfaces with a visual diff ([§10.4](10-autonomy-policy.md#s10-4)) → release + changeset → a bot opens an **adoption PR** with a codemod to each repo; the vertical owner merges within the version skew window ([§4.8](04-technical-foundation.md#s4-8)). Adoption PRs are opened by platform automation, and the owner accountable for that automation is the platform team, so a vertical code owner fills the A1 slot ([§10.4](10-autonomy-policy.md#s10-4)).

*Monorepo option ([§3.3](03-target-architecture.md#s3-3)):* no prerelease or adoption PR. A DS change is one atomic PR containing the package and the codemod applied to every consumer; resolve selects affected surfaces via the project graph, Token Usage Index and sentinel set; approval follows [§10.4](10-autonomy-policy.md#s10-4) (DS owner, plus code owners of consumers with a visual diff or with files modified by the codemod).

<a id="s11-3"></a>

## 11.3. API contract

A new contract is published to the contract registry → the portfolio index identifies consumer repos → a bot opens a PR regenerating client + mock in each repo → type errors indicate what must be fixed → Pact consumer test → `can-i-deploy`. Breaking changes follow **expand → migrate → contract**. *Monorepo option:* a bot opens one regenerate PR for all consumers in the `frontend` repo; the portfolio index is used only for consumers outside the repo.

<a id="s11-4"></a>

## 11.4. Content channel (verify-on-publish, Standard §12.6)

Content from the CMS is a production change that **does not go through a PR**, so it has its own pipeline:

```text
Editor "Request publish" → webhook → preview build of affected pages
→ public-web/docs checks (links, SEO metadata, a11y, consent, rendering, locale)
→ PASS: publish + revalidate cache | FAIL: blocked, editor notified with reasons
```

- Content touching **pricing, terms or legal commitments** (declared content type) requires approval from the business owner.
- A **content model** change is a code change (PR; R3 if migration is needed).
- Publish evidence is stored as a `VerificationRun` (subject = content revision).
- **Never block mitigating actions:** unpublish, takedown of content for legal or incident reasons, and rollback to a previously verified revision go straight through, without checks (audit only).
- **Hotfix publish** (fixing incorrect content currently displayed): the business owner may publish first, with verification running immediately after. A hotfix request declares a **fallback target**: unpublish the affected page, or a verified revision that the owner confirms is correct. The fallback target is never the incorrect revision being fixed. FAIL → automatically revert to the fallback target and alert; if no fallback target can be declared, the default is unpublish.
- **Time budget:** verify-on-publish ≤ 5 minutes (suggested); build and check only affected pages (content graph: revision → consuming pages); slow checks such as lab performance run after publish and do not block.
- **Scheduled publish:** verification runs at scheduling time and runs again immediately before the publish time.
- **Risk and approval.** The content sub-tier comes from the declared content type: pricing, terms and legal content require a business owner slot; campaign pages are R1; docs pages are R0 or R1. Agent-authored content follows the same autonomy policy as code ([§10.4](10-autonomy-policy.md#s10-4)), and an agent never publishes a declared sensitive type.
- **Lifecycle.** The Standard's channel lifecycle applies: `REQUESTED → VERIFYING → AWAITING_APPROVAL → PUBLISHED → OBSERVING → COMPLETED`, plus `ROLLED_BACK` and `BLOCKED`. Evidence is a `VerificationRun` with `channel: content` and `subject.revisionId`.
- **Feature flag changes** (channel `config`) follow the same model. A targeting change made by the flag owner is R1. Each flag declares `safeState` ([§4.5](04-technical-foundation.md#s4-5), FE-FLAG-001): `off` (default for feature flags: turning the flag off returns the system to the previous verified behavior) or `on` (the off state removes a security, privacy or legal control, e.g., a flag enabling step-up auth, a new CSP or consent gating). Only returning a flag to `safeState` is a mitigating action and is never blocked; moving a flag out of its `safeState` is a change governed by the autonomy policy, and for a `safeState: on` flag it is a loosening (R3, requires the security owner). A flag gating `x:security-sensitive` behavior or pricing is R3.

<a id="s11-5"></a>

## 11.5. Release & incident

Immutable artifacts; preview → staging → production; canary for surfaces with `criticality` `high` or above; per-surface monitoring; on-call; rollback = redeploy the previous artifact.

**Channel `release` (pre-production gate).** Promoting a merged image to production is a change that does not go through a PR, so the profile governs it as a change channel, with the same model as content and config: the subject is the image digest; evidence is a `VerificationRun` with `channel: release` on the staging environment; checks gated before production ([§8.1](08-verification.md#s8-1)) MUST be satisfied before promotion; the vertical's release owner approves according to the autonomy policy. Rollback to a previously verified image is a mitigating action and is never blocked.

**Incidents** are classified by cause (requirement, contract, DS, verification gap, impact miss, incorrect or stale documentation, agent playbook, release…) → the output is a rule, test (playbook `regression-from-incident`, [§9.10](09-agent-operating-model.md#s9-10)), eval case, documentation fix at the source, playbook or ADR — not a checklist addition by default.

<a id="s11-6"></a>

## 11.6. Merge queue (Standard §6.4, §7.4)

Every product repo and the platform repo take changes in through a merge queue. Mandatory deterministic checks re-run on the latest main with the change applied. AI review does not re-run; its result remains bound to the reviewed SHA. Console team modules share one queue, and batching with automatic bisection is allowed.

Failure attribution is operational: re-run the failing check on the latest main, with and without the change.

| Without change | With change | Attribution |
|---|---|---|
| Fail | — | Main is broken. Pause the queue and notify the owner of the breaking change; no budget consumed |
| Pass | Fail | Caused by this change, including semantic conflicts with just-merged work; consumes `mergeQueueAttempts` |
| Pass | Pass (failure not reproducible) | Flake candidate ([§8.8](08-verification.md#s8-8)); no budget consumed |

<a id="s11-7"></a>

## 11.7. Change Shepherd (Standard §12.2.1)

For surfaces with `criticality` `high` or above or with a `critical` journey, a Change Shepherd follows each change throughout rollout. It:

1. identifies the change's flags;
2. proposes success and failure signals from RUM, error tracking and OpenTelemetry (a human confirms for R2 and R3);
3. builds a per-change view from existing telemetry;
4. reports health at each rollout stage;
5. recommends continue, pause or rollback.

Returning a flag to `safeState` and pausing a rollout MAY be automated for R0–R2 via an audited OpenFeature provider. Image rollback, revert on main, and anything affecting data require a human.

<a id="s11-8"></a>

## 11.8. Degraded mode & break-glass (Standard §12.7)

The degraded behavior of each Factory component and the break-glass rules (scope of use, two-person activation, time limit, checks that can still run still run, post-hoc verification, not used for control files or loosening changes) belong to the Standard. The profile adds:

| Component down | Frontend-specific behavior |
|---|---|
| AI reviewer | R0/R1 changes go A1 with slots per [§10.4](10-autonomy-policy.md#s10-4); R2/R3 wait in the queue |
| CMS verify-on-publish | Only mitigating actions (unpublish, takedown, rollback to a verified revision) and owner-approved hotfix publishes, with post-hoc verification |

- **Time limit and post-hoc review:** the profile sets 24 h for both the break-glass duration and post-hoc verification (plus attestation from the Phase 1a follow-up); failure opens an incident and leads to a revert.
- **Drills** once per quarter. Slice 0 has a degraded mode runbook and one break-glass drill; automatic switching to degraded mode is deferred ([§15](15-roadmap.md#s15)).

---

[Contents](README.md) · [← 10. Autonomy Policy](10-autonomy-policy.md) · [12. Evaluation →](12-evaluation.md)
