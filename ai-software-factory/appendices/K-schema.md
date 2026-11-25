# Appendix K – Canonical schemas

Every enumerated type below takes its values from [`aisf-vocabulary.yaml`](../aisf-vocabulary.yaml) (Appendix J). A profile MAY add fields only under `extensions`, with keys of the form `x-<profile>`.

## K.1. Shared types

```typescript
type SchemaVersion = "aisf/4.0";
type RiskTier = "Low" | "Medium" | "High";
type Reversibility = "HIGH" | "MEDIUM" | "LOW";
type TrustLevel = "T0" | "T1" | "T2";
type AutonomyLevel = "A0" | "A1" | "A2" | "A3";
type Channel = "code" | "content" | "config" | "data";
type CheckStatus = "PASS" | "FAIL" | "ERROR" | "NOT_ASSESSED";
type DispositionType = "not-applicable" | "assessed-elsewhere" | "baselined" | "waived" | "deferred";
type Gate = "merge" | "publish" | "post-release";
type Enforcement = "observe" | "warn" | "blocking";
type Severity = "BLOCKER" | "WARNING" | "INFO";
type CheckRequirement = "required" | "advisory";            // per profile (§8.7)
type ProfileLayer = "core" | "domain" | "organization" | "repository" | "surface" | "task-class" | "risk" | "task";
type Executor = "ci" | "agent-sandbox" | "local";
type RuleLifecycle = "PROPOSED" | "DRAFT_WITH_FIXTURES" | "OBSERVE" | "WARN" | "BLOCKING" | "MONITOR" | "REVISE" | "DEPRECATE" | "RETIRE";
type DataClass = "public" | "internal" | "confidential" | "restricted";          // §10.6
type RetentionClass = "ephemeral" | "standard" | "audit" | "legal-hold";        // §10.7
type DependencyPolicyStatus = "APPROVED" | "CONDITIONAL" | "DEPRECATED" | "PROHIBITED";
type VulnerabilitySeverity = "critical" | "high" | "medium" | "low";           // §11.7
type ConformanceLevel = "L1" | "L2" | "L3" | "L4";

type TaskState =
  | "REQUESTED" | "TRIAGED" | "NEEDS_CLARIFICATION" | "REJECTED" | "CONTEXT_READY" | "PLANNED"
  | "AWAITING_PLAN_APPROVAL" | "EXECUTING" | "VERIFYING" | "REPAIRING" | "REVIEWING"
  | "AWAITING_MERGE_APPROVAL" | "IN_MERGE_QUEUE" | "MERGED" | "RELEASING" | "PAUSED" | "OBSERVING"
  | "ROLLED_BACK" | "REVERTED_ON_MAIN" | "INCIDENT_ANALYSIS" | "FORWARD_FIX" | "COMPLETED"
  | "BLOCKED" | "ESCALATED" | "CANCELLED";
type StepKind = "mandatory" | "conditional" | "optional";
type SideEffect = "none" | "reversible" | "compensable" | "irreversible";
type RecoveryAction = "retry" | "resume" | "rollback" | "compensate" | "escalate";
type ReviewCheckpoint =
  | "requirement-review" | "specification-review" | "design-review" | "plan-review"
  | "implementation-review" | "independent-verification" | "release-readiness";
type CheckpointOutcome = "passed" | "failed" | "not-applicable";
type BasisChangeDecision = "continue" | "replan" | "reapprove" | "stop";

type KnowledgeStatus = "DRAFT" | "PROPOSED" | "ACTIVE" | "NEEDS_REVALIDATION" | "SUPERSEDED" | "DEPRECATED" | "ARCHIVED";
type KnowledgeKind = "spec" | "design" | "adr" | "proposal" | "living-doc" | "runbook" | "agent-guide" | "skill" | "playbook";
type KnowledgeAuthority = "intent" | "decision" | "implementation-description" | "procedure" | "guidance";
type KnowledgeRelation = "describes" | "basedOn" | "generatedFrom" | "verifiedBy" | "implements" | "supersedes";
type ReconciliationDecision = "keep" | "update" | "supersede" | "deprecate" | "review";

type HumanRef = { id: string };                       // authenticated identity, never free text
type ActorRef = HumanRef | { agent: string; role: string; config?: string } | { policy: string };
type ArtifactRef = { id: string; version: string; digest?: string };   // knowledge artifact or contract, id@version
type Extensions = Record<`x-${string}`, unknown>;
```

