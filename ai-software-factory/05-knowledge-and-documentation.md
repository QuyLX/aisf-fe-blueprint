# 5. Engineering knowledge and documentation governance

## 5.1. Distributed knowledge, centralized governance

1. Knowledge SHOULD live where the people who own it can maintain it, according to the following table.

| Scope | Authoritative content | Location |
|---|---|---|
| Organization | Shared engineering standards, security policies, global templates, shared skills and playbooks | Central Engineering repository |
| Product/system | Product purpose, domain glossary, current architecture, ADR index, service contracts, runbooks | Product/system repository |
| Repository | Stack, local commands, module boundaries, local rules, verification profile | Repository root and `docs/` |
| Feature/module | Behavior, edge cases, local test strategy, implementation decisions | Near the feature/module or in a linked spec |
| Cross-repo catalog | Index and relationships, not duplicated descriptions | Control Plane registry |

**Rationale.** Knowledge is placed close to the people who own and maintain it; the centrally governed part is only the index and relationships. The cross-repo catalog does not duplicate descriptions because a centralized copy of the content would quickly drift from the original in the repository.

## 5.2. `AGENTS.md` is a map, not an encyclopedia

1. The root instruction file SHOULD be short, stable and operational.
2. The root instruction file SHOULD answer the following questions:
   - What is this repository?
   - What are the most important constraints?
   - Which commands are used to build, test, lint and run?
   - Where is the architecture map?
   - Where are the rules and verification profile?
   - How should an agent find domain-specific guidance?
   - Which actions require approval?
3. Detailed knowledge SHOULD live in separate documents and skills and be loaded only when relevant.
4. Each governed repository MUST have a root agent instruction file (for example `AGENTS.md`) stating at minimum: a repository map, build, test and lint commands, the most important constraints, and the actions that require approval.

**Rationale.** The root instruction file acts as a map: it points to where knowledge can be found rather than containing all of it. OpenAI's harness article describes this progressive disclosure pattern along with mechanical checks on documentation freshness [R01]. Similarly, Anthropic distinguishes always-loaded project context from skills, hooks, plugins and MCP integrations loaded on demand [R03, R04].

### 5.2.1. `REVIEW.md`: owner-written review guidance

**`AGENTS.md`** (or the equivalent agent instruction file of the runtime in use, for example `CLAUDE.md`) is the file that tells agents how to work in the repository. **`REVIEW.md`** is a separate file, written by code owners, that tells reviewers — both human and AI — what matters when evaluating a change in a given area.

1. `REVIEW.md` MAY be placed at the repository root and in sub-areas owned by different teams.
2. Conformance with `REVIEW.md` MUST be checked as a separate review concern.
3. `REVIEW.md` SHOULD be short and owned by the code owners of that area.
4. `REVIEW.md` SHOULD define what counts as a bug in that area.
5. `REVIEW.md` SHOULD cap the number of nits and list paths to skip (generated code, vendored files, fixtures).
6. `REVIEW.md` SHOULD require file-and-line evidence for every finding, not inference from names.
7. `REVIEW.md` SHOULD require the summary to open with the number of findings and to state explicitly when there is nothing to fix.
8. Review agents SHOULD flag changes that make a line in `AGENTS.md` or `REVIEW.md` stale.
9. Entry-point files SHOULD be short and point to other documents.
10. Recurring review comments SHOULD be moved into `REVIEW.md` first, then into lint rules, architectural constraints or skills when they can be detected mechanically (§8.5). See Appendix F.
11. Each governed area with AI review enabled (§7.9) MUST have review guidance written by the code owners of the area, in `REVIEW.md` or an equivalent file readable by the AI reviewer in use.

**Rationale.** Separating `REVIEW.md` from `AGENTS.md` distinguishes two different questions: "how to work in this repository" and "what matters when evaluating a change in this area" [R29]. The entry-point file is kept short because it is loaded on every review, so every superfluous line costs context on every turn.

