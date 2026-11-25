# Appendix G – PR template

```markdown
## Intent
One line. Links: issue, Change Brief / spec, acceptance criteria IDs.

## Risk
Tier: Low | Medium | High (profile sub-tier: ) — reason:
Autonomy level (§9.5): A0 | A1 | A2 | A3
Protected paths touched: yes / no
Needs human judgment on: (trade-offs, deviations, open questions)
What the automated checks would NOT catch:

## Testing
Commands run and their results (output or evidence-bundle link).
Non-PASS checks and their dispositions (§7.5).

## How to revert
Flag / revert commit / data-recovery steps. Irreversible parts:

## Review findings
Open: n · Resolved: n (fixed n, closed with reason n)

## Workflow and checkpoints
Workflow: <id>@<version> · final sub-state:
Checkpoints (§6.12): requirement-review ✔ by … · plan-review n/a (policy) · implementation-review … · independent-verification … · release-readiness …
Basis changes during the task (§6.5): none | <artifact> → decision (continue/replan/reapprove/stop) by …

## Knowledge impact
Affected artifacts (§5.9) and reconciliation decision (§5.10):
- <id>@<version> (authority) — keep | update | supersede | deprecate | review
Knowledge conflicts recorded (§4.7): none | <link>

## Accountability
Requested by:            Agent / playbook / model snapshot:
Agent assistance (optional; metrics only, §6.15): none | local interactive | Factory agent
Cost vs. task budget (§6.4):
Independent approval required (§9.10): yes / no
Control files touched (§7.13): yes / no
[ ] I have read the evidence and I own this change.
```