## K.2. Verification and evidence

```typescript
interface VerificationRun {
  schemaVersion: SchemaVersion;
  runId: string;
  workItemId: string;                  // §7.5 item 1
  channel: Channel;
  subject: { repo?: string; commitSha?: string; baseSha?: string; revisionId?: string; artifactDigest?: string };
  resolvedProfiles: Array<{ id: string; version: string }>;
  resolution?: ProfileResolution;      // §7.3
  environment: { executor: Executor; runner?: string; imageDigest?: string };   // "local" never satisfies a required check
  adapters?: Array<{ id: string; version: string }>;   // §7.16
  checks: CheckResult[];
  summary: { byStatus: Record<CheckStatus, number>; requiredUnmet: string[] };
  provenance: { factoryVersion: string; verifierRef: string; producer: ActorRef | { ciJob: string };
                attestation?: string; startedAt: string; finishedAt: string };
  extensions?: Extensions;
}

interface ProfileResolution {
  layers: Array<{ layer: ProfileLayer; id: string; version: string }>;
  conflicts: Array<{
    checkId: string; parameter: string; values: unknown[];
    resolvedBy: "stricter" | "union" | "decision" | "rejected";
    decisionRef?: string;              // resolution entry or ADR, when resolvedBy = decision
  }>;
}

interface CheckResult {
  checkId: string; checkVersion: string; implementsRules?: string[];
  requiredBy: string[];                // profiles that require it
  requirement: CheckRequirement;       // strictest requirement after profile resolution (§7.3)
  enforcement: Enforcement; severity: Severity; gate: Gate;
  status: CheckStatus;
  reason?: string;                     // mandatory unless PASS
  disposition?: Disposition;
  requirementIds?: string[];           // acceptance-criteria IDs
  command: string;                     // command or procedure actually run (§7.5 item 1)
  durationMs: number;
  attempts: number;
  flaky?: { registered: boolean; quarantineId?: string };
  tool?: { name: string; version: string };
  findings?: Finding[];
  evidence: Array<{ kind: string; uri: string; digest: string; retentionClass: RetentionClass; dataClass?: DataClass }>;
  extensions?: Extensions;
}

interface Disposition {
  type: DispositionType;
  decidedBy: "policy" | HumanRef;
  decidedAt: string;
  policyRef?: string;                  // not-applicable, assessed-elsewhere, baselined
  waiverId?: string;                   // waived
  baselineRef?: string;                // baselined
  assessedElsewhere?: { runId: string; providerVersion: string; validUntil: string };
  deferredTo?: Gate;                   // deferred
}

interface Finding {
  source: "check" | "ai-review" | "security-review" | "integrity";
  ruleId?: string; tag?: "Important" | "Nit" | "Pre-existing";
  severity: string;                    // profile-defined inside checks
  location?: string; message: string;
  state: "open" | "fixed" | "dismissed";
  authorResponse?: "fixed" | "wrong" | "intentional" | "didnt-matter";   // §7.12
  dismissedBy?: HumanRef; dismissReason?: string;
}

interface VerificationRequest {        // §7.16 — the only input to an adapter
  schemaVersion: SchemaVersion;
  requestId: string;
  workItemId: string;
  channel: Channel;
  subject: VerificationRun["subject"];
  gate: Gate;
  plan: Array<{                        // resolved plan: everything an adapter needs to emit a conformant CheckResult
    checkId: string; checkVersion: string; implementsRules?: string[];
    requiredBy: string[]; requirement: CheckRequirement;
    enforcement: Enforcement; severity: Severity;
    params?: Record<string, unknown>;
  }>;
  requirementIds?: string[];
  timeoutSeconds: number;
}

interface VerificationAdapter {        // registry entry, §7.16
  id: string; version: string; owner: string;
  toolchain: Array<{ name: string; version: string }>;
  imageDigest: string;
  checks: Array<{ checkId: string; evidenceLayers: string[] }>;
  sandbox: { networkEgress: "deny" | { allowlist: string[] }; secrets: string[]; writablePaths: string[] };
  timeouts: { perCheckSeconds: number; totalSeconds: number };
  fixtures: string[];                  // mutation proof (§8.7)
}
```