**Example.** How to define "what counts as a bug" in each area: documentation, "it says something wrong"; configuration, "it routes incorrectly"; billing, "it calculates the wrong amount".

## 5.3. Recommended repository knowledge structure

1. A repository SHOULD create only those knowledge directories that have a real owner and a use case.

**Rationale.** The structure below is a target pattern, not a list of mandatory directories. Empty or unowned directories only create the false impression that the knowledge exists.

**Example.**

```text
repository/
├── AGENTS.md
├── REVIEW.md
├── README.md
├── factory.yaml
├── catalog/
│   ├── repository.yaml
│   ├── domains/
│   ├── features/
│   └── dependencies/
├── specs/
│   ├── features/
│   └── changes/
├── contracts/
│   ├── openapi/
│   ├── events/
│   └── consumer-tests/
├── docs/
│   ├── index.md
│   ├── product/
│   │   ├── vision.md
│   │   └── domain-glossary.md
│   ├── architecture/
│   │   ├── system-context.md
│   │   ├── current-state.md
│   │   ├── module-boundaries.md
│   │   ├── dependency-map.md
│   │   ├── decisions/
│   │   └── proposals/
│   ├── development/          # Diátaxis structure [R25]
│   │   ├── tutorials/
│   │   ├── how-to/
│   │   ├── explanations/
│   │   └── reference/
│   └── operations/
│       ├── deployment.md
│       ├── rollback.md
│       └── runbooks/
├── engineering/
│   ├── rules/
│   ├── verification/
│   └── policies/
├── .factory/
│   ├── skills/
│   ├── playbooks/
│   └── evaluations/
├── src/
└── tests/
```

## 5.4. Living documentation and decision records

**Living documentation** is documentation that describes the current state, including:

- Current architecture.
- Module/domain boundaries.
- API and event reference documentation.
- Development guides.
- Runbooks.
- Policies and standards currently in effect.

An **RFC/Proposal** is a document used for discussion before a decision. An **ADR** preserves the history of a decision; it records an accepted or rejected decision. **Current-state documentation** describes what is actually true at this point in time.

1. Living documentation MUST be updated when behavior or architecture changes.
2. An ADR SHOULD include the following content:
   - Context and problem.
   - Constraints and assumptions.
   - Options considered.
   - Decision and rationale.
   - Accepted trade-offs.
   - Consequences and migration.
   - Conditions that trigger reconsideration.
   - Superseding ADR, if any.
3. Old ADRs MUST NOT be rewritten to make history look consistent.
4. When a decision changes, the Factory MUST create a new ADR and move the old ADR to `SUPERSEDED` (§5.8).
5. All living documents, ADRs, RFCs/proposals and specs are knowledge artifacts and MUST follow the lifecycle in §5.8.

**Rationale.** RFCs/proposals, ADRs and current-state documentation answer three different questions. Rewriting an old ADR for "consistency" erases the reason why the system was once designed that way; creating a new ADR and moving the old ADR to `SUPERSEDED` preserves both the current decision and the history. On ADRs as records of decision history, see [R27]; arc42 is a usable template for living architecture documentation [R26].

## 5.5. Context hierarchy

The **context hierarchy** has five levels:

- **Level 1:** Global policy and concise root instructions.
- **Level 2:** Repository context: stack, architecture, standards, glossary, commands.
- **Level 3:** Task context: spec, affected modules, contracts, test guidance.
- **Level 4:** Relevant decisions: ADRs in state `ACTIVE` and significant history.
- **Level 5:** Execution context: source code, tests, configuration, CI, dependency graph, runtime output.

1. The Factory SHOULD NOT load all five levels into every task.
2. The Factory SHOULD resolve context progressively based on changed files, task type, metadata and risk.
3. The context resolver MUST return the state of every knowledge artifact along with it.
4. The context resolver MUST NOT return `SUPERSEDED`, `DEPRECATED` or `ARCHIVED` artifacts as current context, unless the task requires historical lookup.

