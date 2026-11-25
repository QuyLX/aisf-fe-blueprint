[Contents](README.md) · [← 15. Roadmap](15-roadmap.md) · [17. Decisions to make (Phase 0) →](17-decisions-to-make.md)

<a id="s16"></a>

# 16. Current state: problems and data, not constraints

The design makes no concessions to the current state. Existing systems are used in three ways.

<a id="s16-1"></a>

## 16.1. Evidence of the problem

See [§0.1](00-summary.md#s0-1) (P1–P6). Every major design decision MUST identify which problem it solves.

<a id="s16-2"></a>

## 16.2. Calibration data for the Factory

The new platform has no history yet; the history of existing systems fills this gap:
- **Risk classifier:** label risk on a sample of historical PRs (suggested ≥ 100) → calibration set for classifier v0 (Phase 1a exit).
- **History replay:** fixed bugs → eval cases; keep only cases with tests that check observable behavior.
- **Seeded defects and scenarios:** real defects that have occurred (permission, tenancy, XSS, CSV, SLA) → seeded defects and scenario fixtures.
- **Journeys:** critical journeys of running products → initial Journey Registry ([§7.7](07-factory-metadata-registry.md#s7-7)).
- **Reviewer eval set:** PRs that caused incidents or reverts (bug location confirmed by engineers who know the code) and known-clean PRs become the incident-derived and known-clean sets of Standard §7.12.
- **Baseline metrics** (Standard §13.4): lead time, human review time, change-failure rate and revert rate of existing products, measured before Slice 0 so the showcase can show a measurable difference.
- **Content:** existing content → input for content model design and for testing migration scripts.

<a id="s16-3"></a>

## 16.3. Migration principles

1. **Strangler by route:** the edge/CDN routes migrated paths to the new app, and the rest to the old system; share the session where possible.
2. **Characterization first:** E2E for the old system's critical journeys as parity evidence.
3. **No further development on EOL frameworks** — rebuild along the golden path in a generator-created repo. Code already on the target stack (Next.js/React) is **upgraded to the standard in place**: add the descriptor, presets and mandatory pipeline, starting at L1; existing violations go into the ratchet baseline (using the tool's native suppressions where available, e.g., ESLint bulk suppressions); `baselineable: false` checks are never ratcheted.
4. **Prioritize** by risk (EOL framework, admin, billing), then by change frequency.
5. **Old systems only receive bug fixes and security patches** once the corresponding surface has a migration schedule; new features are built on the platform.
6. **CMS content** is migrated with scripts verified against the new content model; redirects for old URLs.
7. **Shut down the old system** when the last route has been migrated and evidence parity is reached.

---

[Contents](README.md) · [← 15. Roadmap](15-roadmap.md) · [17. Decisions to make (Phase 0) →](17-decisions-to-make.md)