## K.3. Policy and approval

```typescript
interface PolicyDecision {
  schemaVersion: SchemaVersion;
  workItemId: string; channel: Channel;
  workflow?: { id: string; version: string };          // §6.7
  commitSha?: string; revisionId?: string;
  requester: { identity: HumanRef | null;   // null ⇒ source "unknown" ⇒ not eligible, independent approval required (§9.5 rule 4, §9.10 item 7)
               source: "ci-dispatch" | "issue-command" | "cli-login" | "automation-owner" | "unknown" };
  risk: { tier: RiskTier; subTier?: string; reasons: string[]; classifierVersion: string; overrideCheckVersion: string };
  reversibility: Reversibility;
  trust: { taskClass: string; level: TrustLevel; ownershipEligible: boolean;
           agentConfig?: { modelSnapshot: string; playbook: string; skills: string[] } };
  autonomy: AutonomyLevel;
  approvalSlots: ApprovalSlot[];       // the only place human approval is represented
  requiredCheckpoints: Array<{ checkpoint: ReviewCheckpoint; required: boolean; condition?: string }>;
  aiReview?: { mode: "shadow" | "advisory" | "blocking"; depth: "quick" | "standard" | "deep"; degraded: boolean };
  budgets: Record<"verifyRepairIterations" | "reviewRounds" | "mergeQueueAttempts", { limit: number; used: number }>
         & { cost?: { limit: number; used: number; unit: "usd" | "tokens" } };
  tierStatus: "provisional" | "official";      // official = recomputed from the diff (§6.7.3)
  protectedCategories?: string[];              // vocabulary protectedCategory
  acceptedRuns: string[];              // attested runs only, from L3
  blockingChecks: string[];
  openFindings: number;
  breakGlass?: { invokedBy: HumanRef[]; reason: string; expiresAt: string; postHocDueBy: string };
  policyVersion: string; decidedAt: string;
  extensions?: Extensions;
}

interface ApprovalSlot {
  kind: "requester-signoff" | "independent" | "code-owner" | "control-file-owner" | "security-owner"
      | "business-owner" | "plan-approval" | "product-owner" | "architect" | "service-owner"
      | "incident-commander" | "domain-owner";
  checkpoint?: ReviewCheckpoint;       // §6.12
  scope?: string;                      // area, control-file path, or content type
  countsTowardLevel: boolean;          // requester sign-off counts only where §9.10 allows
  subject?: ArtifactRef | { commitSha: string } | { artifactDigest: string };   // exact object approved
  basisRef?: string;                   // TaskBasis the approval relied on (§6.5)
  filledBy?: HumanRef; commitSha?: string; filledAt?: string;
  invalidatedAt?: string; invalidatedBy?: string;      // e.g. new commit, basis change
}

interface Waiver {
  id: string; checkId: string; scope: string; reason: string;
  owner: HumanRef; approvedBy: HumanRef;
  createdAt: string; expiresAt: string;
}

interface TrustEvaluation {            // §9.4
  taskClass: { workflow: string; workflowMajor: number; scope: string; riskCeiling: RiskTier };
  agentConfig: { modelSnapshot: string; playbook: string; skills: string[] };
  window: { from: string; to: string };
  strata: Array<{ name: string; tasks: number; accepted: number; minRequired: number }>;
  criticalScenarios: { total: number; passed: number };
  holdout: { score: number; seenScore: number; maxGap: number };
  regression: { evalSets: string[]; regressions: number };
  escapedDefects: number;
  failureRateUpper95: number;          // confidence bound, not a point estimate
  interventionRate: number;
  resultLevel: TrustLevel;
  status: "CURRENT" | "RE-EVALUATION_REQUIRED";
  decidedBy: HumanRef[];               // area owner and Factory owner (§9.10)
  decidedAt: string;
}
```