**Rationale.** Loading all five levels into every task dilutes context and increases cost; progressive resolution lets the agent see only what is relevant. The state accompanies each knowledge artifact so that the agent does not inadvertently rely on a document that has been superseded or is no longer in effect.

## 5.6. Quality gates for documentation and gardening

1. Automated checks MAY verify:
   - Broken links and orphaned documents.
   - Mandatory frontmatter/metadata.
   - Knowledge artifact lifecycle metadata (§5.8): valid state, owner still exists, bidirectional `supersedes`/`supersededBy` links.
   - Artifacts in `NEEDS_REVALIDATION` past the handling deadline declared by the owner.
   - Feature/spec IDs referenced by tests and evidence.
   - Ownership and last review date of important documents.
   - Updates to API contract documentation when public interfaces change.
   - Whether current architecture documentation conflicts with the discovered module structure.
2. A scheduled documentation-gardening workflow SHOULD propose focused updates.
3. Proposals from the documentation-gardening workflow MUST go through the reconciliation process (§5.10).
4. The documentation-gardening workflow MUST NOT silently rewrite significant architectural decisions.
5. Changes to product purpose, policy or accepted trade-offs MUST have human review.

**Rationale.** The checks in item 1 are mechanical, cheap and deterministic. The part requiring judgment (content updates) is left to the documentation-gardening workflow to propose and goes through reconciliation, because an automated proposal can be correct in wording but wrong in intent. That is also why significant architectural decisions are not silently rewritten, and why changes to product purpose, policy or accepted trade-offs require human review.

## 5.7. On-demand explainers and context sharing

An **on-demand explainer** is an explanation generated by an agent from the current code, with file/line references, for exactly the slice that an engineer is about to change. An explainer is a single-use work product.

1. The Factory SHOULD supplement living documentation with on-demand explainers.
2. Explainers MUST NOT replace living documentation.
3. Explainers MUST NOT be used as a source of truth unless they have been reviewed and promoted to living documentation.
4. An explainer that conflicts with current-state documentation MUST produce a documentation-gardening finding.
5. An explainer that conflicts with a spec MUST be handled as a knowledge conflict (§4.7).
6. Engineering channels for agent work SHOULD be public by default.
7. New engineers SHOULD use an agent together with the documentation index as their first onboarding interface.

**Rationale.** When agents write most of the code, fewer engineers hold a complete mental model of the system, and code review becomes a less effective channel for spreading knowledge. On-demand explainers compensate for this: engineers ask an agent to explain exactly the part they are about to change, for example "how are managed settings retrieved?", and receive an answer generated from the current code with file/line references [R29]. An explainer has not been reviewed by anyone, so it only supplements and does not replace living documentation.

Engineering channels that are public by default for agent work make in-progress changes visible while they are being built [R29]. Using an agent together with the documentation index as the first onboarding interface for new engineers is described in [R28].

## 5.8. Knowledge artifact lifecycle

A **knowledge artifact** is a versioned artifact that humans or agents use as a basis for decisions: specs and change specs, design and architecture documents, ADRs, RFCs/proposals, living documentation (API reference, development guides), runbooks, agent instruction files (`AGENTS.md`, `REVIEW.md`), skills and playbooks (§6.7). Code, tests, rules, checks and tasks are not knowledge artifacts; they have their own lifecycles but still participate in the dependency graph in §5.9.

**Mandatory metadata.**

1. Every governed knowledge artifact MUST have metadata, placed in frontmatter or a sidecar file, including: `id`, `kind`, `status`, `version`, `owner`, `authority`, `describes`, `basedOn`, `supersedes`/`supersededBy` where applicable, and `validatedAgainst` (the commit, contract version or artifact version against which the content was confirmed correct). The format is in Appendix E; enumerated values are in the vocabulary (Appendix J).
2. The `authority` field MUST take one of the following values.

