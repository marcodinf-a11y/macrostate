# ADR-0013: Godley & Lavoie Wage Equation

**Status:** Accepted

## Context

FR-LBR-001 specified that wages must be "sticky downward" and "influenced by sector conditions, labor scarcity, firm profitability" but did not define a functional form. The ambiguity left open whether wages should use a market-clearing mechanism (neoclassical), a partial adjustment model, or a Phillips curve variant.

The choice of wage equation is consequential — it determines how labor costs feed into the pricing engine, how fiscal policy affects the real economy, and whether the model's inflation dynamics are consistent with post-Keynesian theory.

## Decision

**Godley & Lavoie (2012) wage equation** (Model GROWTH, Ch. 11):

```
DW/W = O_0 + O_1 x (DPr/Pr) - O_2 x UR + O_3 x pi
```

Where:
- `O_0` = autonomous wage growth (institutional baseline, e.g., minimum wage policy)
- `O_1 x (DPr/Pr)` = productivity growth pass-through (workers claim a share of productivity gains)
- `O_2 x UR` = unemployment rate dampens wage growth (weaker bargaining power when jobs are scarce)
- `O_3 x pi` = inflation catch-up (workers try to maintain real wages)

Omega coefficients are per-sector parameters loaded from data files. Government wages use distinct coefficients reflecting civil service stickiness (slower adjustment).

**Rationale — why Godley & Lavoie over alternatives:**

- **SFC canon:** This equation is from the same modeling tradition as the rest of the project (SFC, post-Keynesian). Internal consistency matters.
- **Not market-clearing:** Wages reflect bargaining outcomes, not marginal productivity. This is a core post-Keynesian position — wages are determined by power, institutions, and labor market conditions, not by the intersection of labor supply and demand curves.
- **Downward rigidity is emergent:** With reasonable Omega calibration, wage growth approaches zero in recessions but rarely goes negative. No separate "downward stickiness" mechanism is needed — it falls out of the math.
- **Clear economic meaning:** Each coefficient has a direct interpretation (productivity sharing, unemployment pressure, inflation expectations). This makes calibration transparent and moddable.

**Alternatives rejected:**

- **Asymmetric partial adjustment toward target wage:** Considered during review. Rejected because it requires defining "target wage" (which reintroduces the G&L variables anyway), adds an unnecessary layer of complexity, and forces wages into the same functional form as markup adjustment — but wages and markups are different economic processes.
- **Caiani et al. (2016) simplified wage rule:** `W(t) = W(t-1) x (1 + h x Du/u)`. Simpler but loses the productivity and inflation channels, which are important for realistic wage-price dynamics.
- **Dosi et al. (2015) indexation:** Similar inputs to G&L but with firm-level indexation weights. More appropriate for models with firm heterogeneity; the G&L formulation works at the sector level which matches the current architecture.

## Consequences

- Sector data files gain four Omega coefficients per sector: `wageAutonomousGrowth`, `wageProductivityPassthrough`, `wageUnemploymentSensitivity`, `wageInflationCatchup`.
- Government sector data gains its own Omega coefficients (smaller values = slower adjustment = civil service stickiness).
- Wage-price spiral dynamics emerge naturally: wage growth feeds into unit labor costs (ULC), which feed into markup pricing (FR-PRC-001), which feeds back into inflation (pi), which feeds back into wage demands via O_3. The spiral's intensity depends on Omega calibration.
- No separate "wage floor" or "downward rigidity" parameter is needed.

**Affected documents:** PRD (FR-LBR-001), ECONOMIC-MODEL.md (Wage Dynamics), ARCHITECTURE.md (Labor Market).
