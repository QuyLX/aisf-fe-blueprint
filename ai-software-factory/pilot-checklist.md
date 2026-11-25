# Pilot checklist — first 6–8 weeks

A condensed version of §15.0 and the P0 items in §16 for the pilot team; the loosenings granted to pilots are in §20.8. Teams without a domain profile use the Core Profile (§20.7). **Do only what is on this page.** The rest of the Standard describes the target system and is brought in when the pilot measures a need; do not build the factory before you have a workshop [R35].

## Single objective

Run this flow on real issues in **one** repository:

```text
issue → Bug Fix workflow → checkpoint and approval → independent verification → evidence → knowledge reconciliation
```

## Before starting (Phase 0)

- [ ] Pilot Charter filled in: repository, owner, task class, duration, staffing, stop criteria ([Appendix H](appendices/H-pilot-charter.md))
- [ ] Each in-scope area has at least two qualified owners or one named backup (§9.10)
- [ ] Baseline measured: lead time, review wait time, revert rate, human attention minutes per work item (§13.4, §13.6)
- [ ] Data governance determined for the model providers and tools to be used (§10.2)
- [ ] Git/CI platform and agent runtime settled, along with how each control is implemented on that platform (§18 items 1–3, §14.4)
- [ ] The pilot area has at least three people qualified to review, or a backup reviewer from another team (§9.10)
- [ ] Cost ceiling for the entire pilot and a default per task (§6.4)
- [ ] Schedule allows enough time to run ≥ 15–20 real bug fixes before the Exit evaluation

## To do

**Repository**
- [ ] Concise `AGENTS.md` and owner-written `REVIEW.md` (§5.2, Appendix B, F)
- [ ] PR template with Testing, Risk, How to revert, Checkpoint, Knowledge impact (Appendix G)
- [ ] 3–5 high-value rules, each with passing and violating fixtures (§8.2)

**No change can bypass controls**
- [ ] CODEOWNERS and branch protection; control files protected (§7.13)
- [ ] Required checks run from a central definition that a PR cannot modify, through a verification adapter (§7.15 items 1–2, §7.16)
- [ ] Diff-scanning integrity sensor, pilot mode: `warn`, except skip/focus and control files at `blocking` (§7.13)
- [ ] Separation of duties check with authenticated requester identity (§9.10)
- [ ] Deterministic path → tier mapping table (`riskMap` in `factory.yaml`), a control file with an owner like CODEOWNERS; undeclared paths are High; authors do not choose their own tier (§20.8 item 4)
- [ ] Canonical vocabulary and schemas checked by CI (Appendix J, K)

**Workflow**
- [ ] Bug Fix workflow spec `ACTIVE` following the minimal pilot version ([Appendix C.4](appendices/C-task-workflow-spec.md))
- [ ] Checkpoints `requirement-review`, `independent-verification`, `implementation-review` derived from issue/PR events, not filled in by hand (§6.5, §6.12)
- [ ] Task state derived from labels and events; sub-states derived or omitted (§20.8 item 4)
- [ ] Learning step in compact form: the "Review findings" and "Knowledge impact" sections of the PR are filled in on every pilot PR (§6.6 step 16)
- [ ] Low/Medium bug fixes take no more than two human touches outside exceptions (§13.6)
- [ ] PRs from agents used on developer machines go through exactly the same gates as every PR (§6.15)
- [ ] Reproduction test written by the verifier, read-only for the implementer (§7.13)

**Knowledge**
- [ ] Lifecycle frontmatter for the 3–5 artifacts the pilot actually uses (§5.8, Appendix E)
- [ ] `knowledge.impact` check at `warn`, based on `describes` globs; reconciliation decisions confirmed in the PR by a person belonging to the authority (§5.9–5.10)

**Factory operations**
- [ ] Degraded mode and break-glass runbook, drilled once (§12.7)
- [ ] Weekly pilot review (§17.2)

## Exit

- [ ] The flow above runs on ≥ 15–20 real bug fixes, compared against the baseline: lead time, attention minutes, number of people touching, escaped defects
- [ ] At least one change touched a described artifact and was reconciled
- [ ] No gate was bypassed
- [ ] Pilot conformance claim (`mode: pilot`): L1 plus the list of CORE-IDs in §20.8 item 3 (a full L2 claim is the goal of Phase 1–2)
- [ ] Decision on what to add next, based on workflow metrics (§13.7)

## Not yet in the pilot

Custom workflow engine · knowledge graph or graph database · central Knowledge Registry · dashboard · central catalog · custom-built multi-agent AI review · specialized agents · zero-human merge · trust evaluator · circuit breaker automation · attestation signing (unless the profile requires it) · production signals automatically creating tasks · workflows other than Bug Fix · separately recorded sub-states, handoff records and recovery records as artifacts · merge queue · full Check Registry · automated tier classifier.

## What to read

| Role in the pilot | Read |
|---|---|
| Pilot owner | This page, §15.0, §20.8, Appendix H, Appendix I |
| Harness builder | §6.7, §7.13, §7.15–7.16, Appendix C, J, K |
| Code owner / reviewer | §6.12, §7.6, §9.10, Appendix F, G |
