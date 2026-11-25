# Appendix F – `REVIEW.md` example

```markdown
# Review Guide – pipeline area
Owners: cicd-team (see CODEOWNERS)

## What failure looks like here
- A retry runs twice, or never (idempotency broken).
- The UI shows a pipeline state the backend does not hold.
- A permission check exists only in the UI.

## Always check
- Server-side authorization for every new pipeline action.
- Timeout and retry behavior covered by a test.
- API client changes match contracts/openapi/pipeline-api-v2.yaml.

## Skip
- src/generated/**
- **/__snapshots__/** unless the PR changes behavior

## Rules for findings
- Cite file and line; no findings inferred from names alone.
- At most 3 nits per review.
- Tag every finding: Important | Nit | Pre-existing.
- Start the summary with the count ("2 Important, 1 Nit").
  If there is nothing to fix, say "No issues found."
```
