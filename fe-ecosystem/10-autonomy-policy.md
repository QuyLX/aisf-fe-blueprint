[Contents](README.md) · [← 9. Agent Operating Model](09-agent-operating-model.md) · [11. Delivery & Change Management →](11-delivery-change-management.md)

<a id="s10"></a>

# 10. Autonomy Policy

<a id="s10-1"></a>

## 10.1. Risk — deterministic classifier

The sub-tiers R0–R3 map to the Standard's risk tiers: **R0 and R1 → Low; R2 → Medium; R3 → High** (Standard §9.6). The Standard's controls per tier are the floor; this profile adds checks and autonomy rules per sub-tier.

| Tier | Example |
|---|---|
| **R0** | Docs typo, link fix, comment |
| **R1** | UI copy, style in a `criticality: normal` surface, dependency patch, additional tests |
| **R2** | Feature/bugfix in a `high` or `critical` surface, minor dependency, non-breaking DS change, minor API client change |
| **R3** | `x:security-sensitive`, **privileged paths** of an `x:privileged-admin` surface (below), internal note visibility, billing/payments, AI tool action/sanitizer, loosening a control file, code-form config escape hatch (Standard §7.13), DS breaking change, content model migration, framework major, data deletion |

- Tier = the **max** of all signals: path/surface map, contract change, criticality, dependency, direction of policy change.
- **`x:privileged-admin` surfaces** (Admin, Desk agent workspace) have an **R2** floor for every change. **Privileged paths** are declared in the registry's path-risk map and are always R3: impersonation, cross-tenant and bulk operations, step-up auth, break-glass, displaying or revealing PII, export, audit events, sessions and server-side authorization. The path-risk map is a control file; removing a path from the privileged list is loosening. This floor is stricter than the Standard, which sets High only for the protected groups (Standard §9.6), and those groups remain R3 regardless of path.
- **Indirect risk:** a surface affected through the import graph within the repo, or through the portfolio index when a shared package changes, contributes signals as if directly modified. A package used by an R3 surface (for example `@platform/server`, `@platform/auth`) → changes to it are R3.
- **`codemod.apply`:** the codemod itself is reviewed once (R2/R3); each application is rated by the surfaces touched + evidence parity, not by the number of repos or lines. A diff outside the transform → the exemption is lost.
- **`factory.generate`:** generated files that match the template byte for byte at the pinned version → not counted as an escape hatch (FE-CFG-001); risk follows the surface created.
- An agent may propose **raising** the risk, never lowering it. The path-risk map lives in the registry, not in the profile.
- **Control files** ([§8.7](08-verification.md#s8-7)). Loosening → R3. Tightening or neutral → at least R2, with an independent code owner of the file. The output of a registered **trusted mechanism** (baseline-shrink bot, activation snapshot, generator output matching the template byte for byte) keeps the PR's risk and may merge without a human, provided the PR contains nothing else (Standard §7.13). A change that cannot be classified counts as loosening.
- An **open security finding** raises the PR to at least R2 until it is fixed or dismissed ([§9.9](09-agent-operating-model.md#s9-9)).
- An **unknown** surface, owner or reversibility means the worst case (R3, LOW).

<a id="s10-2"></a>

## 10.2. Reversibility

| Level | Meaning |
|---|---|
| **HIGH** | A single revert + redeploy through the standard pipeline fully recovers (including indexed public pages, if the pipeline revalidates the cache itself) |
| **MEDIUM** | Needs manual coordination, but a redeploy can still erase the impact (a package published externally, removing a URL from the search index) |
| **LOW** | A redeploy cannot erase the impact: email/notifications already sent, data deleted/overwritten, migrations, published prices/terms/legal commitments, transactions |

Prices/terms are identified through **declared routes and content types**, never through content regex.

<a id="s10-3"></a>

## 10.3. Trust

Trust levels T0–T2, the minimum evidence, the five components of trust evaluation, the low-volume rule and the version change rule belong to the Standard (Standard §9.4, §9.9). The profile uses the Standard's *suggested* thresholds as mandatory thresholds, records them in the trust registry for each task class, and adds only the following:

- **T2 counts only merges through A1.** Tasks counted toward T2 MUST be merged to production through A1, not A0, so that T2 evidence always includes a human who read the change.
- **Difficulty tiers:** each class declares its tiers (for example for `bug-fix` on `operational`: one file in one module; multiple files in one surface; touching a `critical` journey or LRO state). The task-count threshold applies to each tier, not only to the total.
- **Critical scenarios:** taken from the `critical` journeys of the in-scope surfaces and from the archetype's seeded defects ([§12](12-evaluation.md#s12)).
- **Holdout:** held by the Agent/Eval engineer, separate from the playbook authors, rotated every quarter.
- **Regression:** the regression eval ([§12](12-evaluation.md#s12)) also runs when the risk classifier changes, in addition to the model, playbook and skill changes that the Standard requires.
- **Consequence of the confidence bound:** 50 tasks, 0 failures → upper bound ≈ 6% (rule of three), sufficient only for R0/R1. Therefore trust is meaningful only for R0/R1 in this design.
- **An incident attributed to a class → back to T0** (stricter than the Standard's one-level downgrade, [§18.4](18-aisf-conformance.md#s18-4)).

<a id="s10-4"></a>

## 10.4. Autonomy levels and matrix

Autonomy levels, approval slots, the lightest-level matrix and the zero-human conditions belong to the Standard (Standard §9.5, §9.10). In this profile, the Standard's "area" is the **surface**, and "affected surface" includes directly modified surfaces and indirectly affected surfaces ([§10.1](#s10-1)). The table below lists only what the profile specifies or tightens:

| Level | Profile specification or tightening |
|---|---|
| **A1** | The requester's sign-off (or the automation owner's, [§10.5](#s10-5)) fills the slot only when all four conditions hold: sub-tier R0 or R1; the requester belongs to the owning team of **every affected surface, direct and indirect**; no circuit breaker of those surfaces or of the task class is tripped ([§10.6](#s10-6)); no `blocking` integrity sensor hit. Otherwise, the slot is filled by an independent code owner, with scope per the paragraph "Indirectly affected surfaces" below |
| **A2** | The independent slot is always filled by a code owner: at least one independent code owner of each directly modified surface, plus the shared asset owner's slot when there are indirectly affected surfaces. The Standard requires a code owner only for sensitive areas ([§18.4](18-aisf-conformance.md#s18-4)). Every A2 slot comes with an independent verification run ([§9.10](09-agent-operating-model.md#s9-10)) |
| **A3** | Of the two independent slots, at least one is an independent code owner of each directly modified surface, plus the shared asset owner's slot when there are indirectly affected surfaces |

| Sub-tier \ Reversibility | HIGH | MEDIUM | LOW |
|---|---|---|---|
| **R0** (Low) | T2 + zero-human conditions → A0 · otherwise → A1 | T2 + zero-human conditions → A0 · otherwise → A1 | A2 |
| **R1** (Low) | T2 + zero-human conditions → A0 · otherwise → A1 | T2 → A1 · otherwise → A2 | A3 |
| **R2** (Medium) | A2 | A2 | A3 |
| **R3** (High) | A3 | A3 | A3 |

The cells stricter than the Standard's matrix are listed in [§18.4](18-aisf-conformance.md#s18-4).

**Indirectly affected surfaces (declared deviation DEV-FE-01, [§18.1](18-aisf-conformance.md#s18-1)).** The Standard requires a code owner slot for *each affected area* at A1 (when the requester's sign-off is not counted) and at A3. A shared asset change (package, token, contract or content model) can have dozens of indirect consumers, so the profile satisfies the "each affected surface" requirement as follows instead of one code owner slot per consumer:

1. Each directly modified surface has a slot filled by an independent code owner.
2. Each indirectly affected surface is covered by three components at once: consumer verification evidence on that surface itself ([§11.2](11-delivery-change-management.md#s11-2)); the slot of the shared asset owner (DS owner, Platform owner, contract or content model owner), who is accountable for the impact on consumers; and visual baseline approval by the consumer's code owner, only when that surface has a visual diff (FE-TEST-002).
3. The owner of every indirect consumer is informed (I) and may block with a finding through the standard channel; `x:privileged-admin` or `x:security-sensitive` consumers are always informed.
4. Risk is still computed over every affected surface ([§10.1](#s10-1)), and this coverage never applies to the requester's sign-off at A1 (conditions in the table above).

Until the Standard's owner approves the ADR for this deviation (PC-4), the Standard applies verbatim: at A1 there is no valid requester sign-off, and at A3 each indirectly affected surface needs an additional slot from a code owner of that surface itself. This rule is also an upstream proposal ([§18.5](18-aisf-conformance.md#s18-5)).

**Additional slots by domain** (added by union, Standard §9.10): security owner for `x:security-sensitive` and privileged paths of `x:privileged-admin`; DS owner for the public DS API; business owner for prices, terms and legal content.

**`propose-only` by protected group (Standard §9.4 item 5).** For a group with `propose-only`, the agent may only propose (analysis, plan, patch attached to the task); a human writes the change and opens the PR, and the trust level does not change that. This table is a control file in the policy repo; changing "Yes" to "No" is loosening (R3).

| Protected group | `propose-only` | Applied to frontend |
|---|---|---|
| `auth-and-secrets` | **Yes** (Standard default) | `@platform/auth`, server-side session and authorization, step-up auth, token and secret handling, privileged paths concerning sessions or authorization |
| `ci-release-and-branch-protection` | **Yes** | Policy repo, CI files, merge queue and branch protection configuration |
| `production-operations` | **Yes** | Production and data plane operations; the Change Shepherd acts only within the limits of its tools ([§11.7](11-delivery-change-management.md#s11-7)) |
| `legal-and-privacy` | **Yes** | Terms text, privacy policy and content with legal obligations |
| `control-file` | **Yes** when loosening; No when tightening or neutral | Every file in the control file set ([§8.7](08-verification.md#s8-7)) |
| `billing-and-payments` | No | A3 by tier; `pay.*` and a separate security pass ([§9.9](09-agent-operating-model.md#s9-9)) |
| `destructive-or-regulated-data` | No | An `irreversible` step needs an explicit human instruction ([§9.12](09-agent-operating-model.md#s9-12)) |
| `public-api-breaking`, `public-api-additive` | No | `design-review`, consumer verification and codemod ([§11.2](11-delivery-change-management.md#s11-2)) |

**Zero-human conditions (A0).** The Standard's six conditions (Standard §9.5 item 11) apply, with the following specifications:

1. Only **R0 with HIGH or MEDIUM reversibility, or R1 with HIGH reversibility** (matching the matrix above); eligibility is assessed over every affected surface, direct and indirect.
2. Opt-in is the surface's `autonomy.zeroHuman: true` in `factory.yaml` (a control file).
3. `blocking` AI review is the surface's `review.aiMode: blocking` ([§9.9](09-agent-operating-model.md#s9-9)).
4. Any finding dismissal on an A0 candidate moves the change to A1 ([§10.5](#s10-5)).
5. Breakers per [§10.6](#s10-6); error budget per [§19.4](19-nfr-enterprise.md#s19-4).

**A0 candidates:** `docs.link-fix`, `docs.typo`, `deps.patch-bump` (profile passes, no visual diff, Renovate owned by the vertical), `web.copy-change` (does not touch SEO, analytics, prices or terms), and `gardening.lint-autofix` run by the vertical.

**Not A0:** `codemod.apply` opened by platform automation in a vertical's repo. The automation owner is not in the vertical's owning team, so a code owner of the vertical fills the A1 slot: one approval, with evidence parity.

**Never A0 in this profile:** R1 with MEDIUM reversibility, every R2 or R3 change, every control file change not coming from a trusted mechanism, and PRs created by an agent on a developer machine (Standard §6.15, [§8.7](08-verification.md#s8-7) item 6).

<a id="s10-5"></a>

## 10.5. Separation of duties (SoD) (Standard §9.10)

The SoD table and the definition of "independent" belong to the Standard. The profile implements and adds:

- **Source of requester identity:** the task runner records it from the triggering event: the CI dispatch actor, the author of the issue or command, the CLI login, or, for automation, its accountable owner. The agent cannot write this field; the `Requested-by:` trailer is for display only.
- **`policy.sod`** is a required check in the mandatory pipeline, implementing the Standard's SoD check on the `PolicyDecision` for the reviewed SHA. It reads breaker state from the trust registry to apply the A1 conditions ([§10.4](#s10-4)). The Git platform's settings ("require approval of the most recent push", "require review from Code Owners") are also enabled, but are not sufficient by themselves.
- **Backup owner.** `factory doctor --owners` reports every surface or control file lacking a backup. If the sole owner is the requester, the backup approves; with no backup, the change is escalated to the vertical's tech lead or the Frontend Platform Owner.
- **Waiver approver** (Standard §7.5). Product check: the vertical's tech lead. DS check: DS owner. `baselineable: false` check: only the security owner. Baseline waivers and every waiver longer than 90 days: Frontend Platform Owner. The requester never approves a waiver on a `BLOCKER` check.
- **Finding dismissal.** For an A0 candidate, any dismissal moves the change to A1. A security finding is dismissed by the security owner or an independent person.

<a id="s10-6"></a>

## 10.6. Circuit breakers (Standard §9.9)

Breakers, automatic reactions, minimum sample size, early indicators and model pinning belong to the Standard. Breakers are tracked per surface and per task class. The profile specifies:

| Signal | Reaction in this profile |
|---|---|
| Any revert of an A0 merge | Absolute trigger for low volume: pause A0 on the surface until a human reviews |
| Early indicators: hotfix within 48 h, failing post-merge check, Change Shepherd pause | Counted in the rate; two within the window pause A0 |
| Escaped defect attributed to a task class | The class's trust goes to T0 ([§10.3](#s10-3)) |
| The surface's error budget is exhausted | Per the error budget policy ([§19.4](19-nfr-enterprise.md#s19-4)) |

**While a breaker is tripped, the requester's sign-off is never counted.** When any breaker of an affected surface or of the task class is tripped, A0 is closed, and R0/R1 changes go through A1 with the slot filled by an independent code owner ([§10.4](#s10-4)); `policy.sod` rejects the requester's sign-off for that slot.

A breaker may be loosened only by a decision of the surface owner and the Frontend Platform Owner, with evidence.

---

[Contents](README.md) · [← 9. Agent Operating Model](09-agent-operating-model.md) · [11. Delivery & Change Management →](11-delivery-change-management.md)