| `authority` | Question the artifact answers | Example |
|---|---|---|
| `intent` | What the system must do | Spec, acceptance criteria, product decision |
| `decision` | Why this option was chosen | ADR |
| `implementation-description` | How the system is currently built | Current architecture documentation, API reference, development guide |
| `procedure` | Which steps to follow to do this | Runbook, playbook |
| `guidance` | How agents and reviewers should work | `AGENTS.md`, `REVIEW.md`, skill |

**States.**

3. Knowledge artifacts MUST use the states and transitions in the following table.

| State | Meaning | Usable as a basis | Valid transitions |
|---|---|---|---|
| `DRAFT` | Being drafted | No | `PROPOSED`, `ARCHIVED` |
| `PROPOSED` | Awaiting the corresponding review checkpoint (§6.12) | For discussion only; not a basis for implementation | `ACTIVE`, `DRAFT`, `ARCHIVED` (rejected, with reason recorded) |
| `ACTIVE` | Approved and currently in effect | Yes | `NEEDS_REVALIDATION`, `SUPERSEDED`, `DEPRECATED` |
| `NEEDS_REVALIDATION` | A dependency has changed, drift has been detected, or the owner is gone; validity is uncertain | Conditionally: tasks using it handle it per §6.5 (basis change). MUST NOT be a basis for new tasks at tier Medium or High until revalidated | `ACTIVE` (revalidated), `SUPERSEDED`, `DEPRECATED` |
| `SUPERSEDED` | Replaced by another artifact; `supersededBy` MUST be present | Not for new work; kept for history | `ARCHIVED` |
| `DEPRECATED` | No longer in effect and no direct replacement artifact; the reason MUST be recorded | Not for new work | `ARCHIVED` |
| `ARCHIVED` | Frozen, read-only, for historical lookup only | No | — |

**Only `ACTIVE` artifacts are the default basis.**

4. The Factory and agents MUST use only `ACTIVE` artifacts as the default basis.
5. `NEEDS_REVALIDATION` artifacts MUST be used only under the conditions in the table above.
6. A task MUST record the artifacts it relies on in `basis` (§6.4).

**Who changes state.**

7. A transition to `ACTIVE`, `SUPERSEDED` or `DEPRECATED` MUST be a decision of the corresponding authority (§5.10) through a review checkpoint (§6.12).
8. Automation and agents MUST only transition `ACTIVE → NEEDS_REVALIDATION` (flagging a suspicion) and propose other transitions.
9. Automation and agents MUST NOT revalidate, supersede or deprecate an artifact on their own.

**Validity is not inferred from time.**

10. The Factory MUST NOT conclude that an artifact is still or no longer in effect based solely on `last_modified`, document age or number of edits.
11. Artifact validity MUST be determined from the state, `validatedAgainst`, and the absence of any unhandled revalidation events.
12. Artifact age MUST be used only to trigger periodic review for important artifacts.

**Edit in place or replace.**

13. An update that clarifies or corrects a wrong description without changing meaning MUST increment the `version` of the same `id`.
14. A change of intent or decision MUST create a new artifact that `supersedes` the old artifact.
15. The decision content of an `ACTIVE` ADR MUST NOT be modified; only state metadata and links MAY be updated.

**Immutable history.**

16. `SUPERSEDED`, `DEPRECATED` and `ARCHIVED` artifacts MUST NOT be modified other than link metadata.
17. Every version referenced by a completed task MUST remain retrievable (§6.5).

**Every `ACTIVE` artifact has an owner.**

18. Every `ACTIVE` artifact MUST have an owner.
19. When the owner is removed without a replacement, the artifact MUST move to `NEEDS_REVALIDATION`.

**Distinction from other lifecycles.**

