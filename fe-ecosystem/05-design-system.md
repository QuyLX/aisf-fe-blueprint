[Contents](README.md) · [← 4. Technical foundation](04-technical-foundation.md) · [6. Patterns by product →](06-product-patterns.md)

<a id="s5"></a>

# 5. Design System

<a id="s5-1"></a>

## 5.1. Three tiers

- **Foundation:** tokens (DTCG format → CSS variables + typed TS constants), typography, spacing, color, motion, icons, a11y primitives, core controls.
- **Experience patterns:** Console UI (resource table, filter, status, LRO progress, error taxonomy, destructive confirm), Content UI, Marketing UI, Docs UI, Chat UI (message list, composer, streaming bubble).
- **Product extensions:** domain-specific components, located in modules, not in the DS.

<a id="s5-2"></a>

## 5.2. Governance

Every public component/token has: an owner, a public API, a **Contract Spec** ([§5.4](#s5-4)), an a11y contract, supported themes/modes, SemVer, deprecation with a codemod, stories + tests. Semantic color pairs are **checked for contrast automatically** for every theme/mode.

<a id="s5-3"></a>

## 5.3. Agent-ready

1. Typed docs placed next to the component (`Button.doc.ts`) and released with every version; agents read the docs of **exactly the version in the repo's lockfile**. The canonical source and the drift-prevention mechanism are in [§5.6](#s5-6).
2. A Storybook components manifest + MCP (or DS CLI) lets agents look up props/examples, find affected stories and run tests.
3. Uniform conventions (prop names, slots, composition) that are **measured** by the agent-legibility eval.
4. A breaking change is released with a codemod; once consumer verification against the prerelease passes, a bot opens an adoption PR with the codemod to every consumer repo ([§11.2](11-delivery-change-management.md#s11-2)).

<a id="s5-4"></a>

## 5.4. Contract Spec (Knowledge Record `kind: component`)

| Item | Content |
|---|---|
| Ownership boundary | What is owned / not owned (and by whom) |
| Public concepts | Closed values/states, defaults, stability, behavior with invalid values |
| Behavioral contract | Invariants with an ID and a **basis** |
| Allowed variation | Variations that do not count as a regression |
| A11y contract | Linked to an **a11y pattern contract** (button, dialog, tabs, menu, grid, combobox…) — written once per pattern, traced to WCAG/APG, with a jsdom lane and a real-browser lane |
| Theming anatomy | Which part receives which token; the `unsettled` part is a **human decision** |
| Verification map | Clause → check, with "what goes wrong if this behavior is removed" |
| Decision log / Open questions | `DEC-n`; `OQ-n` labeled `checkable` / `human-design` / `human-api` |

<a id="s5-5"></a>

## 5.5. Impact analysis for tokens/components

- **Consumer graph** = portfolio index (descriptor + lockfile of every repo) + import analysis (components) + **Token Usage Index** (static scan + alias resolution + build-output scan), each edge carrying a confidence level. *Monorepo option ([§3.3](03-target-architecture.md#s3-3)): the project graph replaces the descriptor + lockfile part; the Token Usage Index is still needed.*
- Lint rule FE-DS-001 keeps `unknown-dynamic` ≈ 0.
- **Sentinel set:** always verify a representative set of surfaces (top critical per archetype × theme), regardless of what the graph says.
- **Consumers outside the portfolio** (embedded widgets, SDKs, public CSS): the graph cannot see them; safety relies on a compatibility policy (SemVer, support period, advance deprecation notice).

<a id="s5-6"></a>

## 5.6. Component documentation: canonical source, versioning and drift prevention

Agents generate UI based on component documentation, so wrong documentation produces wrong UI. Each kind of component information has **exactly one canonical source**, placed next to the code in the platform repo; everything else is a view generated from that source.

| Information | Canonical source | Generated view |
|---|---|---|
| Public API (props, slots, events, types) | Exported TypeScript types of the package | Props tables in Storybook and the DS site; `agent-manifest.json` |
| Usage, do/don't | `*.doc.ts` (typed, references props by typed name) | DS site, Storybook docs, MCP/DS CLI |
| Examples | Stories (`*.stories.tsx`); every story is also an interaction test | Examples on the DS site and in the manifest |
| Behavior, a11y, theming | Contract Spec ([§5.4](#s5-4), Knowledge Record `kind: component`) | "Behavior" section on the DS site; verification map |
| Changes | Changeset + codemod | Changelog, migration guide |

**Versioning in sync with the package.** Docs, manifest and story index are built in the same release as the package and **bundled inside the package itself** (`dist/agent-manifest.json`), so agents read docs matching the version in the lockfile without needing the network. The DS site keeps docs for every version within the support window ([§4.8](04-technical-foundation.md#s4-8)) and displays the latest version by default; the Storybook MCP is configured per the version the repo uses.

**Detecting docs and examples that drift from the public API** (checks in the `design-system` profile):

- `ds.docs-api-sync`: the props tables in docs and the manifest are regenerated from types and compared; any difference fails. Every public prop must have a description. A `@deprecated` prop must be marked in the docs with its replacement.
- `docs.code-samples`: examples in `*.doc.ts` and MDX are type-checked and built against the exact package version; an example that does not compile fails.
- `ds.story-interaction`: stories run and interactions pass, so examples are always executable code.
- Every clause in a Contract Spec's verification map points to an existing check; a clause without a check is a `factory doctor` finding.
- The agent-legibility eval ([§12](12-evaluation.md#s12)) measures whether agents use components correctly per the docs; a poor result is a signal that the docs need fixing, not that a prompt needs patching.

---

[Contents](README.md) · [← 4. Technical foundation](04-technical-foundation.md) · [6. Patterns by product →](06-product-patterns.md)
