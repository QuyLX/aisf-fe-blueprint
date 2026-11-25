# Appendix E – Knowledge artifact metadata

## E.1. Frontmatter

1. Every governed knowledge artifact (§5.8) MUST have metadata per the table below, placed in YAML frontmatter at the top of the file or in a sidecar file with the same name.
2. Metadata MUST validate against the `KnowledgeArtifactMeta` schema (Appendix K); enumerated values MUST come from the vocabulary (Appendix J).
3. The `validatedAgainst` field MUST only be updated by a registered trusted mechanism, after merge (§5.10).

| Field | Required | Meaning |
|---|---|---|
| `id` | Yes | Stable identifier, unique within the repository |
| `kind` | Yes | `spec`, `design`, `adr`, `proposal`, `living-doc`, `runbook`, `agent-guide`, `skill`, `playbook` |
| `status` | Yes | Lifecycle status (§5.8) |
| `version` | Yes | Incremented when content changes while meaning is preserved (§5.8 item 13) |
| `owner` | Yes | Responsible team or person |
| `authority` | Yes | `intent`, `decision`, `implementation-description`, `procedure`, `guidance` |
| `describes` | When the artifact describes code, a contract or an entity | Path glob, entity or `contract:<id>`; SHOULD be as narrow as possible (§5.9) |
| `basedOn` | When validity depends on other artifacts | List of `<id>@<version>` |
| `generatedFrom` | When the artifact is generated automatically | Approved source |
| `supersedes` / `supersededBy` | When a supersession relationship exists | `supersededBy` is required with status `SUPERSEDED` |
| `validatedAgainst` | For `ACTIVE` artifacts | Commit, contract version or artifact version against which the content is confirmed correct |
| `reviewTriggers` | No | Events that trigger review |
| `deprecationReason` | With status `DEPRECATED` | Reason the artifact is no longer in effect |

**Example.** Spec (`authority: intent`):

```yaml
---
id: SPEC-PIPE-RETRY
kind: spec
status: ACTIVE
version: 3
owner: cicd-team
authority: intent
describes:
  - src/domains/pipeline/retry/**
  - contract:pipeline-api-v2
basedOn:
  - ADR-0012@1
  - contract:pipeline-api-v2@2.3.0
validatedAgainst: { commit: 9f0e1d2, at: 2026-11-18 }
reviewTriggers: [contract:pipeline-api-v2 changes major version]
---
```

A superseded ADR (`authority: decision`):

```yaml
---
id: ADR-0007
kind: adr
status: SUPERSEDED
version: 1
owner: cicd-architecture
authority: decision
supersededBy: ADR-0012
---
```

A generated document (`authority: implementation-description`), allowed to be regenerated automatically:

```yaml
---
id: DOC-PIPELINE-API-REF
kind: living-doc
status: ACTIVE
version: 14
owner: cicd-team
authority: implementation-description
generatedFrom: contract:pipeline-api-v2@2.3.0
describes: [contract:pipeline-api-v2]
validatedAgainst: { contract: pipeline-api-v2@2.3.0 }
---
```

## E.2. Dependency index

**Example.** `knowledge-index.json` generated in CI from frontmatter, links and requirement IDs in tests (§5.9).

```json
{
  "schemaVersion": "aisf/4.0",
  "builtFrom": { "repo": "pipeline-service", "commit": "def5678" },
  "artifacts": {
    "SPEC-PIPE-RETRY": { "status": "ACTIVE", "version": 3, "authority": "intent", "owner": "cicd-team" },
    "DOC-PIPELINE-RETRY-ARCH": { "status": "ACTIVE", "version": 5, "authority": "implementation-description", "owner": "cicd-team" }
  },
  "edges": [
    { "from": "SPEC-PIPE-RETRY", "rel": "describes", "to": "path:src/domains/pipeline/retry/**", "source": "declared" },
    { "from": "SPEC-PIPE-RETRY", "rel": "basedOn", "to": "ADR-0012@1", "source": "declared" },
    { "from": "AC-PIPE-031", "rel": "verifiedBy", "to": "test:retry/timeout.spec.ts", "source": "discovered" },
    { "from": "DOC-PIPELINE-RETRY-ARCH", "rel": "describes", "to": "path:src/domains/pipeline/retry/**", "source": "declared" }
  ]
}
```

## E.3. Reconciliation record

**Example.** A contract changes its minor version; the index finds three affected artifacts.

```yaml
kind: ReconciliationRecord
id: REC-2026-0311
trigger: { type: contract-version, ref: contract:pipeline-api-v2@2.4.0, workItemId: TASK-2026-00131 }
affected:
  - artifact: DOC-PIPELINE-API-REF@14
    relation: generatedFrom
    decision: update               # regenerated mechanically from the approved contract
    decidedBy: policy
  - artifact: SPEC-PIPE-RETRY@3
    relation: basedOn
    decision: keep                 # additive field; intent unchanged
    decidedBy: user:cicd-product-owner
  - artifact: RUNBOOK-PIPE-RETRY@2
    relation: basedOn
    decision: review               # moved to NEEDS_REVALIDATION; work item opened for the owner
    decidedBy: user:cicd-sre
activeTasksNotified: [TASK-2026-00140]   # had SPEC-PIPE-RETRY@3 in its basis; decision: continue
completedAt: 2026-11-21
```

## E.4. Knowledge conflict

**Example.** The agent detects that the code differs from the spec during diagnosis; it records a conflict instead of editing the spec (§4.7).

```yaml
kind: KnowledgeConflict
id: KC-2026-0045
workItemId: TASK-2026-00124
sources:
  - { ref: SPEC-PIPE-RETRY@3, says: "a timed-out job is retried exactly once" }
  - { ref: "code:src/domains/pipeline/retry/policy.ts#L42", says: "retries up to 3 times on timeout" }
affectsCorrectnessOrRisk: true
authority: product-owner
proposal: treat as implementation defect; spec unchanged
resolution: { decision: implementation-defect, decidedBy: user:cicd-product-owner, decidedAt: 2026-11-19 }
```
