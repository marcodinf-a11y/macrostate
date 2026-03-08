# ADR-0019: Household Income Classification Architecture

**Status:** Accepted

## Context

The PRD (FR-AGT-004) specifies three household income classes (low, middle, high) with distinct AIDS parameters and consumption propensities, but leaves key questions unresolved (prd-review A13):

1. **Population counts** — How many households per class?
2. **Class ratios** — What fraction of the population in each class?
3. **Reclassification** — Can households move between classes?
4. **Income tracking** — How is income structured per household?

A literature review of SFC and ABM macroeconomic models reveals two established patterns:

- **Aggregate SFC models** (Dos Santos & Zezza 2008): Two *functional* types — workers (wage income, consume all) and rentiers (profit/interest income, save). No population counts; the split is by income source, not headcount.
- **Agent-based models** (EURACE, K+S, JAMEL): Individual agents in a single class. Heterogeneity in income and wealth emerges endogenously from labor market matching, skill dynamics, and wealth accumulation. No pre-defined income brackets.

No surveyed model uses pre-defined income-level classes with distinct behavioral parameters. Our 3-class design is a deliberate game design choice — it makes distributional impacts legible to the player and enables class-differentiated consumption (AIDS). This ADR grounds that choice in the literature while keeping the engine architecture standard.

## Decision

### Engine: granular income tracking per household

The engine tracks every household's income by source, not by class:

- **Wage income** — from employment (firm or government)
- **Dividend income** — from equity holdings in firms/bank
- **Interest income** — on bank deposits
- **Transfer income** — government transfers (unemployment benefits, etc.)

This is standard SFC accounting. Every model surveyed tracks these flows. The engine computes:

```
Gross income = wages + dividends + interest + transfers
Disposable income (YD) = gross income - taxes - debt service
```

Tax policy can apply different rates to wage vs capital income (dividends + interest), enabling a real-world policy lever without requiring the engine to classify households.

### Behavioral layer: income classes determine consumption parameters

Each household is assigned to an income class (low, middle, high) that determines:

- AIDS parameters (alpha, beta, gamma) — sector allocation of consumption
- Consumption function propensities (alpha1, alpha_f, alpha_h) — total expenditure from income and wealth
- Reservation wage multiplier
- Job search probability

Income class is a **behavioral parameterization**, not just an analytical view. It affects how households consume and interact with the labor market.

### Analytical indicator: functional composition

The engine computes a **functional composition indicator** per household each tick:

```
Capital income share = (dividends + interest) / gross income
```

This is not a behavioral classification — it does not affect household behavior. It serves two purposes:

1. **Macro diagnostics for the player**: Aggregate wage share vs profit share, a central indicator in MMT/Kaleckian economics (rising profit share → income shifts to higher-saving households → demand drag).
2. **Tax policy foundation**: Enables differential taxation of wage vs capital income.

### Population and class assignment

- **Total household count**: Scenario parameter loaded from data files. Represents the number of representative household groups, not literal population.
- **Class population shares**: Scenario parameter loaded from `households.json`. For the US MVP scenario, calibrated from Pew Research Center (2024): 30% low income, 51% middle income, 19% high income.
- **Initial conditions** (income, wealth per class): Scenario parameters loaded from data files, calibrated from US Census CEX and Federal Reserve SCF data.
- **Class membership for MVP: fixed.** Households do not reclassify during simulation. This is consistent with every SFC and ABM model surveyed — none implement income-threshold reclassification.

### Post-MVP: dynamic reclassification

Post-MVP, households may reclassify based on rolling average total income crossing thresholds defined in scenario data. Design constraints for future implementation:

- **Hysteresis**: Must exceed threshold by a margin AND sustain for N ticks before reclassifying (prevents oscillation at boundaries).
- **Propensity transition**: AIDS parameters shift to the new class upon reclassification (no blending — discrete switch, consistent with the class-based parameterization).
- **Empirical anchor**: US Treasury income mobility data shows ~94-96% of households stay in their income class per year, providing a calibration target for threshold/hysteresis tuning.

## Consequences

### Positive

- **Engine stays standard.** Granular income tracking by source is basic SFC accounting — no novel mechanisms.
- **Grounded in literature.** Functional income decomposition (wage vs capital) follows Kaldor (1956), Dos Santos & Zezza (2008). Income classes with distinct consumption parameters extend this with empirical calibration from CEX/Pew data.
- **Player-legible.** Three income classes make distributional policy impacts visible. Wage share vs profit share provides macro-level feedback.
- **Moddable.** Population shares, income thresholds, and per-class parameters are all in data files.
- **Tax policy ready.** Income tracked by source enables wage vs capital income taxation without additional engine changes.

### Negative

- **Novel relative to literature.** No existing SFC or ABM model uses income-level classes with distinct AIDS parameters. Calibration will require judgment, not just literature replication.
- **Discrete class boundaries.** Households near a threshold behave identically to those well within the class but differently from those just across the boundary. This is a simplification accepted for legibility.
- **Fixed classes limit dynamics (MVP).** A household that loses its job and income stays in its original class for MVP. Post-MVP reclassification addresses this.

## References

- Dos Santos, C.H. & Zezza, G. (2008). "A simplified benchmark stock-flow consistent post-Keynesian growth model." *Metroeconomica*, 59(3):441-478.
- Kaldor, N. (1956). "Alternative theories of distribution." *Review of Economic Studies*, 23(2):83-100.
- Dawid, H. et al. (2019). "Macroeconomics with heterogeneous agent models." *Journal of Evolutionary Economics*, 29:39-71. [EURACE]
- Dosi, G. et al. (2018). "Causes and consequences of hysteresis." OFCE Working Paper n.27. [K+S]
- Seppecher, P., Salle, I. & Lavoie, M. (2018). "What drives markups?" *Industrial and Corporate Change*, 27(6):1045-1067. [JAMEL]
- Caiani, A. et al. (2016). "Agent-based stock-flow consistent macroeconomics." *JEDC*, 69:375-408. [JMAB]
- Pew Research Center (2024). "Are You in the American Middle Class?"
- US Federal Reserve, Survey of Consumer Finances (2022).
- US Treasury, Income Mobility Study (2008).
