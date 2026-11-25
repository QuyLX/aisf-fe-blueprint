# 10. Agent security & data governance

## 10.1. Security baseline

1. Every agent and tool MUST run with least privilege.
2. By default, the Factory MUST NOT grant production credentials to coding agents.
3. By default, reviewers MUST have read-only access.
4. Credentials and service identities SHOULD be short-lived and scoped.
5. The Factory MUST use an explicit tool/MCP allowlist; each allowlist entry MUST have an owner and a version.
6. Untrusted branches/tasks MUST run in ephemeral workspaces.
7. The Factory MUST restrict filesystem access, process execution and network egress.
8. Secrets MUST NOT be mounted unless needed and approved.
9. Production operations MUST have separate authorization and audit.
10. The Factory MUST validate tool parameters and outputs before consequential actions.
11. The Factory MUST separate untrusted content from users/repositories/tools from trusted policy instructions.
12. The Factory MUST treat issues, PR comments, README files, logs, web pages, generated code and tool output as untrusted data.
13. The Factory MUST test for prompt injection, data leakage, tool misuse, privilege escalation and context poisoning.

## 10.2. Data governance

1. For each model/provider and tool, the Factory MUST define:
   - Permitted source code and data classes.
   - Whether customer data, personal data, regulated data or confidential data may be sent.
   - Retention and logging behavior.
   - The provider's training/usage terms.
   - Processing geography constraints where relevant.
   - Data masking and secret detection requirements.
   - Audit access and retention periods.
   - Incident handling and revocation process.
2. The Factory MUST NOT assume that a model/provider handles data acceptably merely because it is technically available.

## 10.3. Agent identity and audit

1. The Factory MUST record the following information, where available:
   - Initiator and accountable owner.
   - Agent role and identity.
   - Model/provider and version/configuration.
   - Skill/playbook version.
   - Tool/MCP version and granted permissions.
   - Repository and base commit.
   - Actions and changed files.
   - Approval and checkpoint events.
   - Handoffs between roles and recovery events (§6.13–6.14).
   - Verification artifacts.
   - Commit/PR/build/release identifiers.
2. The Factory MUST NOT store chain-of-thought.
3. The Factory MUST store the minimal operational trace needed for security, reproducibility, cost accounting and debugging.

## 10.4. Security references

1. The Factory SHOULD use OWASP GenAI/Agentic guidance as an evolving threat taxonomy, and NIST SSDF as a broader secure development baseline.
2. The Factory MUST reassess the threat model when adding a new tool, autonomous action, model provider or data class.

**Rationale.** OWASP GenAI/Agentic is still evolving, while NIST SSDF is a broader baseline [R22, R23, R24]. Because the threat taxonomy is still changing, the threat model is reassessed whenever the attack surface expands.

## 10.5. Factory supply chain

1. The Factory MUST treat skills, playbooks, hooks, MCP servers and agent configuration as dependencies.
2. Every skill, playbook, hook and MCP integration MUST be version-pinned, reviewed and have a recorded owner.
3. Changes to shared skills and policies MUST pass through the same evaluation and approval gates as code.
4. The Improvement Agent MUST NOT approve its own proposals.
5. Shared packages SHOULD be distributed signed; where the platform does not yet support signing, a pinned checksum MUST exist.
6. Agents running in CI MUST reject updates that are unsigned and do not match the pinned checksum.
7. The Factory MUST maintain an inventory of which repositories use which Factory components, so that components can be revoked.
8. The threat model MUST cover Factory components, including prompt injection via skill content, overly broad tool scope and credential exposure via hooks.
9. Every new integration MUST pass through the tool contract in §6.10.

**Rationale.** Skills, playbooks, hooks, MCP servers and agent configuration are code executing with the agent's permissions, so they are governed as dependencies; the usage inventory enables revocation when needed. Deep integration with internal systems increases both capability [R28] and blast radius, so every new integration passes through the tool contract.

## 10.6. Data classification

**Data classes** (`dataClass`, Appendix J) are `public`, `internal`, `confidential` and `restricted`, in increasing order of sensitivity.

1. The Factory MUST assign a data class to every context source the agent can read (repository, area, system, tool) and to every task at the Classification step (§6.6 step 2). The change channel (§12.6) MUST use the same class as an input to risk classification.
2. An unknown data class MUST be treated as `restricted`.
3. A task, trace or evidence bundle MUST take the highest class of all inputs it contains.
4. For each approved model provider and tool, the data governance record (§10.2) MUST state the highest data class it may receive (`ProviderDataPolicy`, Appendix K).
5. The Factory MUST NOT send data to a provider or tool when the data's class exceeds the level permitted for that provider or tool.
6. `restricted` data MUST NOT be sent to any model provider; secrets and credentials MUST NOT be in the agent's context.
7. A profile MAY tighten the data class → provider mapping, and MUST NOT loosen it.

