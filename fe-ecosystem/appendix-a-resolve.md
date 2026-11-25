[Contents](README.md) · [← 21. References](21-references.md) · [Appendix B — Glossary →](appendix-b-glossary.md)

<a id="appendix-a"></a>

# Appendix A — Resolve pseudo-code

```text
surfaces = surfacesOf(diff) ∪ importGraph(repo).dependents(changedFiles).surfaces
# shared assets: runs in the pipeline of each consumer repo against the prerelease / proposed version (§11.2, §11.3)
# monorepo option (§3.3): portfolioIndex.consumers(...) = projectGraph.dependents(...) within the repo
if repo == platform and diff touches published package:
    surfaces ∪= portfolioIndex.consumers(changedPackages) ∪ tokenUsage.consumers(changedTokens)
if diff touches contract in contract registry:
    surfaces ∪= portfolioIndex.consumers(changedContracts)
if repo == content-models and diff touches schema:
    surfaces ∪= contentGraph.consumers(changedModels)
if any shared asset changed: surfaces ∪= sentinelSet()

layers = [core, domain, organization, repository(repo)]           # Standard §7.3, general → specific
plan = {}; trace = []
for s in surfaces:
    caps = closure(s.capabilities ∪ requiredCapabilities(s.archetypes))
    surfaceLayer = [baseline] + archetypes(s) + packs(caps) + crossCutting(s) + [productProfile(s)]
    for p in layers + surfaceLayer + [taskClass(task), risk(task), task.contract]:
        for c in p.checks where c.when(s):
            if loosens(c, plan[c.id]): reject(p, c.id)                 # no layer loosens another
            merged, rule = merge(plan[c.id], c)   # requirement=max, gate=stricter, ordinal=stricter, set=union
            if rule == incomparable:
                d = recordedResolution(c.id, p)                        # versioned entry in a more specific profile, or ADR
                if d is not none:
                    merged, rule = d.value, decision
                elif blockingResolveEnabled():                         # from Phase 2, or 2027-04-01 if earlier (§8.3, §15.3)
                    block(task, conflict(c.id, s.id, p))               # task → BLOCKED; never resolved silently
                else:                                                  # interim: warn + recorded owner decision
                    warn(task, conflict(c.id, s.id, p)); requestOwnerDecision(c.id, p)
                    merged, rule = c, provisional                      # value of the more specific layer, marked in the trace
            plan[c.id] = merged; trace.append((p.id, p.version, c.id, rule))

for id, e in plan:
    lc = registry.check(id).lifecycle
    if lc in [planned, deprecated]: remove plan[id]
    if lc == shadow: e.enforcement = observe

for w in activeWaivers(surfaces): plan[w.check].waiver = w.id   # expired waivers are ignored

assert required(standardCore) ∪ required(baseline) ⊆ plan   # a profile can never remove Standard core checks
emit plan + versions + verifierRef + resolution(trace)        # resolution trace goes into evidence
```

---

[Contents](README.md) · [← 21. References](21-references.md) · [Appendix B — Glossary →](appendix-b-glossary.md)
