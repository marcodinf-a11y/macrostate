# ADR-0015: Adaptive Demand Estimation

**Status:** Accepted

## Context

FDR-006 identified that no document specified how firms estimate demand for production planning. This is a core behavioral rule — it determines production quantities, which in turn drive employment, input purchases, inventory accumulation, and pricing dynamics. Without a demand estimation rule, the production decision has no basis.

The choice of expectation formation mechanism is one of the most consequential modeling decisions in an agent-based macroeconomic simulation. It determines whether the model exhibits realistic business cycle dynamics (overproduction → inventory buildup → contraction) or implausible perfect foresight.

## Decision

**Adaptive expectations with trend extrapolation** (Caiani et al. 2016):

```
ExpectedDemand_t = PreviousSales_t-1 × (1 + trend_t)
trend_t = trend_t-1 + adaptationSpeed × (salesGrowth_t-1 - trend_t-1)
salesGrowth_t-1 = (PreviousSales_t-1 - PreviousSales_t-2) / PreviousSales_t-2
```

Where:
- `PreviousSales` is the firm's actual sales (quantity sold) in the previous tick
- `trend` is a smoothed estimate of sales growth, updated each tick via partial adjustment
- `adaptationSpeed` is a data-driven parameter (e.g., 0.25) controlling how quickly firms revise their expectations

**Production target** links demand estimation to inventory management:

```
ProductionTarget = ExpectedDemand + max(0, TargetInventory - CurrentInventory)
TargetInventory = ExpectedDemand × TargetInventoryRatio
```

**Edge cases:**
- `PreviousSales = 0` (new firm or total demand collapse): set `ExpectedDemand = InitialDemandEstimate` (per-sector data parameter)
- `PreviousSales_t-2 = 0` (only one tick of history): set `salesGrowth = 0` (no trend signal)

**Rationale — why adaptive expectations over alternatives:**

- **Bounded rationality:** Firms use simple, backward-looking heuristics rather than solving optimization problems. This is consistent with the post-Keynesian view of fundamental uncertainty — firms cannot know future demand, so they extrapolate from experience.
- **Self-correcting feedback:** Overestimation → inventory buildup → markup falls (via ADR-0011) → prices drop → demand recovers. Underestimation → inventory depletion → markup rises → prices increase → demand moderates. The demand estimation error is the driver of the inventory-markup feedback loop.
- **ABM canon:** Adaptive expectations are the standard approach in the Caiani et al. (2016) benchmark model and most post-Keynesian ABMs. Using the same mechanism ensures comparability with the literature.
- **Minimal parameters:** Only one tunable parameter (`adaptationSpeed`) per sector. Simple to calibrate and mod.

**Alternatives rejected:**

- **Rational expectations:** Firms solve for the model-consistent forecast. Rejected on theoretical grounds — inconsistent with post-Keynesian epistemology (fundamental uncertainty). Also computationally expensive and would eliminate the inventory cycle dynamics that make the simulation interesting.
- **Fixed/naive expectations (`ExpectedDemand = PreviousSales`):** Too simple. No trend component means firms never anticipate growth or contraction, leading to permanently lagging production in growing economies. Used in some SFC models but produces unrealistic step-response dynamics.
- **Weighted moving average of N periods:** Considered. Smoother than single-period lookback but requires storing N periods of sales history per firm and introduces a window-length parameter with no clear empirical basis. The trend-extrapolation approach achieves smoothing via the `adaptationSpeed` parameter without additional state.
- **Chartist/fundamentalist switching (Dosi et al.):** More sophisticated — firms switch between extrapolative and mean-reverting strategies. Appropriate for models with firm heterogeneity and evolutionary selection, but over-engineered for the current architecture where firms within a sector are aggregated.

## Consequences

- Sector data files gain two parameters: `demandAdaptationSpeed` (e.g., 0.25) and `initialDemandEstimate` (startup expectations for new firms or demand-collapse recovery).
- The demand estimation → inventory → markup feedback loop is now fully specified end-to-end (this ADR + ADR-0011).
- Business cycle dynamics emerge from estimation errors: systematic over/underestimation drives inventory cycles, which drive price cycles, which drive real demand cycles.
- The `adaptationSpeed` parameter is moddable — higher values make firms more reactive (shorter cycles, more volatile), lower values make them more sluggish (longer cycles, smoother).

**Affected documents:** ECONOMIC-MODEL.md (Demand Estimation), ARCHITECTURE.md (Production phase), PRD (FR-PRC-001 linkage).
