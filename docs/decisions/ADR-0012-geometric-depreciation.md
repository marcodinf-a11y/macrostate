# ADR-0012: Geometric Depreciation for Capital Stock

**Status:** Accepted

## Context

FR-INV-001 and FR-INV-002 specified that public and private capital "must depreciate over time" but did not specify the depreciation method or formula. This left a key investment mechanic under-specified.

Three main depreciation methods exist:

1. **Geometric (exponential) depreciation:** `K(t+1) = K(t) x (1 - d) + I(t)`. Capital decays at a constant rate per period. Standard in macroeconomic modeling.
2. **Straight-line depreciation:** `K(t+1) = K(t) - D + I(t)`. Fixed amount per period. Standard in accounting, rarely used in macro simulation.
3. **One-hoss shay / vintage capital:** Capital works at full capacity until it dies. More realistic for some equipment but requires tracking individual capital cohorts.

## Decision

**Geometric depreciation** (Godley & Lavoie 2012):

```
K(t+1) = K(t) x (1 - d) + I(t)
```

Where `d` is the depreciation rate per tick and `I(t)` is new investment in period t.

- Public capital: depreciation rates per category (`publicInfrastructure`, `publicServices`) stored in `parameters.json`
- Private capital: depreciation rates per sector (`capitalDepreciationRate`) stored in `sectors.json`

**Rationale:**

- **SFC standard:** Godley & Lavoie (2012) use geometric depreciation in every model with capital (e.g., Model GROWTH, Ch. 11). It is the default in the SFC tradition this project follows.
- **National accounting standard:** The US Bureau of Economic Analysis (BEA) uses geometric depreciation for capital stock estimates.
- **Growth model standard:** Used in the Solow model and virtually all DSGE models.
- **Simple and well-understood:** One parameter per capital category, no cohort tracking needed.

**Alternatives rejected:**

- **Straight-line:** Appropriate for tax/accounting purposes but unrealistic for aggregate capital dynamics. A fixed depreciation amount means the rate varies with capital stock size, producing unusual dynamics.
- **Vintage capital:** More realistic but substantially more complex — requires tracking each investment cohort separately. Can be added post-MVP if needed for specific gameplay scenarios.

## Consequences

- Each sector in `sectors.json` has a `capitalDepreciationRate` parameter (already present).
- Public capital categories in `parameters.json` have depreciation rates (already present).
- Capital stock dynamics are fully determined: investment adds to stock, depreciation erodes it. Zero net investment means gradual decay — the player must maintain spending to preserve capacity.
- Depreciation creates ongoing demand for capital goods (replacement investment), which flows through the manufacturing sector via the Leontief I-O matrix (ADR-0009).

**Affected documents:** PRD (FR-INV-001, FR-INV-002), ECONOMIC-MODEL.md (Public Investment, Private Investment).
