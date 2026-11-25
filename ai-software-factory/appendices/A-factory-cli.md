# Appendix A – Suggested Factory CLI

The CLI is the initial developer interface, not a commitment to any particular command implementation.

```bash
factory init                 # register or initialize repository harness
factory doctor               # validate harness, tools, docs, permissions, control-file coverage, conformance level
factory catalog              # discover products, repositories, domains, and dependencies
factory task                 # create a task from a goal/change brief
factory task status <id>     # inspect task state, sub-state, basis, checkpoints and handoffs
factory workflow validate    # validate Task Workflow Specifications against the schema (Appendix C)
factory workflow list        # list workflows with status and task classes
factory verify               # run the selected verification profile
factory review               # run independent review and evidence checks
factory eval                 # evaluate agent/skill/playbook changes
factory docs check           # validate documentation structure, links and artifact metadata
factory knowledge impact     # list knowledge artifacts affected by the current change (§5.9)
factory knowledge status     # show artifacts in NEEDS_REVALIDATION and open conflicts
factory rules check          # run rule catalog validation
factory evidence show <id>   # inspect evidence bundle for a task/commit
```

The CLI should provide a small number of golden paths. Avoid creating one command per internal component.
