# Parameter Commentary

> **Note:** This file is incremental. Commentary is added as implementation phases are completed and new parameters enter `parameters.json`. Sections below cover only the parameters defined so far.

This document explains the economic rationale behind every value in `parameters.json`. It serves two purposes:

1. **For contributors:** Understand *why* a parameter has a particular value, not just what the value is.
2. **For modders:** Make informed decisions when overriding values via the modding system (FR-MOD-001).

All values listed here are tunable — modders can override any parameter by providing a replacement `parameters.json` in their mod directory. The base game values below represent reasonable defaults grounded in real-world data and macroeconomic research.

As new parameters are added to `parameters.json` in later implementation phases, their commentary should be added here.

## Conversion Note

Parameters expressed as "per tick" rates correspond to monthly values (one tick = one month). To convert between monthly and annual rates:

```
annual = 1 - (1 - monthly)^12
monthly = 1 - (1 - annual)^(1/12)
```

For small rates, `annual ≈ monthly × 12` is a reasonable approximation.

## Investment Parameters

| Parameter path | Value | Unit | Rationale |
|---|---|---|---|
| `investment.capacityUtilizationThreshold` | 0.85 | ratio | Standard macro threshold — firms begin expansion plans when running above ~85% capacity. Below this, idle capacity can absorb demand increases without new investment. Above it, bottlenecks and delivery delays make expansion profitable. Based on Federal Reserve capacity utilization data where investment spending historically accelerates above 80-85%. |
| `investment.depreciationRates.publicInfrastructure` | 0.005 | per tick (monthly) | ~6% annual depreciation. Real infrastructure (roads, bridges, utilities) depreciates at 2-4% annually, but game time is compressed and maintenance neglect accelerates decay. 6% balances realism with gameplay pressure to maintain spending. |
| `investment.depreciationRates.publicServices` | 0.004 | per tick (monthly) | ~4.7% annual depreciation. Public service capital (training programs, institutions) degrades more slowly than physical infrastructure but still requires sustained funding. Lower rate than infrastructure reflects that institutional knowledge persists longer. |


> **Note:** Private sector depreciation rates are per-sector properties defined in `sectors.json` (`capitalDepreciationRate` field), not in `parameters.json`. See MODDING.md for the sector data format.

## Household Population Parameters

> **Note:** Per-class behavioral parameters (AIDS coefficients, consumption propensities, reservation wage multipliers, job search probabilities) are defined in `households.json` and `consumption.json`, not in `parameters.json`. See MODDING.md for the data format. The values below are scenario-level parameters that apply to the household sector as a whole.

| Parameter path | Value | Unit | Rationale |
|---|---|---|---|
| `household.classes.low_income.populationShare` | 0.30 | ratio | Pew Research Center (2024): 30% of US households are lower-income (below 2/3 of median). |
| `household.classes.middle_income.populationShare` | 0.51 | ratio | Pew Research Center (2024): 51% of US households are middle-income (2/3 to 2x median). |
| `household.classes.high_income.populationShare` | 0.19 | ratio | Pew Research Center (2024): 19% of US households are upper-income (above 2x median). |

> **Design note (ADR-0019):** Class membership is fixed for MVP. Income classes are a behavioral parameterization — they determine AIDS parameters and consumption propensities. The engine also tracks income by source (wages, dividends, interest, transfers) for macro indicators (wage share, profit share) and differential tax policy. See ADR-0019 for full architecture.

## Bank Lending Parameters

| Parameter path | Value | Unit | Rationale |
|---|---|---|---|
| `bank.spread` | 0.002 | per tick (monthly) | ~2.4% annual spread over policy rate. Represents bank's base profit margin on lending. US commercial bank net interest margins have averaged 2.5-3.5% in recent decades (FDIC data). |
| `bank.firmDSCRThreshold` | 1.25 | ratio | Standard commercial lending minimum. A DSCR of 1.25 means the firm's net operating income is 25% above total debt service — the most common floor in US commercial lending (Corporate Finance Institute). |
| `bank.householdDTIThreshold` | 0.40 | ratio | 40% debt-to-income threshold for household loans. Sits within the conventional US lending range (36-43%). Below 36% is considered excellent; above 50% is generally unacceptable. _(post-MVP — household borrowing is deferred)_ |
| `bank.householdLoanTerm` | 60 | ticks (months) | 5-year loan term for household installment loans. Balances between short consumer loans (12-36 months) and long mortgages (180-360 months). _(post-MVP)_ |
| `bank.defaultGracePeriod` | 3 | ticks (months) | Number of consecutive ticks with zero income AND zero savings before a household defaults. Provides a short buffer against temporary income loss. _(post-MVP)_ |
| `bank.firmDefaultGracePeriod` | 3 | ticks (months) | Number of consecutive ticks a firm can miss debt service before default is triggered. |
| `bank.maxFirmLoanTerm` | 120 | ticks (months) | Maximum firm loan term (10 years). Firm loan terms are derived from capital useful life (`1/depreciationRate`) but capped at this value. Prevents unreasonably long loans in low-depreciation sectors. Matches the upper end of typical US commercial bank term loans (3-10 years). |

| `policy.lags.spendingChange.min` | 1 | ticks (months) | FR-TIM-001 specifies 1-2 ticks. Minimum reflects rapid reallocation of existing budget authority to departments already executing programs. |
| `policy.lags.spendingChange.max` | 2 | ticks (months) | FR-TIM-001 upper bound. Maximum reflects new spending requiring procurement setup, contract awards, or hiring pipelines. |
| `policy.lags.spendingReallocation.min` | 2 | ticks (months) | FR-TIM-001 specifies 2-3 ticks. Minimum reflects shifting funds between existing functional divisions with established delivery mechanisms. |
| `policy.lags.spendingReallocation.max` | 3 | ticks (months) | FR-TIM-001 upper bound. Maximum reflects reallocation requiring new program design, staffing changes, or procurement redirection. |
| `investment.lags.infrastructureToCapacity.min` | 6 | ticks (months) | FR-TIM-001 specifies 6-12 ticks. Minimum reflects fast-track infrastructure projects (equipment upgrades, minor expansions). |
| `investment.lags.infrastructureToCapacity.max` | 12 | ticks (months) | FR-TIM-001 specifies 6-12 ticks. Maximum reflects major projects (new facilities, transportation infrastructure). Actual lag for each investment is drawn uniformly from [min, max]. |
| `investment.lags.servicesToProductivity.min` | 12 | ticks (months) | FR-TIM-001 specifies 12-24 ticks. Education, training, and institutional improvements take longer to translate into measurable productivity gains. |
| `investment.lags.servicesToProductivity.max` | 24 | ticks (months) | FR-TIM-001 upper bound. Full effect of workforce training and institutional reform. |
| `investment.lags.privateInvestmentToCapacity.min` | 3 | ticks (months) | FR-TIM-002 specifies 3-6 ticks. Firms can expand faster than government — purchasing and installing equipment takes months, not years. |
| `investment.lags.privateInvestmentToCapacity.max` | 6 | ticks (months) | FR-TIM-002 upper bound. Larger expansions (new production lines, facilities) take up to 6 months. |
