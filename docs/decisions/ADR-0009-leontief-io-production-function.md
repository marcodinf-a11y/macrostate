# ADR-0009: Leontief Input-Output Production Function

**Status:** Accepted

## Context

The sector table in ECONOMIC-MODEL.md listed inputs and outputs for each sector (e.g., Agriculture needs labor and land to produce food and raw materials) but never specified the mathematical form of the production function. The `inputWeights` in `sectors.json` could be interpreted as either Cobb-Douglas exponents or Leontief coefficients, with very different economic implications.

This was flagged as critical finding F3 in the MMT accuracy review. The production function form determines how the economy responds to input changes and embeds fundamental assumptions about wage determination and resource allocation.

## Decision

Use a **Leontief Input-Output** production function (fixed proportions with inter-sector linkages).

Each sector has fixed technical coefficients — producing one unit of output requires fixed quantities of each input, with no substitution between inputs:

```
Output_s = min(Input_1 / a_s1, Input_2 / a_s2, ..., Labor / a_sL)
```

Inter-sector dependencies are captured by an input-output matrix specifying how much of each sector's output is required per unit of production in every other sector. Coefficients are data-driven, loaded from `sectors.json`.

**Rationale — alternatives considered:**

- **Cobb-Douglas** (smooth substitution, σ=1): Rejected. Implies that capital can freely replace labor, which underpins neoclassical marginal productivity theory — the claim that wages equal marginal product. This is theoretically incompatible with MMT/post-Keynesian economics, where wages are determined by bargaining power, institutional factors, and labor market conditions.
- **CES** (constant elasticity of substitution): Generalizes both Leontief (σ=0) and Cobb-Douglas (σ=1). Empirical estimates of the elasticity of substitution consistently fall in the 0.4-0.7 range (Chirinko 2008, Oberfield & Raval 2021), much closer to Leontief than Cobb-Douglas. A low-σ CES would be more empirically precise but adds a calibration parameter per sector without meaningful gameplay difference.
- **Capacity utilization (Kaleckian)**: Firms have maximum capacity and adjust utilization to demand. Viable and consistent with post-Keynesian theory, but Leontief I-O gives richer inter-sector dynamics (supply chain propagation).
- **Plain Leontief** (without I-O matrix): Simpler but misses inter-sector linkages. The sector table already implies these linkages (Manufacturing uses raw materials from Agriculture), so an explicit I-O matrix makes them first-class.

Leontief I-O is the simplest model consistent with both the empirical evidence and the Godley/Lavoie (2012) SFC modeling tradition.

## Consequences

- `inputWeights` in `sectors.json` are Leontief technical coefficients, not Cobb-Douglas exponents.
- An inter-sector input-output matrix defines how sectors consume each other's output as intermediate inputs.
- Supply shocks propagate through the matrix — a disruption in Agriculture constrains Manufacturing, which constrains Construction.
- Government procurement enters through the I-O matrix — infrastructure spending creates demand in Construction, which pulls from Manufacturing.
- Coefficients are fixed in the short run. Long-run technological change (coefficient drift) is a post-MVP enhancement.