20. Knowledge artifact state is a separate vocabulary; the Factory MUST NOT use it for the objects in the following table, and MUST NOT use the lifecycles of those objects for knowledge artifacts.

| Object | Lifecycle | Defined in | Relationship to knowledge artifacts |
|---|---|---|---|
| Knowledge artifact | `DRAFT` … `ARCHIVED` | §5.8 | — |
| Task | `REQUESTED` … `COMPLETED`, plus workflow sub-states | §6.5 | Tasks reference artifacts through `basis`; an artifact state change triggers basis evaluation (§6.5) |
| Rule | `PROPOSED` … `RETIRE` | §8.3 | Impact analysis may require a rule to move to `REVISE`, without using artifact states |
| Check | `planned`, `shadow`, `active`, `deprecated` | §8.7 | A `required` check needs a basis that is an `ACTIVE` artifact (§8.7 rule 2) |
| Dependency | `APPROVED`, `CONDITIONAL`, `DEPRECATED`, `PROHIBITED` | §11.4 | Independent |
| Test | No lifecycle of its own; it is code | §7 | A test verifying an acceptance criterion is flagged for review when the artifact containing that criterion changes |

21. A playbook is both a knowledge artifact (its state indicates whether the workflow may be used) and a control file (§7.13); playbooks MUST follow both sets of rules.

**Rationale.** This lifecycle answers a question that ordinary documentation cannot: whether this document can still be used as a basis. Without an explicit state, agents treat every document they find as current, including superseded ADRs or abandoned specs.

- Automation and agents are allowed to flag suspicion (`ACTIVE → NEEDS_REVALIDATION`) because doing so is safe: it only lowers the level of trust and does not create new validity.
- Validity is not inferred from time because a document that has not been edited in a long time can still be entirely correct, and a freshly edited document can still be wrong. Age is only a signal to trigger periodic review.
- Knowledge artifact state is kept separate from the lifecycles of tasks, rules, checks, dependencies and tests to avoid confusion when the same word (for example `DEPRECATED`, `PROPOSED`) appears in several lifecycles with different meanings.

## 5.9. Dependencies between knowledge artifacts and impact analysis

1. The Factory MUST be able to determine, for each change, which knowledge artifacts may be affected.

**Relationships.**

2. The knowledge dependency graph MUST use the relationships of §4.3 according to the following table.

| Relationship | From → to | Meaning when the "to" side changes |
|---|---|---|
| `describes` | Artifact → path, module, service, contract, entity | The artifact may no longer describe it correctly |
| `basedOn` | Artifact → artifact or contract version | The artifact's validity MUST be re-evaluated |
| `generatedFrom` | Artifact → approved source | The artifact is mechanically regenerated |
| `verifiedBy` | Acceptance criterion → test or check | The test SHOULD be reviewed when the criterion changes |
| `implements` | Code → spec, via requirement IDs in the task record | The code SHOULD be checked when the spec changes |
| `supersedes` | New artifact → old artifact | Artifacts `basedOn` the old artifact MUST be re-evaluated |

3. `describes` SHOULD be as narrow as possible (one module, one contract, one group of paths).
4. Repository-level instruction files such as `AGENTS.md` SHOULD declare `describes` for what they actually describe (golden commands, directory structure), not `**`.