## K.4. Task and workflow

```typescript
interface TaskWorkflowSpec {           // §6.7, Appendix C
  apiVersion: SchemaVersion; kind: "TaskWorkflowSpec";
  metadata: { id: string; version: string; owner: string; status: KnowledgeStatus };
  spec: {
    taskClasses: string[]; riskCeiling: RiskTier;
    trigger: string[];
    inputs: { required: string[]; optional?: string[] };
    preconditions: string[];
    subStates?: Partial<Record<TaskState, string[]>>;  // first entry = initial sub-state
    checkpoints: Array<{ id: ReviewCheckpoint; kind: "mandatory" | "conditional"; when?: string; after: string }>;
    steps: WorkflowStep[];
    handoffs?: Array<{ from: string; to: string; validIn: TaskState[]; requiredArtifacts: string[]; excludeContext?: string[] }>;
    maxHandoffRejections?: number;
    budgets?: Partial<Record<"verifyRepairIterations" | "reviewRounds" | "mergeQueueAttempts", number>>
            & { cost?: { limit: number; unit: "usd" | "tokens" } };
    recoveryDefaults?: RecoveryProps;
    completion: { requiredOutputs: string[] };
    metrics?: string[];
  };
}

interface RecoveryProps {              // §6.14
  timeout?: string;
  retry?: { max: number; backoff: "none" | "linear" | "exponential"; on: Array<"ERROR" | "timeout"> };
  idempotent?: boolean;
  sideEffect?: SideEffect;
  rollback?: string;                   // reversible steps
  compensation?: string;               // compensable steps; reviewed before the step runs
}

interface WorkflowStep extends RecoveryProps {
  id: string;
  skeletonStep: number;                // 1–16 (§6.6)
  kind: StepKind;
  when?: string;                       // conditional steps; evaluated by policy
  subState?: string;
  role: string;                        // §6.3 role, or "policy" | "human"
  skills?: string[]; tools?: string[];
  inputs: string[]; outputs: string[];
  verification: string;                // check ID or completion criterion
  onFailure: string | Record<string, unknown>;
}

interface TaskBasis {                  // §6.4, §6.5
  id: string;
  digest: string;                      // digest of the snapshot below
  workItemId: string;
  artifacts: ArtifactRef[];
  assumptions: string[];               // confirmed only
  scopeRef: string;                    // Change Brief version
  capturedAt: string;
}

interface StateTransition {            // every state or sub-state change (§6.5)
  workItemId: string;
  from: { state: TaskState; subState?: string };
  to: { state: TaskState; subState?: string };
  actor: ActorRef; at: string;
  inputs?: string[]; outputs?: string[];
  policyDecisionRef?: string; checkpointRef?: string;
  commitSha?: string; workspace?: string;
  reason?: string;
}

interface BasisChange {                // §6.5
  workItemId: string;
  trigger: { type: "artifact-version" | "artifact-status" | "assumption-falsified" | "scope-discovery"; ref?: ArtifactRef; detail: string };
  previousBasis: string; newBasis?: string;
  decision?: BasisChangeDecision;
  decidedBy?: HumanRef; decidedAt?: string; reason?: string;
  invalidatedApprovals: string[];
  blockedGate: string;
}

interface CheckpointRecord {           // §6.12
  id: string; workItemId: string;
  checkpoint: ReviewCheckpoint;
  subject: ArtifactRef | { commitSha: string } | { artifactDigest: string };
  outcome: CheckpointOutcome;
  condition?: string;                  // evaluated condition, for not-applicable
  reviews: string[];                   // finding IDs (review)
  approvals: ApprovalSlot[];           // approval
  verificationRuns: string[];          // verification
  basisRef?: string;
  recordedAt: string;
}

interface Handoff {                    // §6.13
  id: string; workItemId: string;
  from: { step: string; role: string; actor: ActorRef };
  to: { step: string; role: string; actor?: ActorRef; queue?: string };
  at: { state: TaskState; subState?: string };
  artifacts: Array<{ name: string; uri: string; digest: string }>;
  basisRef: string;
  decisions: string[]; openQuestions: string[]; assumptions: string[];
  evidenceRefs: string[];
  workspace?: { branch: string; baseSha: string; headSha?: string };
  excludedContext?: string[];
  budgetUsed?: Record<string, number>;
  deadline?: string;
  status: "accepted" | "rejected";
  missing?: string[];                  // when rejected
  respondedAt: string;
}

interface RecoveryEvent {              // §6.14
  workItemId: string; step: string;
  cause: "timeout" | "ERROR" | "FAIL" | "budget-exhausted" | "handoff-rejected" | "basis-change" | "compensation-failed" | "paused";
  action: RecoveryAction;
  attempt?: number;
  checkpointRef?: string;
  idempotencyKey?: string;
  escalatedTo?: HumanRef;
  at: string; detail?: string;
}
```

