# ADR-0018: Two-Layer Government Spending (Economic Flows + Functional Divisions)

**Status:** Accepted

## Context

The PRD (FR-CTL-001, FR-AGT-001) and ECONOMIC-MODEL.md specify three ad-hoc spending buckets: infrastructure, public services, and direct transfers. This design has several problems:

1. **No grounding in economic literature.** The three categories are not drawn from any standard taxonomy — not from SFC models, MMT literature, national accounts, or real-world budget structures.
2. **Mixed abstraction levels.** "Infrastructure" and "public services" are functional purposes (what the spending is for), while "direct transfers" is an economic flow type (how the money moves). These are orthogonal dimensions conflated into one list.
3. **Ambiguous decomposition.** Each functional bucket silently splits into public employment and procurement with no documented ratio (flagged as FDR-016). The player cannot see or control this split.
4. **Hard to extend.** Adding a new spending purpose (e.g., defense, education) requires engine changes rather than data changes.

Real-world national accounts solve this with a cross-classification. Eurostat Table 1100 crosses COFOG functional divisions (10 categories: education, defense, health, social protection, etc.) with ESA economic transaction types (compensation of employees, intermediate consumption, social benefits, gross capital formation, interest). Every EU member state reports this annually. The US federal budget similarly decomposes each budget function into personnel, procurement, grants, and capital.

The empirical data shows that functional divisions have dramatically different economic compositions:

| Function | Compensation | Procurement | Transfers | Capital |
|----------|-------------|-------------|-----------|---------|
| Education | ~80% | ~11% | minimal | ~10% |
| Defense | ~21% | ~20% | minimal | ~2% |
| Healthcare | small | small | ~88% | minimal |
| Social protection | minimal | minimal | dominant | minimal |

These different compositions produce different macroeconomic effects through different SFC flow channels — which is exactly what an MMT simulation should capture.

## Decision

**Replace the three ad-hoc spending buckets with a two-layer architecture:**

### Layer 1: Engine (Economic Flow Types)

The simulation engine knows only about economic flow types, drawn from the SFC/SNA tradition:

- **Compensation of employees (W_g):** Wages and benefits paid to public sector workers. Creates household income directly; absorbs labor from the labor market.
- **Intermediate consumption (G_ic):** Purchases of goods and services from private sector firms. Creates firm revenue and sectoral demand.
- **Social benefits / transfers (TR):** Cash payments to households (pensions, unemployment benefits, welfare). Creates household income with no direct resource claim — recipients spend via the AIDS demand system.
- **Gross capital formation (G_cf):** Government investment in fixed assets (roads, buildings, equipment). Creates sectoral demand similar to intermediate consumption but accumulates as public capital stock.
- **Interest payments (r·B):** Interest on outstanding government bonds. Automatic, not a player choice. Flows to bondholders (banks in MVP).

These are the actual flows in the SFC transaction matrix. The engine computes stock-flow consequences, multiplier effects, and sectoral impacts using these primitives.

**MVP simplification:** Gross capital formation can be merged with intermediate consumption for MVP (both create sectoral demand). This reduces engine flow types to four: compensation, procurement, transfers, interest. Capital formation is separated post-MVP when public capital stock tracking is added.

### Layer 2: Presentation (Functional Divisions)

The player sees and controls functional spending categories, inspired by COFOG but simplified for gameplay. Each functional division is a **data-driven recipe** that decomposes a budget allocation into engine-level flows.

A functional division definition specifies:
- **Employment share:** What fraction goes to public sector wages vs. procurement/transfers
- **Sector targeting:** Which sectors receive procurement demand and from which sectors workers are hired
- **Transfer share:** What fraction flows directly to households

Example definitions (illustrative, not final parameterization):

```
education:
  compensation_share: 0.80
  procurement_share: 0.10
  capital_share: 0.10
  transfer_share: 0.00
  employment_sectors:
    services: 0.90
    manufacturing: 0.10
  procurement_sectors:
    manufacturing: 0.60
    services: 0.40

defense:
  compensation_share: 0.25
  procurement_share: 0.55
  capital_share: 0.15
  transfer_share: 0.05
  employment_sectors:
    services: 0.70
    manufacturing: 0.30
  procurement_sectors:
    manufacturing: 0.60
    construction: 0.20
    services: 0.20

social_protection:
  compensation_share: 0.05
  procurement_share: 0.02
  capital_share: 0.00
  transfer_share: 0.93
  employment_sectors:
    services: 1.00
  procurement_sectors:
    services: 1.00
```

The presentation layer performs the decomposition: player sets $100B for education → presentation layer computes $80B compensation (mapped to sectors), $10B procurement (mapped to sectors), $10B capital formation (mapped to sectors) → these flow-type amounts are passed to the engine.

### MVP Functional Divisions

For MVP (4 sectors: Agriculture & Primary, Manufacturing, Construction, Services), a minimal set of functional divisions:

1. **Public services** (education, health, administration bundled) — high employment share
2. **Infrastructure** — high procurement/capital share, targets construction and manufacturing
3. **Social transfers** — nearly pure transfers
4. **Defense** — balanced employment/procurement

Post-MVP, these can be unbundled (education, health, and administration become separate divisions) as sub-sectors are added, without engine changes.

### Interaction with Policy Levers

The player controls:
1. **Total government spending level** (the budget)
2. **Allocation across functional divisions** (percentage sliders)
3. **Tax rate**

The decomposition into engine flows is automatic and transparent — the player can inspect how their functional allocation translates into employment, procurement, and transfers, but does not need to manage that split directly.

**Alternatives considered:**

- **Single G variable (textbook macro):** Too abstract for a game. Players need meaningful policy levers.
- **Direct flow-type controls (player sets compensation/procurement/transfers directly):** Economically clean but unintuitive. Real governments budget by function, not by economic type. Players think "spend more on education," not "increase compensation of employees in the services sector."
- **Keep three ad-hoc buckets:** No theoretical grounding, mixed abstraction levels, hard to extend. Rejected for the reasons in Context.
- **Full COFOG (10 divisions):** Too granular for MVP with only 4 sectors. The simplified set captures the key economic distinctions (employment-heavy vs. procurement-heavy vs. transfer-heavy) while remaining manageable.

## Consequences

- **Engine stays clean.** It processes flow types (compensation, procurement, transfers, capital formation, interest) without knowing what "education" or "defense" means.
- **Data-driven extensibility.** New functional divisions are added as data files, not code changes. Different scenarios (US, Sweden, developing country) can have different functional mixes.
- **Moddable.** Modders define new functional divisions by specifying decomposition ratios.
- **Testable.** Engine tests use raw flow types directly. Presentation layer tests verify decomposition arithmetic.
- **Grounded in real-world practice.** The two-layer structure mirrors Eurostat Table 1100 (COFOG × ESA economic type), which every EU member state reports annually.
- **Resolves A15.** The relationship between spending allocation and public employment costs is now explicit: each functional division declares its compensation share, and the engine receives the computed wage bill directly.
- **Resolves FDR-016.** The default wage/procurement split is no longer an undocumented assumption — it is an explicit parameter in each functional division definition.
- **PRD and ECONOMIC-MODEL.md must be updated** to replace the three-bucket model with this two-layer architecture.

**Affected documents:** PRD (FR-AGT-001, FR-CTL-001), ECONOMIC-MODEL.md (Government section), GAME-DESIGN.md (spending controls), PARAMETERS-COMMENTARY.md (functional division definitions).
