# Appendix B – Minimal `AGENTS.md` example

```markdown
# Repository Guide

## Purpose
One-paragraph description of this repository and its product role.

## Golden commands
- Install:
- Dev:
- Build:
- Lint:
- Type check:
- Test:

## Architecture map
- Current state: docs/architecture/current-state.md
- Boundaries: docs/architecture/module-boundaries.md
- Domain glossary: docs/product/domain-glossary.md
- ADR index: docs/architecture/decisions/README.md

## Engineering rules
- Rule catalog: engineering/rules/README.md
- Verification profile: engineering/verification/profile.yaml
- Review guide: REVIEW.md

## Working agreements
- Follow the ACTIVE workflow for the task class (.factory/playbooks/); add steps if needed, never skip mandatory steps or checkpoints.
- Keep changes small and update the plan for complex work.
- Do not add dependencies without checking the policy.
- Run the required verification profile before claiming completion.
- Report `NOT_ASSESSED` explicitly; never claim an unrun check passed.
- Show command output as evidence; never state that work is done without it.
- Never weaken tests, rules, thresholds, or CI configuration to make a check pass; propose such changes separately.
- Use only ACTIVE knowledge artifacts as your basis; report the status of anything else you rely on.
- If code and an ACTIVE spec disagree, record a knowledge conflict and stop at the next gate; never edit a spec, acceptance criterion, or test oracle to match the code.
- List affected knowledge artifacts and a reconciliation decision for each before packaging the PR.

## Approval boundaries
List protected paths, public APIs, migrations, security-sensitive changes, and production actions that require approval.
```