## K.5. Knowledge

```typescript
interface KnowledgeArtifactMeta {      // §5.8, Appendix E
  id: string; kind: KnowledgeKind; status: KnowledgeStatus; version: string;
  owner: string; authority: KnowledgeAuthority;
  describes?: string[];                // paths (glob), entities, "contract:<id>"
  basedOn?: string[];                  // "<id>@<version>"
  generatedFrom?: string;
  supersedes?: string[]; supersededBy?: string;
  validatedAgainst?: { commit?: string; contract?: string; artifact?: string; at?: string };
  reviewTriggers?: string[];
  deprecationReason?: string;
  extensions?: Extensions;
}

interface KnowledgeIndex {             // §5.9
  schemaVersion: SchemaVersion;
  builtFrom: { repo: string; commit: string };
  artifacts: Record<string, Pick<KnowledgeArtifactMeta, "status" | "version" | "authority" | "owner">>;
  edges: Array<{ from: string; rel: KnowledgeRelation; to: string; source: "declared" | "discovered" | "observed" }>;
}

interface ReconciliationRecord {       // §5.10
  id: string;
  trigger: { type: "describes-change" | "contract-version" | "adr-status" | "spec-change" | "drift" | "incident" | "owner-removed"; ref: string; workItemId?: string };
  affected: Array<{
    artifact: string;                  // "<id>@<version>"
    relation: KnowledgeRelation;
    decision: ReconciliationDecision;
    state: "proposed" | "decided";
    decidedBy?: "policy" | HumanRef;   // "policy" only for update on a generatedFrom artifact; otherwise a human of the artifact's authority
    newVersion?: string; supersededBy?: string; workItem?: string;
  }>;
  activeTasksNotified: string[];
  completedAt?: string;
}

interface KnowledgeConflict {          // §4.7
  id: string; workItemId?: string;
  sources: Array<{ ref: string; says: string }>;
  affectsCorrectnessOrRisk: boolean;
  authority: string;                   // role that decides (§4.7 table)
  proposal?: string;
  resolution?: {
    decision: "implementation-defect" | "intent-changed" | "artifact-corrected" | "superseded" | "no-conflict";
    decidedBy: HumanRef; decidedAt: string; followUp?: string;
  };
}
```

## K.6. Task, rule, check and conformance claim