**Relationship sources.** Relationships follow the three metadata sources of §4.4: *declared* in frontmatter (`describes`, `basedOn`); *discovered* from links, requirement IDs in tests and contract references; *observed* from task records (the task's `basis` and the files the task changed).

5. Discovered relationships MUST NOT silently overwrite declared relationships.
6. Differences between discovered and declared relationships MUST be reported as drift.

**Dependency index.**

7. Repository CI MUST generate a lightweight dependency index (for example `knowledge-index.json`, Appendix E) from metadata and links on the main branch, stored as a versioned artifact.
8. The dependency index MUST answer the query: *given a set of changed paths, artifacts or contract versions, which artifacts are directly and indirectly affected* (transitively through `basedOn` and `supersedes`, with a depth limit).

**Knowledge impact check.**

9. Every change on the `code` change channel MUST run the `knowledge.impact` check.
10. The `knowledge.impact` check MUST list the affected artifacts in the evidence and the PR.
11. The `knowledge.impact` check MUST return `FAIL` when an `ACTIVE` artifact with authority `intent`, `decision`, `procedure` or `guidance` is affected and the task has not recorded a reconciliation decision (§5.10) for it.
12. The `knowledge.impact` check MUST return `FAIL` when that reconciliation decision was not made by a person belonging to the artifact's authority; `decidedBy` MUST be a valid human identity according to the authority table in §5.10.
13. Agent-proposed decisions have state `proposed` and MUST NOT satisfy the `knowledge.impact` check.
14. For affected `implementation-description` artifacts, the `knowledge.impact` check MUST only warn (`warn`).
15. The `knowledge.impact` check MUST follow the Check Registry lifecycle (§8.7): starting at `shadow`, then becoming `active`.

**Upgrade path.**

16. The dependency index SHOULD be upgraded along the following path:
    - *Pilot:* frontmatter in Git, `describes` as path globs, index generated in the CI of a single repository.
    - *Multiple repositories:* index aggregated by the Control Plane's Knowledge Registry (§3.2), together with catalog metadata (§4.8).
    - *Graph service:* the final step of the path, under the conditions in item 17.
17. A graph service SHOULD be introduced only when there is measured evidence that the index no longer suffices (for example, cross-repository impact queries exceed the CI time budget, index rebuilds exceed budget, or workflows frequently need transitive cross-repository queries).
18. The decision to introduce a graph service MUST be recorded as an ADR.

**Rationale.** This is the impact analysis of §4 applied to knowledge: instead of asking "which services are affected", the Factory asks "which documents, specs, ADRs and skills may no longer be correct after this change".

`describes` should be narrow so that impact analysis does not turn every change into a review of all knowledge. An `AGENTS.md` declaring `describes: **` would be flagged on every change and would quickly be ignored.

The `knowledge.impact` check only warns for `implementation-description` artifacts because these artifacts are handled by scheduled reconciliation when they are not updated in the same change; by contrast, `intent`, `decision`, `procedure` and `guidance` artifacts require a decision by a human with authority before the change can proceed.

The upgrade path starts with frontmatter and an index generated in CI because most impact queries at pilot scale fall within a single repository.

## 5.10. Reconciliation when the system changes

**Triggers.**

1. The Factory MUST trigger reconciliation when:
   - a change touches the `describes` scope of an artifact;
   - a contract changes version;
   - an ADR moves to `ACTIVE`, `SUPERSEDED` or `DEPRECATED`;
   - a spec or product requirement changes;
   - drift is detected (§4.4, §5.6), an explainer conflicts (§5.7), or an incident reveals incorrect knowledge (§12.4);
   - the owner of an artifact is gone.

**Process.**

2. **Find related artifacts:** reconciliation MUST find related artifacts through the dependency index (§5.9).
3. **Analyze dependencies and impact:** reconciliation MUST distinguish direct from indirect impact, and identify which in-progress tasks have that artifact in their `basis`.
4. **Assess validity:** reconciliation MUST assess, for each artifact, whether it is still correct.
5. For `intent` or `decision` artifacts, a difference from the implementation MUST be handled as a conflict per §4.7, not as drift.
6. **Decide:** the decision MUST be recorded using one of the following values.

| Decision | Meaning | Consequence |
|---|---|---|
| `keep` | Still valid | Update `validatedAgainst`; state remains `ACTIVE` |
| `update` | Content needs correction, meaning unchanged | Same `id`, increment `version`, review per the authority table below |
| `supersede` | A new replacement artifact is needed | The new artifact goes through the corresponding checkpoint; the old artifact moves to `SUPERSEDED` |
| `deprecate` | No longer in effect, no replacement | Move to `DEPRECATED` with a reason |
| `review` | Needs authority review before a decision | Move to `NEEDS_REVALIDATION`; open a work item for the owner |

7. **Synchronize:** the decision MUST be applied through the normal change channel (PR), with approval per the authority table and §9.10.
8. Agent instruction files and playbooks are control files and MUST additionally be subject to the rules of §7.13.
9. **Revalidate:** reconciliation MUST update `validatedAgainst`, transition the state as appropriate, rerun the affected `verifiedBy` tests and checks, and notify in-progress tasks that have the artifact in their `basis` (§6.5).
10. Each reconciliation MUST produce a `ReconciliationRecord` (Appendix K) in the Feedback & Evidence Plane.

**Who may record decisions.**

11. `decidedBy: policy` MUST be used only for `update` decisions on artifacts `generatedFrom` an approved source.
12. All other decisions MUST be made by a human belonging to the artifact's authority.
13. Agents MUST only propose reconciliation decisions.
14. A `keep` decision MAY be confirmed by the person filling the PR's approval slot if that person belongs to the artifact's authority (for example, the code owner of the area for `implementation-description` and `guidance`).

**`validatedAgainst` is updated after merge.**

15. A `keep` or `update` decision MUST be recorded in the PR and in the `ReconciliationRecord`.
16. The artifact's `validatedAgainst` field MUST be updated after merge by a registered trusted mechanism (§7.13), using the commit actually present on main.
17. The Factory MUST NOT treat lifecycle-metadata-only changes produced by this mechanism as control file changes requiring approval (§7.13).

**Authority by artifact type.**

18. The Factory MUST apply the following authority table.

| `authority` | What agents or automation may do | Decision maker |
|---|---|---|
| `implementation-description` | Propose `update`. Automatic regeneration only when the artifact is `generatedFrom` an approved source | Document owner, reviewed like code |
| `intent` | Only flag `NEEDS_REVALIDATION` and record a knowledge conflict | Product/domain owner, through Specification Review (§6.12) |
| `decision` | Only propose a new ADR | Architect or code owner of the area, through Design Review (§6.12) |
| `procedure` | Propose | Owner of the runbook or playbook; playbooks follow control file rules (§7.13) |
| `guidance` | Propose | Independent code owner (§7.13, §9.10) |

**Rules.**

19. The Factory MUST NOT automatically modify a document merely because it differs from the implementation. The only exception is regenerating a `generatedFrom` artifact from an approved source.
20. For `intent` and `decision` artifacts, the Factory MUST NOT automatically modify content; differences MUST be handled per §4.7.
21. When a task changes behavior according to an approved spec, the spec MUST be updated and approved *before* implementation (it is part of the `basis`).
22. In the case of item 21, affected `implementation-description` artifacts SHOULD be updated in the same change.
23. Scheduled or batch reconciliation MUST run as a separate workflow (*Knowledge Reconciliation*, Appendix D), with a budget.
24. Scheduled or batch reconciliation MUST NOT be combined with behavior changes.
25. The Factory MUST NOT rewrite history: superseded artifacts MUST retain their content unchanged.
26. Completed tasks MUST retain their basis snapshot unchanged (§6.5).

**Rationale.** Documents are not automatically modified merely because they differ from the implementation, because the difference may be a bug in the code. For `intent` and `decision` artifacts, modifying the document to match the code would legitimize the bug as intent.

The person filling the PR's approval slot (when belonging to the artifact's authority) may confirm `keep` so as not to create additional human touches.

`validatedAgainst` is updated after merge because only after merge is there an actual commit on main to reference; recording the commit of an unmerged branch would point to a state that may never exist on main.

Scheduled or batch reconciliation is kept separate from behavior changes so that reviewers do not have to evaluate a behavior change and a batch of unrelated documentation updates at the same time.
