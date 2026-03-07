# ADR-0010: Asymmetric Markup Adjustment and Seller's Inflation

**Status:** Accepted

## Context

The pricing model in ECONOMIC-MODEL.md used a symmetric demand adjustment factor for markups: markups rise when demand exceeds capacity and fall when there is slack, at the same speed in both directions. This was flagged as medium finding F4 in the MMT accuracy review, identifying two gaps:

1. **Symmetric adjustment is unrealistic.** Empirically, firms raise markups quickly but resist lowering them — the price equivalent of downward wage rigidity (which the model already includes). Gas prices spike overnight when oil rises but take weeks to come back down.

2. **No supply-side markup channel.** The model could only produce demand-pull inflation. Weber & Wasner (2023) showed that during 2021-2023, firms raised profit margins even without excess demand, using supply disruptions as cover to widen markups. Post-Keynesian conflict theory (Rowthorn 1977, Lavoie 2014) models inflation as a distributional conflict between workers and firms — markup expansion by firms with market power is a primary inflation source.

Without these mechanisms, the simulation's inflation dynamics are incomplete: inflation is too responsive to demand-side policy and has no "corporate greed" component.

## Decision

**1. Asymmetric markup adjustment speeds.**

Markups adjust at different speeds in each direction:
- Upward: fast (e.g., speed factor 0.5) — firms exploit pricing power quickly
- Downward: slow (e.g., speed factor 0.1) — firms resist margin compression

Per-sector parameters (`markupUpwardSpeed`, `markupDownwardSpeed`) loaded from `sectors.json`.

**2. Supply-side markup pressure (seller's inflation).**

When input availability drops, firms raise markups independently of demand:

```
Supply pressure factor = Normal input availability / Actual input availability
Markup = Base markup x Demand adjustment x Supply pressure factor
```

The supply pressure factor uses the same asymmetric adjustment speeds — it ratchets up fast but decays slowly as supply normalizes.

**Rationale — alternatives considered:**

- **Symmetric adjustment only:** Rejected. Contradicts empirical pricing behavior and makes inflation too easy to resolve with demand management alone.
- **Stochastic markup shocks:** Random markup increases would produce inflation events but without causal grounding. Supply pressure ties markup expansion to observable input scarcity, making the mechanism legible to players.
- **Explicit oligopoly/market power model:** A full market structure model with concentration indices driving markup behavior would be more realistic but adds substantial complexity. The supply pressure factor captures the key insight (firms exploit scarcity) without modeling market structure directly. Market power differentiation can be added post-MVP via per-sector parameters.

## Consequences

- `sectors.json` gains two new per-sector parameters: `markupUpwardSpeed` and `markupDownwardSpeed`.
- Supply shocks now cause persistent inflation that does not resolve just by cutting demand — the player must address the supply side or wait for the slow markup decay.
- The Leontief I-O matrix (ADR-0009) provides the input availability data that feeds the supply pressure factor — the two mechanisms compose naturally.
- The three inflation buffers — (1) productivity gains, (2) demand slack / idle capacity, and (3) profit margin compression (see [ECONOMIC-MODEL.md, Three Buffers Against Inflation](../design/ECONOMIC-MODEL.md#three-buffers-against-inflation)) — remain intact; seller's inflation is an additional inflation source, not a replacement.