```typescript
type AllowedAutonomy = "propose-only" | "implement-and-request-review" | "verify-and-open-pr" | "merge-on-eligible-path";

interface Task {                       // §6.4, §6.4.1
  apiVersion: SchemaVersion; kind: "Task";
  metadata: { id: string;
              requestedBy: HumanRef | null;    // recorded by the task runner, never by the agent; null ⇒ requesterSource "unknown" ⇒ not eligible
              requesterSource: PolicyDecision["requester"]["source"]; supersedesTask?: string };
  spec: {
    goal: string; why: string;
    taskType: string;
    workflow: string;                  // "<workflow-id>@<version>", "generic@<version>" when no specific workflow is ACTIVE
    repositories: string[];
    scope: { in: string[]; out: string[] };
    doneWhen: Array<{
      id: string; statement: string;
      oracle: "existing-test" | "new-independent-test" | "contract" | "runtime-check" | "human-judgment";
      critical: boolean;               // confirmed at requirement-review
    }>;
    assumptions: Array<{ statement: string; status: "confirmed" | "unconfirmed" }>;
    risksAndUnknowns: string[];
    howToVerify: string; howToRevert: string;
    expectedEvidence: string[];
    basis?: { basisId: string; digest: string };   // TaskBasis (K.4) snapshot
    policy: {
      verificationProfile: string;
      budgets: Partial<Record<"verifyRepairIterations" | "reviewRounds" | "mergeQueueAttempts", number>>
             & { cost?: { limit: number; unit: "usd" | "tokens" } };
    };
  };
}

interface Rule {                       // §8.2
  id: string;                          // <SCOPE>-<CATEGORY>-<NNN>
  version: string;                     // semver
  name: string;
  category: string;                    // core or profile-registered rule category
  lifecycle: RuleLifecycle;            // §8.3
  enforcement: Enforcement;            // follows lifecycle (OBSERVE/WARN/BLOCKING)
  scope: { include: string[]; exclude?: string[] };
  detection: { type: string };
  message: string; owner: string;
  fixtures: { positive: string[]; negative: string[] };
  exception?: { approvalRequired: boolean; expiryRequired: boolean };
}

interface Check {                      // §8.7
  kind: "Check";
  id: string; version: string;         // <domain>.<name>
  owner: string; fixOwner: string;
  lifecycle: "planned" | "shadow" | "active" | "deprecated";
  ownership: "local" | "cross-team-contract";
  implementsRules: string[];
  basis: { class: "standard" | "contract" | "security" | "privacy" | "legal" | "policy" | "record"; source: string };
  evidenceLayer: string;
  baselineable: boolean;
  fpSlo: number;
  autoDemote: boolean;                 // false for core checks and security/privacy/legal/contract; only opens a proposal
  fixtures: { conforming: string[]; violating: string[] };
  remediation: string;
}

interface ConformanceClaim {           // §20.5
  kind: "ConformanceClaim";
  scope: { repo: string; areas?: string[] };
  standard: string;                    // exact pinned version, e.g. "aisf@4.0.0" (§17.4 item 2)
  profile: string;                     // registered domain profile "<id>@<version>", or the Core Profile "aisf-core@<standard version>" (§20.7)
  mode: "pilot" | "standard";          // §20.8
  level: ConformanceLevel;
  additionalRequirements: string[];    // CORE-IDs above the claimed level
  evidence: Array<{ requirement: string; scenario?: string; run?: string; registryRecord?: string }>;
  deviations: string[];                // ADR references (PC-4)
  assessedBy: HumanRef; assessedAt: string; validUntil: string;
}
```

## K.7. Profile, registry and repository configuration