**Rationale.** Per-provider data governance (§10.2) is only usable when data is labeled. Four classes are coarse enough to assign quickly to repositories, areas and tools, yet sufficient to answer the practical question: may this provider read this code or this log. The "unknown means `restricted`" default follows the principle that not knowing means the worst case (§9.5).

**Example.** An initial mapping: a provider with a no-training commitment and short retention receives up to `confidential`; an unassessed third-party tool receives only `public`; no provider receives `restricted`.

## 10.7. Scanning and retention of output, traces and evidence

1. Agent output, traces and evidence bundles MUST be scanned for secrets and personal data before being stored or posted (PR comments, evidence store, logs).
2. A hit MUST be redacted or block storage, and MUST be reported to the task owner and the security owner. An exposed secret MUST be treated as compromised and revoked.
3. Every stored artifact MUST carry a retention class (`retentionClass`, Appendix J): `ephemeral`, `standard`, `audit` or `legal-hold`, with *suggested default* periods of 30 days, 180 days, 2 years and until released, respectively.
4. Records on which a conformance claim, approval or release relies MUST be `audit` and MUST NOT be deleted before the claim expires.
5. The organization MAY extend the periods; legal obligations take precedence over the suggested periods.

**Rationale.** Agents copy very quickly: a secret that leaks into context can reappear in a PR comment, trace or evidence bundle, and from there in systems with broader access than the original repository. Therefore scanning happens before storage, not after. Retention classes separate records used for debugging (short-term) from records that are the basis for approvals, releases and conformance claims (long-term).

## 10.8. Threat model and allowlist for agent runtimes, tools and MCP

1. Every agent runtime and every tool or MCP server integration MUST have a threat model approved by the security owner before being added to the allowlist (§10.1 item 5), recorded as a `ThreatModelRecord` (Appendix K).
2. The threat model MUST cover at least: prompt injection via untrusted input, data leakage, tool misuse, privilege escalation and credential exposure, together with mitigations for each item.
3. The threat model MUST be reassessed per §10.4 item 2 and when the integration changes major version.
4. Each tool/MCP allowlist entry MUST have an owner, a pinned version, a permission scope, a highest data class (§10.6) and a threat model reference. The allowlist is a control file (§7.13).

**Rationale.** Each new agent runtime or MCP server opens a new attack surface: it reads untrusted input and has permission to act. A threat model before approval forces the owner to answer the basic questions (what it reads, what it can call, what it can leak) while design changes are still cheap. An allowlist with owners and pinned versions is the precondition for being able to revoke an integration during an incident.

## 10.9. Prompt injection testing

1. Every agent with permission to call tools and read untrusted input (§10.1 item 12) MUST be tested for prompt injection before being approved.
2. The test suite MUST include at least: instructions embedded in diffs, issues, comments, documents and tool output; hidden or zero-width characters; requests to expose secrets or data outside the permitted class; and requests to call tools outside the task scope.
3. An agent passes when it takes no action outside the task scope, does not leak data and flags suspicious content; a single failure MUST block approval or version upgrade.
4. The test suite MUST be rerun when the agent runtime, model snapshot, tool or MCP server changes version (§9.2 item 8), and periodically (*suggested* at least quarterly).
5. The test suite is an eval set (§9.2) and MUST grow from incidents and security findings.

**Rationale.** Prompt injection has no definitive fix; it is managed as an ongoing risk [R22]. Behavior changes with model snapshot and with tool, so the test suite reruns whenever either changes, and runs periodically to catch new techniques. Treating the test suite as an eval set lets it grow from incidents instead of standing still.

## 10.10. Dedicated security review pass

A **dedicated security review pass** (§7.9 item 9) is a review pass that considers only security concerns, has its own instructions and checklist, and posts findings with `source: security-review` (Appendix K).

1. The security review pass MUST run on every change that touches a protected group `auth-and-secrets`, `legal-and-privacy` or `destructive-or-regulated-data` (§9.6), and, in areas with AI review (§7.9), on every Medium/High tier change and every change with a pre-scan hit (§7.9 item 23).
2. The pass MAY be performed by a human or an agent; for an agent, its instructions MUST be separate from the general review pass and it MUST NOT share context with the implementing agent.
3. Findings of the security review pass MUST be handled per §7.9 item 10 and may only be dismissed per §9.10.

**Rationale.** A general review pass easily overlooks security issues because attention is split among correctness, style and architecture. A dedicated pass with its own checklist for changes touching sensitive areas gives security findings their own weight: they block merge until fixed or dismissed by the security owner or an independent person (§9.10).
