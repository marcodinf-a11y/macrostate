# ADR-0016: Accelerator Investment Function

**Status:** Accepted

## Context

FDR-007 identified that the private investment decision function was left as a placeholder `f(...)` with no specified form. The investment function is one of the most consequential behavioral rules in the simulation — it drives capital accumulation, credit demand, employment in the capital goods sector, and long-run growth dynamics.

In post-Keynesian economics, investment is the key autonomous demand component. The choice of investment function determines whether the model produces realistic growth patterns, credit cycles, and the Keynesian paradox of thrift.

## Decision

**Simple accelerator-profit model** (Fazzari et al. 1988, Godley & Lavoie 2012 Ch. 11):

```
DesiredInvestment = ReplacementInvestment + ExpansionInvestment

ReplacementInvestment = depreciationRate × CurrentCapital
ExpansionInvestment = accelerator × max(0, ExpectedDemand - CurrentCapacity)
```

Where:
- `depreciationRate` is the sector-specific geometric depreciation rate (ADR-0012)
- `accelerator` is a data-driven parameter (e.g., 0.5) controlling how aggressively firms expand
- `ExpectedDemand` comes from the adaptive demand estimation rule (ADR-0015)
- `CurrentCapacity` is the firm's current maximum output given its capital stock

**Funding and rationing:**

```
MaxBorrowing = max(0, creditLimit - existingDebt)
AvailableFunds = RetainedProfits + MaxBorrowing
ActualInvestment = min(DesiredInvestment, AvailableFunds)
```

Internal funds (retained profits) are used first. If insufficient, the firm seeks a bank loan for the shortfall up to its credit limit. If even with borrowing the firm cannot fund desired investment, it invests what it can afford.

**Rationale — why the accelerator model over alternatives:**

- **SFC consistency:** The accelerator is the standard investment function in Godley & Lavoie's SFC models (Ch. 11 GROWTH model). Using the same formulation ensures the investment-saving identity and stock-flow norms behave as expected.
- **Demand-driven:** Investment responds to the gap between expected demand and capacity, not to interest rates or asset prices as primary drivers. This is the core Keynesian insight — investment is driven by expected sales, not by the cost of capital.
- **Credit constraint channel:** The funding/rationing mechanism creates a realistic credit channel. Tight lending standards or high debt levels reduce actual investment below desired investment, producing credit crunches without needing an explicit "financial accelerator" mechanism.
- **Endogenous money linkage:** When investment is bank-funded, the loan creates new deposits. At the macro level, investment creates saving, not the reverse. The funding mechanism makes this MMT/post-Keynesian principle operational.
- **Minimal parameters:** One key parameter (`accelerator`) per sector, plus the credit limit from banking parameters. Easy to calibrate and mod.

**Alternatives rejected:**

- **Tobin's Q:** Investment responds to the ratio of market value to replacement cost of capital. Rejected because (a) there is no equity market in the MVP, so Q cannot be computed, and (b) the empirical performance of Q models is poor (Chirinko 1993). More importantly, Q theory assumes efficient asset markets, which is inconsistent with post-Keynesian epistemology.
- **Kaleckian profit-rate model:** Investment as a function of profit rate and capacity utilization (`g = g_0 + g_1*r + g_2*u`). Considered seriously — this is the standard in neo-Kaleckian growth models. Rejected for MVP because it introduces profit-rate dynamics that interact with income distribution in ways that require careful calibration. The accelerator is simpler and captures the key mechanism (demand drives investment). The Kaleckian form could be offered as a post-MVP alternative via modding.
- **Animal spirits / stochastic investment:** Investment includes a random component reflecting Keynesian uncertainty. Interesting but adds noise that complicates debugging and calibration in early development. Can be added post-MVP as a parameter (e.g., `investmentVolatility`).
- **Interest-rate-sensitive investment (IS-LM style):** `I = I_0 - b*r`. Rejected on theoretical grounds — the interest elasticity of investment is empirically weak (Sharpe & Suarez 2014), and this framing implies loanable funds (investment competes for a fixed pool of saving). In the endogenous money framework, the bank creates the money; the interest rate affects profitability of the investment but is not the primary driver.

## Consequences

- Sector data files gain one parameter: `investmentAccelerator` (e.g., 0.5).
- Investment is tightly coupled to demand estimation (ADR-0015): overestimated demand → excess capacity → no expansion investment. Underestimated demand → capacity shortfall → expansion investment → bank lending → money creation.
- Credit cycles emerge naturally: expansion investment → new loans → higher debt service → profit squeeze → reduced investment. The cycle's amplitude depends on the `accelerator` parameter and banking credit limits.
- The manufacturing sector plays a dual role: it produces consumption goods AND capital goods for all sectors. This creates an interesting inter-sectoral dynamic where a manufacturing boom can be self-reinforcing (investment demand for capital goods boosts manufacturing output, which may trigger further investment).
- Post-MVP, the Kaleckian profit-rate formulation can be offered as an alternative via the modding system (different `investmentFunction` parameter in sector data).

**Affected documents:** ECONOMIC-MODEL.md (Private Investment), ARCHITECTURE.md (Investment phase), PRD (FR-INV linkage).