```typescript
interface ProfileRegistryEntry {       // §20.2 (PC-1 – PC-7)
  kind: "Profile";
  id: string; version: string;         // e.g. "aisf-frontend@1.1.0"; the Core Profile is "aisf-core"
  owner: HumanRef; backupOwner: HumanRef;
  standard: string;                    // exact pinned Standard version (PC-1)
  supportedLevels: ConformanceLevel[];
  vocabularyMapping: string;           // URI of the PC-2 mapping
  extensions: string[];                // IDs of ExtensionRegistryEntry (PC-5)
  deviations: string[];                // ADR references (PC-4)
  delegatedDeviationApproval?: { bounds: string; grantedBy: HumanRef; expiresAt: string };   // §17.4 item 19
  conformanceScenarios: string[];      // PC-6
  status: KnowledgeStatus;
}

interface ExtensionRegistryEntry {     // §20.3, PC-5
  id: string;                          // namespaced, e.g. "x-frontend.sub-tier.R0"
  profile: string;                     // "<id>@<version>"
  point: "risk-sub-tier" | "autonomy-matrix" | "composition-dimension" | "check" | "rule-category"
       | "finding-severity" | "evidence-field" | "change-channel" | "task-class" | "workflow" | "sub-state"
       | "skill" | "knowledge-kind" | "knowledge-relation" | "verification-adapter" | "profile-layer"
       | "conformance-criterion" | "trust-threshold";
  mapsTo?: string;                     // canonical term it maps to (PC-2), e.g. a tier or parent task state
  owner: HumanRef; registeredAt: string;
}

interface FactoryConfig {              // factory.yaml — control file (§7.13); only the fields used normatively
  apiVersion: SchemaVersion; kind: "FactoryConfig";
  profile: string;                     // "<id>@<version>", domain profile or Core Profile (§20.7)
  verificationProfile: string;         // repository layer of §7.3
  promptAssets: string[];              // globs exempt from the §7.9 pre-scan, protected by code-owner approval
  autonomy: {
    zeroHumanOptIn: string[];          // areas opted in to A0 (§9.6 item 9)
    aiReviewMode: Record<string, "shadow" | "advisory" | "blocking">;   // per area (§7.9 item 30)
  };
  riskMap?: Array<{ paths: string[]; tier: RiskTier; protectedCategory?: string }>;   // deterministic path → tier map (§20.8 item 4); unmatched paths are High
  dataClass?: Array<{ paths: string[]; class: DataClass }>;                          // §10.6
  extensions?: Extensions;
}

interface DependencyDecisionRecord {   // §11.6
  id: string; workItemId: string;
  dependency: { kind: "third-party-package" | "internal-module" | "service" | "platform-toolchain" | "ai-model";
                name: string; version: string; ecosystem?: string };
  action: "add" | "upgrade" | "replace" | "remove";
  alternativesConsidered: string[];    // including existing approved capabilities searched
  evaluation: Record<"function" | "architecture" | "compatibility" | "security" | "license" | "maintenance"
                     | "performance" | "ecosystem" | "cost" | "reversibility", string>;   // §11.3
  hardGates: { security: "pass" | "fail"; license: "allow" | "deny" | "review"; dataPolicy: "pass" | "fail" };
  policyStatus: DependencyPolicyStatus;
  lockedVersion: string; sbomUpdated: boolean;
  aiModel?: { snapshot: string; deprecationDate?: string; fallback?: string; evalRunRef?: string };   // §11.9
  decidedBy: HumanRef; decidedAt: string;
  reviewTrigger?: string;
}

interface ProviderDataPolicy {         // §10.2, §10.6
  provider: string;                    // model provider or tool, with the approved version or snapshot
  maxDataClass: DataClass;             // highest class it may receive
  retention: string; trainingUse: "none" | "opt-out" | "allowed";
  region?: string; owner: HumanRef; approvedAt: string; reviewBy: string;
}

interface ThreatModelRecord {          // §10.8
  subject: { kind: "agent-runtime" | "tool" | "mcp-server"; id: string; version: string };
  threats: string[];                   // including prompt injection, data exfiltration, tool misuse, privilege escalation
  mitigations: string[];
  approvedBy: HumanRef; approvedAt: string; reviewBy: string;
}
```
