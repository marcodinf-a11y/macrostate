# ADR-0011: Inventory-Based Demand Adjustment for Markup Pricing

**Status:** Accepted

## Context

FR-PRC-001 specified that markups adjust "based on demand relative to capacity" but did not define the functional form for the demand adjustment factor. The previous description in ECONOMIC-MODEL.md used vague capacity utilization language ("when demand > capacity: markup increases"). This left the core pricing mechanism under-specified — different implementations could produce very different inflation dynamics.

Two main approaches exist in the literature:

1. **Capacity-utilization-based (Kaleckian tradition):** `DemandAdjustmentFactor = CapacityUtilization / CapacityThreshold`. Firms respond to how full their production capacity is. Standard in Kaleckian growth models (Dutt 1984, Rowthorn 1981).

2. **Inventory-based (ABM tradition):** `DemandAdjustmentFactor = TargetInventoryRatio / ActualInventoryRatio`. Firms respond to whether their inventories are depleting or piling up. Standard in modern agent-based SFC models (Caiani et al. 2016, Seppecher 2012).

## Decision

**Inventory-based demand adjustment** (Caiani et al. 2016):

```
DemandAdjustmentFactor = TargetInventoryRatio / ActualInventoryRatio
```

- When inventories deplete (high demand): factor > 1, markup rises
- When inventories at target: factor = 1, markup at base level
- When inventories pile up (low demand): factor < 1, markup falls

This feeds into the full markup formula (with ADR-0010 asymmetric adjustment):

```
TargetMarkup = BaseMarkup x DemandAdjustmentFactor x SupplyPressureFactor
CurrentMarkup += speed x (TargetMarkup - CurrentMarkup)
```

`TargetInventoryRatio` is a per-sector parameter loaded from `sectors.json`.

**Rationale — why inventory-based over capacity-utilization:**

- **Directly observable:** Firms observe their own inventory levels; capacity utilization is an abstract aggregate statistic that individual firms don't compute.
- **Connects to existing mechanics:** The model already has firms holding inventory of unsold goods. Using inventories as the pricing signal creates a natural feedback loop between production, sales, and pricing.
- **Standard in ABM-SFC literature:** Caiani et al. (2016) is the benchmark SFC agent-based model published in the *Journal of Economic Dynamics and Control*. This game's architecture (agents, sectors, SFC accounting) is closer to ABMs than to aggregate Kaleckian models.
- **Works at all scales:** Functions with one representative firm per sector (MVP) and with multiple competing firms (full vision). Capacity utilization becomes ambiguous with firm heterogeneity; each firm's inventory is always well-defined.

## Consequences

- `sectors.json` gains a new per-sector parameter: `targetInventoryRatio`.
- The pricing engine computes `ActualInventoryRatio = CurrentInventory / RecentSales` each tick.
- Inventory dynamics become central to price determination — production overshoots cause price declines, demand spikes cause price increases, with the asymmetric speeds from ADR-0010 governing adjustment rates.
- The three inflation buffers (ECONOMIC-MODEL.md) remain intact — "demand slack" is now precisely defined as inventory above target.

**Affected documents:** ECONOMIC-MODEL.md (Pricing Model), ARCHITECTURE.md (Section 3.10 Pricing Engine), PRD (FR-PRC-001).
