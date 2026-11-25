# 19. Design stance

The AI Software Factory Standard describes a **federated, harness-driven, evidence-based distributed engineering system**, applied to each domain through profiles.

Its durable assets are not models or the number of agents. They are:

1. A legible, well-governed software ecosystem.
2. Repository harnesses that encode architecture, tools, constraints and verification.
3. Workflows specified per task type, with sub-states, checkpoints, handoffs and recovery, together with reusable, evaluated (eval) skills.
4. Product-specific verification profiles, under common evidence contracts.
5. Explicit policies on risk, trust, permissions and human accountability.
6. Feedback loops that continuously improve code, rules, knowledge and operations.
7. Knowledge with an explicit lifecycle, authority and dependencies, reconciled whenever the system changes, without rewriting history.
8. Layered review and fast recovery, in which no single gate — human or AI — is expected to catch everything, and autonomy automatically tightens when quality signals fluctuate.
9. A Standard with stable requirements, a single vocabulary and evidence-backed conformance claims, so that every domain profile, starting with the frontend showcase, can prove what it implements.

**Implementation rule:** prove one workflow end-to-end on one repository before building broad centralized infrastructure. Expand only when the next capability solves a problem measured in the pilot.

> A factory cannot rescue a bad process by running it faster [R05]. Make the inner loop reliable first; then scale.
