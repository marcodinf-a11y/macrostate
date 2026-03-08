# Economic Model

This document defines the simulation's economic model in detail. The model is grounded in Modern Monetary Theory (MMT) and post-Keynesian economics, using a Stock-Flow Consistent (SFC) framework.

## Foundational Framework: Stock-Flow Consistency (SFC)

The simulation uses double-entry bookkeeping for the entire economy. Every financial transaction has an explicit source and destination. Money never appears from nowhere or vanishes — except through two paired channels (creation and its reverse):

1. **Government spending** — creates new currency (central bank reserves + bank deposits)
2. **Taxation** — destroys currency (removes deposits + reserves from circulation)
3. **Bank lending** — creates new deposit money (and a corresponding debt)
4. **Loan repayment** — destroys deposit money (and extinguishes corresponding debt)

### Institutional Sectors and Balance Sheet Matrix

The engine maintains separate balance sheets for six institutional sectors, following the standard SFC transaction flow matrix structure (Godley & Lavoie 2012, Ch. 6):

1. **Households** — hold deposits, owe loans
2. **Firms** — hold deposits and capital goods, owe loans
3. **Government (Treasury)** — holds Treasury account at CB, owes bonds
4. **Central Bank** — holds bonds and advances, owes reserves and Treasury account
5. **Commercial Banks** — hold reserves, loans, and bonds; owe deposits
6. **Foreign (Rest of World)** — zero balances for MVP (closed economy); post-MVP holds foreign assets/liabilities for trade and capital flows

Every financial instrument is an asset for one sector and a liability for another. The **balance sheet matrix** captures this — each row (instrument) must sum to zero across all columns (sectors):

| Instrument | Households | Firms | Treasury | Central Bank | Banks | Foreign | Sum |
|---|---|---|---|---|---|---|---|
| Treasury account | | | +T_a | −T_a | | | 0 |
| Reserves (H) | | | | −H | +H | | 0 |
| Deposits (D) | +D_h | +D_f | | | −(D_h+D_f) | | 0 |
| Loans (L) | −L_h | −L_f | | | +(L_h+L_f) | | 0 |
| Bonds (B) | | | −B_s | +B_cb | +B_b | | 0 |
| Capital goods (K) | | +K | | | | | +K |
| Housing stock (H_r) | +H_r | | | | | | +H_r |
| **Net worth** | **NW_h** | **NW_f** | **NW_g** | **NW_cb** | **NW_b** | **0** | **−(K+H_r)** |

Note: Capital goods (K) and housing stock (H_r) are real assets with no corresponding liability, so the net worth row sums to −(K+H_r) (total financial net worth equals zero; total net worth including real assets equals K+H_r). Housing stock is a durable real asset held by households, produced by the Construction sector (see [Housing Market](#housing-market)). Household loans (L_h) include both consumer loans and mortgages; the collateral relationship between mortgages and housing is tracked in loan-level data, not as a separate balance sheet instrument. Foreign sector = 0 for MVP (closed economy); columns and instruments for foreign assets/liabilities are added post-MVP.

### Sectoral Balances Identity

For analytical and reporting purposes, the five institutional sectors consolidate into three:

- **Government sector** = Treasury + Central Bank (consolidated)
- **Domestic private sector** = Households + Firms + Commercial Banks
- **Foreign sector** = 0 (MVP, closed economy)

**Consolidation:** When Treasury and Central Bank are consolidated, intra-government items cancel:
- Treasury account at CB: Treasury asset (+T_a) and CB liability (−T_a) → cancels
- Bonds held by CB: Treasury liability (−B_cb) and CB asset (+B_cb) → cancels
- Interest paid by Treasury to CB / CB profit remittance: intra-government flows that net to zero

After consolidation, the government sector's liabilities to the private sector are **H + B_b** (reserves + privately-held bonds). The private sector's net financial assets vis-à-vis the government are exactly H + B_b.

The **sectoral balances identity** must hold at the end of every tick:

```
Government balance + Private sector balance + Foreign sector balance = 0
```

For MVP (closed economy, Foreign = 0):

```
Government balance + Private sector balance = 0
```

This is not an assumption — it is enforced by the double-entry bookkeeping. Every dollar of government deficit is mechanically a dollar of private sector surplus. Equivalently: the change in privately-held government liabilities (ΔH + ΔB_b) equals the government deficit (G + r·B_b − T) each tick.

## Two Money Circuits

The model explicitly tracks two layers of money, reflecting how modern monetary systems actually operate. **This two-layer structure is fundamental to the entire economic model.** All money flows in the simulation must respect which layer they belong to:

- **Reserve layer (central bank money):** Used for settlement between institutions — government↔bank transactions, bank↔bank transactions. Households and firms never touch this layer.
- **Deposit layer (bank money):** Used for all private sector activity — wages, purchases, savings. This is the only money households and firms see.

Any transaction that crosses a bank boundary (e.g., a household at Bank A paying a firm at Bank B) triggers movements in **both** layers simultaneously: deposits move in the lower layer, reserves move in the upper layer. Transactions within the same bank only move deposits.

### Central Bank Money (Reserves)

```
+--------------------------------------------------------------+
|                    CENTRAL BANK LEDGER                        |
|                                                               |
|   Treasury Account          Bank Reserve Accounts             |
|   +--------------+         +--------------+                   |
|   | Government   |         | Bank A       |                   |
|   | balance      |         | reserves     |                   |
|   +--------------+         +--------------+                   |
|                             | Bank B       |                   |
|                             | reserves     |                   |
|                             +--------------+                   |
+--------------------------------------------------------------+
```

- The **Treasury** (government) holds an account at the central bank.
- Each **commercial bank** holds a reserve account at the central bank.
- **Only** the central bank, treasury, and commercial banks operate in this layer.
- The private sector (households, firms) never sees or touches reserves.

**Government spending flow:**
1. Treasury account debited (reserves leave treasury)
2. Recipient's bank reserve account credited (reserves arrive at bank)
3. Bank credits recipient's deposit account (see deposit layer below)

**Taxation flow (reverse):**
1. Taxpayer's deposit account debited at their bank
2. Bank's reserve account at central bank debited
3. Treasury account credited (money effectively destroyed)

**Interbank payment flow (e.g., household at Bank A pays firm at Bank B):**
1. Payer's deposit account debited at Bank A
2. Bank A's reserve account at central bank debited
3. Bank B's reserve account at central bank credited
4. Payee's deposit account credited at Bank B

**Intra-bank payment flow (payer and payee at same bank):**
1. Payer's deposit account debited
2. Payee's deposit account credited
3. No reserves move — internal ledger adjustment only

### Deposit Money (Bank Deposits)

```
+--------------------------------------------------------------+
|                  COMMERCIAL BANK LEDGERS                      |
|                                                               |
|   Household Accounts         Firm Accounts                    |
|   +--------------+         +--------------+                   |
|   | Low income   |         | Agriculture  |                   |
|   +--------------+         +--------------+                   |
|   | Middle income|         | Manufacturing|                   |
|   +--------------+         +--------------+                   |
|   | High income  |         | Construction |                   |
|   +--------------+         +--------------+                   |
|                             | Services     |                   |
|                             +--------------+                   |
|                                                               |
|   This is the only money households and firms see.            |
+--------------------------------------------------------------+
```

- Households and firms hold deposit accounts at commercial banks.
- All private sector transactions occur in this layer.
- Wages, purchases, savings — all are movements of deposit money between accounts.

### Bank Credit Creation (Endogenous Money)

Banks create new deposit money when they issue loans. This is NOT lending out existing deposits — it is the creation of new money:

```
Bank makes a $50,000 mortgage:
  Bank's balance sheet:
    Assets:  +$50,000 (loan receivable)
    Liabilities: +$50,000 (new deposit in borrower's account)

  Borrower's balance sheet:
    Assets:  +$50,000 (new deposit)
    Liabilities: +$50,000 (mortgage debt)
```

When the loan is repaid, the process reverses — deposit money is destroyed.

**Note:** Bank lending does NOT require prior reserves. Banks lend first and seek reserves after (or the central bank provides them). This is a key MMT/post-Keynesian insight about endogenous money.

## Sectors and Agents

### Government

The currency-issuing sovereign. The player controls this sector.

**Balance sheet tracks:**
- Treasury account at central bank (reserves)
- Outstanding bonds (liabilities)
- Accumulated deficit/surplus

> **Critical design rule:** The Treasury account balance never constrains government spending. The government, as currency issuer, can always spend by crediting bank reserve accounts. The Treasury balance is tracked for accounting purposes only — it is not a budget constraint. The real constraint on government spending is inflation (available real resources), not money.
>
> **Political constraints** (debt ceilings, balanced budget rules) are self-imposed policy choices, not operational limits. These are modeled in the policy constraint layer, not in the engine. See GAME-DESIGN.md Section 5.

> **Note:** Government spending and bank lending are both sources of endogenous money creation. Government creates currency (reserves + deposits) by crediting bank reserve accounts; banks create currency (deposits only) by issuing loans. See [Bank Credit Creation](#bank-credit-creation-endogenous-money) for the private-sector channel.

**Actions:**
- Spend money (creates currency) allocated across functional divisions per ADR-0018
- Collect taxes (destroys currency)
- Issue bonds (auction-based, see below)
- Set tax rate
- Set spending level and functional allocation

**Two-layer spending architecture (ADR-0018):**

Government spending uses a two-layer architecture that separates economic mechanics from policy presentation:

**Layer 1 — Engine (economic flow types).** The simulation engine processes only SFC/SNA flow types:
- **Compensation of employees (W_g):** Wages and benefits to public sector workers. Absorbs labor, creates household income directly.
- **Intermediate consumption (G_ic):** Purchases of goods and services from private sector firms. Creates sectoral demand and firm revenue.
- **Social benefits / transfers (TR):** Cash payments to households. Creates household income with no direct resource claim — recipients spend via the AIDS demand system.
- **Gross capital formation (G_cf):** Government investment in fixed assets. Creates sectoral demand and accumulates as public capital stock. (MVP: merged with intermediate consumption.)
- **Interest payments (r·B):** Interest on outstanding bonds. Automatic, flows to bondholders.

**Layer 2 — Presentation (functional divisions).** The player allocates spending across functional divisions (e.g., public services, infrastructure, social transfers, defense). Each division is a data-driven recipe specifying its decomposition into engine flow types and sector targeting. For example, education is ~80% compensation / ~10% procurement / ~10% capital, while social protection is ~93% transfers. See ADR-0018 for full rationale and example definitions.

**Government as employer and procurer:**

The government participates in the real economy through two channels, both driven by the functional division decomposition:

1. **Direct employment** — The compensation share of each functional division determines how many workers the government hires from the labor pool, competing with private firms. Government wage rates are set by a data-driven pay scale that adjusts more slowly than private sector wages (civil service stickiness). Public sector employment is tracked separately from private employment.

2. **Procurement** — The intermediate consumption and capital formation shares of each functional division generate demand in private sectors, with sector targeting specified per division.

This distinction is central to the MMT argument. Transfers increase household income and create demand distributed across sectors via the AIDS consumption model. Compensation and procurement directly absorb real resources — labor and sector output. The same dollar amount produces very different economic effects depending on the functional division's composition — education (employment-heavy) has a very different multiplier profile from social protection (transfer-heavy) or defense (procurement-heavy).

The government's wage offers create a de facto wage floor. If the government pays 3,000/month for public service workers, private firms offering significantly less will lose workers. This demonstrates how fiscal policy sets an implicit minimum wage — a key MMT/post-Keynesian insight and the precursor to the Job Guarantee's role as a wage anchor.

### Central Bank

Operates the reserve system and sets monetary policy.

**Balance sheet tracks:**
- Government bonds held (assets) — acquired via open market operations or as buyer of last resort at primary auctions
- Treasury account (liability) — the government's account at the central bank
- Bank reserve accounts (liabilities) — reserves held by commercial banks
- Net worth (typically zero or near-zero — CB remits profits to Treasury)

**Actions:**
- Maintain reserve accounts for commercial banks
- Set the policy interest rate (base rate)
- Provide reserves to commercial banks on demand at the policy rate (lender of last resort / standing facility). The standing facility rate may differ from the policy rate (corridor/floor system — post-MVP refinement)
- Buy/sell government bonds (open market operations)
- Act as buyer of last resort at primary bond auctions

Reserve provision ensures that bank lending is never constrained by reserve availability. Banks lend based on creditworthiness of borrowers and profitability, then obtain any needed reserves from the central bank (see [Bank Credit Creation](#bank-credit-creation-endogenous-money) above).

### Commercial Banks

Intermediate between the reserve and deposit circuits. Create credit.

**Balance sheet tracks:**
- Reserve account at central bank (asset)
- Loans outstanding (assets)
- Customer deposits (liabilities)
- Government bonds held (assets)
- Equity / retained profits

**Behavior:**
- Accept deposits from households and firms
- Create loans based on creditworthiness assessment (see Lending section)
- Buy government bonds at auction
- Earn interest on loans and bonds
- Pay interest on deposits (if modeled)
- Lending rate = central bank policy rate + bank spread + risk premium

Banking is modeled as a credit-creation and payments sector, not as a production sector. Banks do not produce output that households consume. Their function is to create purchasing power (credit money) that enables real economic activity. This is consistent with the MMT/post-Keynesian SFC tradition (Godley & Lavoie) where the banking sector has a balance sheet and behavioral rules, not a production function. Bank revenue (interest spread) appears on balance sheets through interest flows, not through the AIDS demand system.

### Households

#### Income Architecture (ADR-0019)

The engine tracks every household's income by source, following standard SFC accounting:

- **Wage income** — from employment (firm or government)
- **Dividend income** — from equity holdings in firms/bank
- **Interest income** — on bank deposits
- **Transfer income** — government transfers (unemployment benefits, etc.)

```
Gross income = wages + dividends + interest + transfers
Disposable income (YD) = gross income - taxes - debt service
```

This granular tracking enables differential taxation of wage vs capital income (a real-world policy lever) and provides the data for macro indicators (wage share, profit share).

#### Income Classes

Households are assigned to income classes that determine their behavioral parameters (AIDS, consumption propensities, reservation wages). Class population shares are loaded from scenario data files.

| Class | Population share (US MVP) | Characteristics |
|---|---|---|
| **Low income** | 30% (Pew 2024) | High propensity to consume. Budget share heavily weighted toward agriculture and basic manufacturing. Minimal savings. May take on debt for necessities. |
| **Middle income** | 51% (Pew 2024) | Moderate consumption and saving. More balanced budget shares across sectors. Take on mortgage debt for housing. Some discretionary spending on services. |
| **High income** | 19% (Pew 2024) | Low propensity to consume (relative to income). Budget share shifts toward services and discretionary manufacturing. Significant savings. Housing wealth accumulation via owned housing stock. |

Class membership is **fixed for MVP**. This is consistent with every SFC and ABM model surveyed (Godley & Lavoie, Dos Santos-Zezza, EURACE, K+S, JAMEL) — none implement income-threshold reclassification. Post-MVP, dynamic reclassification based on rolling average income with hysteresis is planned (see ADR-0019).

#### Functional Composition (Analytical)

The engine computes a **capital income share** per household each tick: `(dividends + interest) / gross income`. This is not a behavioral classification — it does not affect household parameters. It provides:

- **Aggregate wage share**: Central Kaleckian/MMT indicator. Rising profit share shifts income toward higher-saving households, reducing aggregate demand.
- **Tax policy foundation**: Enables differential taxation of wage vs capital income.

#### Balance Sheet

- Deposit account at bank (asset)
- Housing stock (real asset — valued at current market price, see [Housing Market](#housing-market))
- Consumer loans (liabilities)
- Mortgages (liabilities — secured by housing stock)
- Net wealth = deposits + housing value - consumer loans - mortgage debt

#### Behavior

- Supply labor to firms and government (see Labor Market)
- Consume according to the two-stage budgeting model: the consumption function determines total expenditure from income and wealth, then AIDS allocates it across sectors (see [Consumption Function](#consumption-function) and [Consumption Model](#consumption-model-almost-ideal-demand-system-aids))
- Save surplus income (savings = disposable income - consumption expenditure)
- Purchase housing from the Construction sector, financed by savings (down payment) and mortgage borrowing (see [Housing Market](#housing-market))
- Borrow from banks for asset/durable goods purchases (see [Household Borrowing](#household-borrowing))
- Pay taxes on income

### Firms (4 Sectors)

All firms are profit-driven: they estimate demand, consider costs, and make production and hiring decisions to maximize profit.

| Sector | Inputs | Outputs | Characteristics |
|---|---|---|---|
| **Agriculture & Primary** | Labor, land | Food, raw materials | Land-constrained, seasonal, low labor elasticity. Small GDP share but essential. |
| **Manufacturing** | Labor, raw materials, capital goods | Manufactured goods, capital goods | Capital-intensive. Produces both consumer goods and investment goods for all sectors. |
| **Construction** | Labor, manufactured materials, capital goods | Built structures, infrastructure, housing units | Highly cyclical, labor-intensive. Key channel for government infrastructure investment. Produces housing as a distinct durable asset for households (see [Housing Market](#housing-market)), alongside commercial/infrastructure construction output. |
| **Services** | Labor, capital goods | Services (retail, transport, healthcare, education, hospitality, professional) | Labor-intensive, largest employment share. Heterogeneous but shares key input characteristics. |

**Sector expansion:** Sectors expand via a sub-sector hierarchy. Each top-level sector may contain sub-sectors with independent production functions and AIDS demand parameters. For example, Services may split into Healthcare, Education, Hospitality, and Professional Services. Sub-sectors are identified by a `parentId` field in `sectors.json`. The AIDS demand system extends naturally — a 4-sector parameter matrix becomes an 8-sector or 12-sector matrix.

**Balance sheet tracks:**
- Deposit account at bank (asset)
- Capital goods / equipment (assets)
- Loans (liabilities)
- Equity / retained profits
- Inventory (unsold goods)

**Behavior:**
- Estimate demand and set production targets
- Post wages and hire workers (see Labor Market)
- Purchase inputs from other sectors
- Produce goods/services
- Set prices (see Pricing)
- Pay wages and taxes
- Invest in capital via bank loans or internal funds
- Hold inventory of unsold goods

### Demand Estimation

Firms use adaptive expectations to forecast demand for production planning (Caiani et al. 2016). Each tick, a firm estimates expected demand based on its recent sales history:

```
ExpectedDemand_t = PreviousSales_t-1 × (1 + trend_t)
trend_t = trend_t-1 + adaptationSpeed × (salesGrowth_t-1 - trend_t-1)
salesGrowth_t-1 = (PreviousSales_t-1 - PreviousSales_t-2) / PreviousSales_t-2
```

Where:
- `PreviousSales` is the firm's actual sales (quantity sold) in the previous tick
- `trend` is a smoothed estimate of sales growth, updated each tick via partial adjustment
- `adaptationSpeed` is a data-driven parameter (e.g., 0.25) controlling how quickly firms revise their expectations

**Production target:** The firm sets its production target to replenish inventory to the target level while meeting expected demand:

```
ProductionTarget = ExpectedDemand + max(0, TargetInventory - CurrentInventory)
TargetInventory = ExpectedDemand × TargetInventoryRatio
```

This links demand estimation to the inventory-based markup system (see Pricing below): if firms consistently overestimate demand, inventories pile up, markup falls, and prices drop. If they underestimate, inventories deplete, markup rises, and prices increase. The feedback is self-correcting.

**Edge cases:** When `PreviousSales = 0` (new firm or total demand collapse), set `ExpectedDemand = InitialDemandEstimate` (a data-driven per-sector parameter representing startup expectations). When `PreviousSales_t-2 = 0` (only one tick of history), set `salesGrowth = 0` (no trend signal).

## Production Function: Leontief Input-Output

Each sector uses **fixed technical coefficients** (Leontief / fixed proportions). To produce one unit of output, a firm needs fixed quantities of each input — inputs cannot substitute for one another.

```
Output_s = min(Input_1 / a_s1, Input_2 / a_s2, ..., Labor / a_sL)

Where:
  a_si = technical coefficient for input i in sector s
  a_sL = labor coefficient for sector s
```

For example, if Manufacturing has coefficients `labor: 0.4, raw_materials: 0.35, capital: 0.25`, then producing 100 units requires exactly 40 units of labor, 35 units of raw materials, and 25 units of capital goods. A shortage of any input constrains output proportionally.

### Inter-Sector Input-Output Matrix

Sectors consume each other's output as intermediate inputs. These linkages are defined by an input-output matrix specifying how much of each sector's output is required per unit of production in every other sector:

| Producing ↓ / Input → | Agriculture | Manufacturing | Construction | Services | Labor |
|---|---|---|---|---|---|
| **Agriculture** | — | low | — | low | high |
| **Manufacturing** | moderate | low | — | low | moderate |
| **Construction** | — | high | — | low | high |
| **Services** | low | low | — | — | high |

The exact coefficients are data-driven, loaded from `sectors.json`. The matrix above shows qualitative magnitudes.

**Key properties:**
- Supply shocks propagate through the matrix: a disruption in Agriculture constrains Manufacturing (raw materials), which constrains Construction (manufactured materials)
- The matrix makes inter-sector dependencies explicit and produces emergent supply chain dynamics
- Government procurement enters through this matrix — infrastructure spending creates demand in Construction, which pulls from Manufacturing
- Coefficients are fixed in the short run; long-run technological change (coefficient drift) allows them to evolve over time

This follows the Godley/Lavoie (2012) SFC modeling tradition and Sraffa's (1960) *Production of Commodities by Means of Commodities*. Input coefficients in `sectors.json` are Leontief technical coefficients, not Cobb-Douglas exponents.

> **Design note — why Leontief, not Cobb-Douglas or CES:**
> Cobb-Douglas assumes smooth input substitution (σ=1), which implies wages equal marginal product — a neoclassical result incompatible with MMT/post-Keynesian wage theory. Empirical estimates of the elasticity of substitution consistently fall in the 0.4–0.7 range (Chirinko 2008, Oberfield & Raval 2021), much closer to Leontief (σ=0) than Cobb-Douglas. CES with low σ would be more empirically precise but adds calibration complexity without meaningful gameplay difference. Pure Leontief is the simplest model consistent with both the empirical evidence and the post-Keynesian theoretical framework.

## Pricing Model

### Cost-Plus Markup with Demand Adjustment

Prices are based on **unit costs**, not raw input prices. This is critical: wage increases alone do not cause inflation if productivity keeps pace.

```
Price = (Unit labor cost + Unit material cost) x (1 + markup)

Where:
  Unit labor cost = Total wages paid / Total output produced
  Unit material cost = Total material costs / Total output produced
  Markup = Base markup x Demand adjustment factor
```

**Demand adjustment factor** (Caiani et al. 2016):
```
DemandAdjustmentFactor = TargetInventoryRatio / ActualInventoryRatio
```
- When inventories pile up (low demand): factor < 1, markup falls (competitive pressure)
- When inventories at target: factor = 1, markup at base level
- When inventories deplete (high demand): factor > 1, markup rises (sellers have pricing power)

**Edge cases:** `ActualInventoryRatio = CurrentInventory / RecentSales`. When `RecentSales = 0` (no sales), treat `ActualInventoryRatio` as equal to `TargetInventoryRatio` (factor = 1, no pressure). When `CurrentInventory = 0`, cap `DemandAdjustmentFactor` at a configurable maximum (e.g., 2.0) to prevent unbounded markup spikes.

Inventory-based adjustment is preferred over capacity-utilization-based because firms directly observe their own inventory levels. `TargetInventoryRatio` is a per-sector parameter from sectors.json.

**Markup adjustment asymmetry:**
Markups adjust at different speeds in each direction:
- Upward: fast — firms exploit pricing power quickly (e.g., speed factor 0.5)
- Downward: slow — firms resist margin compression (e.g., speed factor 0.1)

This mirrors the existing downward wage rigidity. Per-sector parameters (`markupUpwardSpeed`, `markupDownwardSpeed`) loaded from sectors.json.

### Supply-Side Markup Pressure (Seller's Inflation)

When input availability drops (e.g., material shortage from another sector), firms with constrained supply can raise markups independently of demand:

```
Supply pressure factor = Normal input availability / Actual input availability
```

"Normal input availability" is the previous period's value (t-1), following the standard SFC convention (Godley & Lavoie 2012). This requires no extra parameters and naturally adapts if supply conditions permanently shift.

When supply pressure > 1, it multiplies the markup alongside the demand factor:

```
Markup = Base markup × Demand adjustment × Supply pressure factor
```

This models seller's inflation (Weber/Wasner 2023): firms use supply scarcity as cover to widen margins even without excess demand. The supply pressure factor uses the same asymmetric adjustment speeds — it ratchets up fast but decays slowly as supply normalizes.

### Three Buffers Against Inflation

Higher wages do NOT automatically cause inflation. Three mechanisms absorb wage increases before prices must rise:

1. **Productivity gains** — if output per worker rises with wages, unit labor costs are unchanged
2. **Demand slack** — if there is unsold inventory or idle capacity, higher wages increase sales volume without requiring more production (e.g., less food waste, fuller capacity utilization)
3. **Profit margin compression** — firms may accept lower markups under competitive pressure rather than raise prices

**Inflation occurs when:** all three buffers are exhausted — productivity can't keep up, there is no slack, and margins are already compressed. Then unit costs rise and firms pass them through as higher prices.

### Unit Labor Costs (Lohnstuckkosten)

The model tracks unit labor costs per sector, not raw wages:

```
Unit labor cost = Wage rate / Labor productivity
Labor productivity = Output / Workers employed
```

- If wages rise 3% and productivity rises 3% -> ULC unchanged -> no price pressure
- If wages rise 5% and productivity rises 2% -> ULC rises 3% -> cost-push price pressure
- If wages rise 2% and productivity rises 4% -> ULC falls 2% -> prices can fall or margins improve

## Consumption Function

Household consumption uses **two-stage budgeting** (Deaton & Muellbauer 1980, Godley & Lavoie 2012):

1. **Stage 1 — Consumption function:** Determines total consumption expenditure (M_c) from income and wealth
2. **Stage 2 — AIDS allocation:** Allocates M_c across sectors based on prices and real income

### Total Consumption Expenditure

For household class _c_:

```
M_c = alpha1_c * YD_c + alpha_f_c * V_f_c(-1) + alpha_h_c * V_h_c(-1)

Where:
  YD_c     = disposable income (post-tax income - debt service payments)
  V_f_c    = financial wealth (bank deposits)
  V_h_c    = housing wealth (housing stock value at current market price - mortgage debt outstanding)
  alpha1_c = propensity to consume out of income (0 < alpha1 < 1)
  alpha_f_c = marginal propensity to consume out of financial wealth
  alpha_h_c = marginal propensity to consume out of housing wealth
  (-1)     = lagged one tick (previous period's wealth)
```

This is the standard Godley/Lavoie consumption function extended with disaggregated wealth (Zezza 2008). The saving rate emerges endogenously: `savings_c = YD_c - M_c + capital_gains_c`.

### Wealth Decomposition

Total household net wealth decomposes into two components with distinct propensities:

- **Financial wealth (V_f):** Bank deposits. Liquid, immediately available. Lower MPC because households treat liquid savings as a buffer.
- **Housing wealth (V_h):** Market value of housing stock minus mortgage debt outstanding. Illiquid but large. Higher MPC because housing price changes feel permanent and affect borrowing capacity (collateral channel).

### Parameter Calibration

Per-class propensities are loaded from data files. Empirical ranges (annual, from Case/Quigley/Shiller 2005, Zezza 2008, ECB WP 1283):

| Parameter | Range | Notes |
|---|---|---|
| alpha1 (income) | 0.70 – 0.95 | Higher for low income (consume more of income), lower for high income |
| alpha_f (financial wealth) | 0.02 – 0.05 | Lower than housing MPC due to buffer motive |
| alpha_h (housing wealth) | 0.03 – 0.09 | Higher than financial MPC — housing wealth perceived as permanent |

**Key property:** alpha_h ≥ alpha_f is the empirical consensus. This means housing price changes have a larger impact on consumption than equivalent changes in financial wealth, despite housing being less liquid. This reflects housing's wider distribution across income groups and the collateral channel (rising house prices → more borrowing capacity → more spending).

### Interaction with AIDS

The consumption function determines _how much_ to spend. AIDS determines _what_ to spend it on. Housing wealth affects total expenditure but does not enter the AIDS budget share equation — the allocation across sectors is driven by relative prices and real income, not by the composition of wealth. This separation is the standard two-stage budgeting approach in consumer demand theory.

Housing purchases are _not_ part of AIDS consumption demand. Housing is an asset purchase funded by savings (down payment) and mortgage borrowing, handled separately in the [Housing Market](#housing-market) section.

## Consumption Model: Almost Ideal Demand System (AIDS)

Household consumption is modeled using the Almost Ideal Demand System (Deaton & Muellbauer, 1980). AIDS determines how each household class allocates its total consumption expenditure (M_c, from the [Consumption Function](#consumption-function) above) across production sectors based on current prices and real income.

### The AIDS Formula

For household class _c_ and sector _i_:

```
w_ci = alpha_ci + SUM_j(gamma_cij * ln(p_j)) + beta_ci * ln(M_c / P)
```

Where:
- `w_ci` is the budget share household class _c_ spends on sector _i_ output
- `alpha_ci` is the intercept — the baseline budget share at reference prices and income
- `gamma_cij` captures how the budget share for sector _i_ responds to a change in sector _j_'s price (own-price and cross-price effects)
- `beta_ci` captures how the budget share changes as real income changes (income elasticity)
- `M_c` is total consumption expenditure for household class _c_ (determined by the [Consumption Function](#consumption-function) — a function of disposable income and lagged wealth)
- `P` is a price index (Stone price index: `ln(P) = SUM_i(w_ci * ln(p_i))`)
- `p_j` is the price of sector _j_'s output

### Parameter Constraints

The AIDS model satisfies three theoretical constraints that ensure economic consistency:

1. **Adding-up:** Budget shares sum to 1 for each class. `SUM_i(alpha_ci) = 1`, `SUM_i(gamma_cij) = 0`, `SUM_i(beta_ci) = 0`.
2. **Homogeneity:** Demand is homogeneous of degree zero in prices and income (doubling all prices and income leaves budget shares unchanged). `SUM_j(gamma_cij) = 0` for each _i_.
3. **Symmetry:** Cross-price effects are symmetric. `gamma_cij = gamma_cji`.

These constraints reduce the effective number of free parameters and ensure the model produces economically meaningful results.

### Per-Class Parameterization

Each household class has its own set of alpha, beta, and gamma parameters. This reflects the empirical finding that preferences differ structurally across income groups — rich households don't just spend more, they spend _differently_ even at the same price levels.

**Key behavioral properties that emerge from the parameters:**

- **Engel's law:** Low-income households spend a larger budget share on agriculture (food). As income rises, the agriculture share declines (`beta_agriculture < 0`). This is one of the most robust findings in economics.
- **Income elasticity:** Services budget share increases with income (`beta_services > 0`). Manufacturing and construction shares are relatively stable.
- **Price elasticity:** When a sector's price rises, its budget share adjusts based on gamma parameters. Agriculture demand is price-inelastic (people buy food regardless of price). Services demand is more elastic.
- **Cross-price substitution:** When manufacturing prices rise, some demand shifts to other sectors based on gamma cross-terms.

### Empirical Foundation

AIDS parameters are among the most empirically estimated in all of economics. Parameters are sourced from published estimates based on US household expenditure data (Consumer Expenditure Survey, Bureau of Labor Statistics). Key references include AIDS estimates by income group using CEX data and USDA Economic Research Service food demand elasticity estimates.

Different country parameter sets (Germany, Brazil, Nigeria, etc.) can be loaded as scenario data files. Different countries have structurally different consumption patterns — a US economy is services-dominated (~77% of GDP) while a German economy has a significantly larger manufacturing share (~20% of GDP). These differences are captured entirely by different alpha, beta, gamma parameters.

### Edge Cases

- **Near-zero income:** The `ln(M/P)` term diverges as income approaches zero. A small epsilon floor is applied to real income before the logarithm to prevent undefined behavior. This is a numerical safeguard, not an economic subsistence floor — the welfare state (government transfers) is the policy mechanism that prevents destitution, and its absence should be visible as a crisis.
- **Extreme prices:** Budget shares are clamped to `[0, 1]` after computation and renormalized to sum to 1. This prevents the model from producing economically meaningless negative demand at extreme price levels.

### From Budget Shares to Nominal Demand

Each tick, the consumption engine:

1. Computes total consumption expenditure `M_c` for each household class using the [Consumption Function](#consumption-function) (income + lagged wealth)
2. Computes budget shares `w_ci` for each household class and sector using current prices and real income via AIDS
3. Converts to nominal demand: `demand_ci = w_ci * M_c`
4. Passes the demand vector to the goods market for fulfillment against sector inventory/capacity
5. If demand exceeds supply, rationing occurs — unmet demand feeds into the next tick's price adjustments via the pricing engine

Note: Housing purchases are handled separately in the [Housing Market](#housing-market), not through this consumption demand pipeline.

## Labor Market: Wage Posting

Firms and the government post job openings with an offered wage. Households decide whether to accept based on their circumstances.

### Employer Side (Posting)

**Firms:**
- Firms determine labor demand based on their production targets
- Each firm posts a wage for open positions
- Wage influenced by: sector conditions, competition for workers, profitability
- Wages tend to rise when: labor is scarce, the sector is profitable, firms compete for workers
- Wages tend to fall when: labor is abundant, profits are squeezed (but sticky downward — wage cuts are slow)

**Government:**
- Government determines labor demand based on spending allocation (infrastructure, public services)
- Government posts wages based on a data-driven pay scale
- Government wages adjust more slowly than private wages (civil service stickiness)
- Government wage rates are not directly profit-driven — they follow policy-set pay scales
- Government job postings enter the same labor market pool as firm postings

### Household Side (Accepting)
- Each income class has a **reservation wage multiplier** (loaded from `households.json`) applied to the economy-wide average wage
- Low income: lower reservation wage (more willing to accept available work)
- High income: higher reservation wage (can afford to be selective)
- Households accept jobs that meet or exceed their reservation wage
- Preference for higher-paying jobs; workers choose between public and private employment based on offered wage
- Workers move to better-paying sectors/employers over time

### Unemployment
- Occurs when: employers (firms + government) don't post enough jobs, or posted wages are below reservation wages
- Unemployment is involuntary when workers want to work but no suitable jobs exist
- Unemployed households have zero wage income; rely on savings, transfers, or debt

### Wage Dynamics
- Tight labor market (low unemployment) -> wages rise as employers compete for workers
- Loose labor market (high unemployment) -> wages stagnate or fall slowly (downward rigidity)
- Sector-specific: agriculture wages may differ from services wages
- Government wages anchor the lower end of the distribution — private firms cannot push wages far below public pay without losing workers

### Employment Tracking
- Private sector employment tracked per sector (agriculture, manufacturing, construction, services)
- Public sector employment tracked per spending function (infrastructure, public services)
- Total employment = private + public
- Employment indicators distinguish public and private shares

## Bank Lending: Creditworthiness-Based

Banks create money through lending. Lending decisions are based on borrower creditworthiness with interest rates tied to the central bank policy rate.

### Lending Rate
```
Lending rate = Central bank policy rate + Bank spread + Risk premium

Where:
  Central bank policy rate: set by central bank
  Bank spread: bank's base profit margin on lending
  Risk premium: varies by borrower creditworthiness
```

### Creditworthiness Assessment

Banks use different metrics for firms and households, reflecting real-world commercial vs consumer lending practice.

**Firms — Debt Service Coverage Ratio (DSCR):**

```
Approved if: Net Operating Income / Total Debt Service ≥ DSCR_threshold

Where:
  Net Operating Income = revenue - operating costs (before debt service)
  Total Debt Service = principal + interest payments on all existing debt + proposed new debt
  DSCR_threshold ≈ 1.25 (standard commercial lending minimum)
  DSCR_threshold is loaded from bank parameters (data file) and is moddable
```

A DSCR of 1.25 means the firm's operating income is 25% above what's needed to cover all debt payments — a standard safety margin in commercial lending. Higher DSCR → lower risk premium on the lending rate. Below 1.25 → loan denied.

**Households — Debt-to-Income Ratio (DTI):**

See [Household Borrowing](#household-borrowing) for the full specification. DTI threshold ≈ 0.40 (40%).

### Loan Types
- **Business loans** — for firms investing in capital goods. Evaluated via DSCR. Term matched to capital useful life.
- **Household consumer loans** — amortizing installment loans for asset/durable goods purchases. Evaluated via DTI. Fixed 60-tick (5-year) term. _(post-MVP)_
- **Household mortgages** — long-term amortizing loans for housing purchases. Evaluated via DTI + LTV. Fixed 360-tick (30-year) term. Secured by housing stock with collateral-based default. _(post-MVP — see [Housing Market](#housing-market))_

### Firm Loan Structure

Firm loans use the same **fixed amortizing installments** as household loans, with the term derived from capital useful life:

```
Payment = Principal × [r(1+r)^n] / [(1+r)^n - 1]

Where:
  r = lending rate per period (CB policy rate + bank spread + risk premium)
  n = min(floor(1 / depreciationRate), maxFirmLoanTerm)

  depreciationRate = sector-specific per-tick geometric depreciation rate (ADR-0012)
  maxFirmLoanTerm = cap from bank parameters (default: 120 ticks / 10 years)

Each payment splits into:
  Interest portion = outstanding balance × r  (bank revenue, not destroyed)
  Principal portion = payment - interest       (money destruction)
```

This matches real-world commercial lending practice where equipment loan terms are tied to the useful life of the financed asset. The cap prevents unreasonably long loans in sectors with very low depreciation rates.

### Credit Creation Mechanics
When a loan is approved:
1. Bank creates new deposit in borrower's account (money created)
2. Bank records loan as an asset (receivable)
3. Borrower records loan as a liability (debt)
4. **No reserves are needed first** — banks lend first, seek reserves after

When a loan is repaid:
1. Borrower's deposit debited (money destroyed)
2. Bank's loan asset reduced
3. Interest payments are the bank's revenue (not destroyed — they stay as bank deposits)

### Defaults
- If a borrower cannot service their debt, they default
- Bank writes off the loan (loss)
- If bank losses exceed equity, bank becomes insolvent (systemic risk)

### Household Borrowing

Household borrowing is modeled as **amortizing installment loans for asset and durable goods purchases** (mortgage-style). This reflects empirical US household debt composition where ~79% of outstanding debt is mortgage + auto loans (NY Fed Household Debt and Credit Report, Q4 2024).

#### Borrowing Decision
Households seek loans when they wish to purchase durable goods (from the Construction and Manufacturing sectors) whose cost exceeds their current savings. The borrowing motive is asset acquisition, not consumption smoothing — current consumption is funded from income via the AIDS demand system.

#### Bank Evaluation
The bank evaluates household loan applications using a **debt-to-income (DTI) ratio** threshold:

```
Approved if: (existing debt service + new payment) / gross income < DTI_threshold

Where:
  DTI_threshold ≈ 0.40 (40%, consistent with US conventional lending standards)
  DTI_threshold is loaded from bank parameters (data file) and is moddable
```

This is consistent with the endogenous money framework: banks are not reserve-constrained but **are** creditworthiness-constrained. The bank will lend to any household that passes the DTI check, creating new deposits in the process (see [Credit Creation Mechanics](#credit-creation-mechanics)).

#### Loan Structure
Household loans use **fixed amortizing installments**:

```
Each period, the household pays a fixed amount:
  Payment = Principal × [r(1+r)^n] / [(1+r)^n - 1]

Where:
  r = lending rate per period (CB policy rate + bank spread + risk premium)
  n = loan term in ticks (loaded from bank parameters, moddable)

Each payment splits into:
  Interest portion = outstanding balance × r  (bank revenue, not destroyed)
  Principal portion = payment - interest       (money destruction)
```

#### Default
A household defaults when it has **zero income AND zero savings (deposits)** for N consecutive ticks (N loaded from bank parameters, moddable). On default:

1. The bank removes the loan from its asset book (write-off)
2. The loss is absorbed first by the bank's **loan loss reserve** (allowance), then by bank equity if the reserve is exhausted
3. The deposits originally created by the loan **remain in circulation** — default is an equity event for the bank, not a money supply event for the economy
4. The bank's capital ratio worsens, potentially constraining future lending

This correctly reflects the MMT/endogenous money view: when a loan is created, new deposits enter the economy via the borrower's spending. If the borrower later defaults, those deposits are already held by other agents (sellers of the goods purchased). The money doesn't vanish — only the bank's asset (the loan receivable) is destroyed, shrinking the bank's balance sheet and equity.

#### Uniform Across Household Classes
Borrowing rules are uniform across all household classes. The DTI threshold, loan terms, and default triggers do not vary by class. Differences in borrowing outcomes emerge naturally from differences in income levels and existing debt loads.

_Post-MVP: Household borrowing (consumer loans) is fully specified here but deferred to post-MVP implementation. The MVP banking system handles firm lending only. See [MVP-SCOPE.md](../requirements/MVP-SCOPE.md)._

### Housing Market

The housing market models residential real estate as a distinct durable asset on the household balance sheet, financed by mortgage lending. This follows the SFC housing literature (Zezza 2007/2008, Nikolaidi 2014, Herbillon-Leprince 2016) and produces some of the most policy-relevant economic dynamics in the simulation — housing bubbles, wealth effects, balance sheet recessions, and financial contagion through bank losses.

Mortgage debt is ~70% of US household debt (NY Fed Household Debt and Credit Report, Q4 2024). Housing credit cycles drive the most consequential macroeconomic dynamics. A player who deregulates lending standards sees a housing boom followed by a financial crisis. A player who tightens LTV ratios sees stable but slower growth. This is a rich policy lever.

#### Housing Stock

Housing is a **durable real asset** held by households, distinct from consumption goods. It appears as a row in the balance sheet matrix (H_r) and is produced by the Construction sector as a distinct output type alongside commercial construction and infrastructure.

Housing stock accumulates via the **perpetual inventory method** (standard SFC convention):

```
H_r(t) = H_r(t-1) + I_h(t) - delta_h * H_r(t-1)

Where:
  H_r    = housing stock (aggregate units)
  I_h    = residential investment (new housing output from Construction sector)
  delta_h = housing depreciation rate per tick (loaded from data files, e.g., 0.002 per month ≈ 2.4% annually)
```

The Construction sector splits its output between **commercial/infrastructure construction** (demanded by firms and government through normal sector demand channels) and **residential construction** (demanded by households in the housing market). The split is driven by relative demand, not a fixed ratio — if households want more housing, Construction shifts resources toward residential output, subject to its production capacity.

#### House Prices

House prices are **endogenous**, determined by supply and demand in the housing market. The pricing mechanism follows inventory-based adjustment, consistent with the markup pricing used elsewhere in the model:

```
p_h(t) = p_h(t-1) * (1 + adjustment)

adjustment = priceAdjustSpeed * (targetVacancyRate - actualVacancyRate) / targetVacancyRate

Where:
  actualVacancyRate = unsoldHousingStock / totalHousingStock
  targetVacancyRate = loaded from data files (e.g., 0.07 — roughly US historical average)
  priceAdjustSpeed  = loaded from data files (controls price stickiness)
```

- When vacancy is **below target** (tight market): adjustment > 0, prices rise
- When vacancy is **above target** (loose market): adjustment < 0, prices fall
- When vacancy **at target**: prices stable

Price adjustment uses the same **asymmetric speeds** as the markup system — upward adjustment is faster than downward adjustment, reflecting observed house price stickiness (sellers resist lowering asking prices).

**Capital gains** on the existing housing stock are:

```
CG_h(t) = (p_h(t) - p_h(t-1)) * H_r_owned(t-1)
```

Capital gains enter the household wealth accumulation identity and affect consumption via the housing wealth channel in the [Consumption Function](#consumption-function). They are unrealized — households do not receive cash from capital gains, but their balance sheet net worth changes, affecting their propensity to consume.

#### Household Housing Demand

Households seek to purchase housing when they do not currently own housing and can afford a down payment. The housing purchase decision is separate from consumption demand (not allocated via AIDS):

```
Household seeks housing if:
  1. Does not currently own housing (or owns below a class-specific target, loaded from data files)
  2. Has sufficient savings for the down payment: savings >= (1 - LTV_max) * p_h
  3. Would pass the bank's mortgage evaluation (DTI + LTV check)
```

The down payment requirement is implicit in the LTV constraint — if LTV_max = 0.80, the household must fund 20% of the purchase price from savings.

#### Mortgage Specification

Mortgages are **fixed-rate amortizing loans** secured by the housing asset:

```
Mortgage parameters:
  Term:     360 ticks (30 years), loaded from bank parameters, moddable
  Rate:     lending rate at origination (CB policy rate + bank spread + risk premium), fixed for life of loan
  LTV_max:  maximum loan-to-value ratio (e.g., 0.80 = 80%), loaded from bank parameters, moddable
  Amount:   min(LTV_max * p_h, p_h - savings_available)

Monthly payment = Principal * [r(1+r)^n] / [(1+r)^n - 1]

Where:
  r = monthly lending rate (annual rate / 12)
  n = remaining term in ticks
```

#### Mortgage Evaluation

Banks evaluate mortgage applications using **dual criteria** — both must pass:

```
1. DTI check (same as consumer loans):
   Approved if: (existing debt service + new mortgage payment) / gross income < DTI_threshold
   DTI_threshold ≈ 0.40 (40%), loaded from bank parameters, moddable

2. LTV check (mortgage-specific):
   Approved if: loan amount / house price <= LTV_threshold
   LTV_threshold ≈ 0.80 (80%), loaded from bank parameters, moddable
```

The LTV threshold is a key policy lever. Lower LTV → larger down payments → fewer buyers qualify → slower housing appreciation → more stable but less accessible market. Higher LTV → easier entry → faster price appreciation → higher systemic risk.

#### Collateral-Based Default

Mortgage default follows different mechanics from unsecured consumer loan default, reflecting the collateral relationship:

```
Mortgage default triggers:
  Household has zero income AND zero savings (deposits) for N consecutive ticks
  (same trigger as consumer loan default; N loaded from bank parameters, moddable)

On mortgage default:
  1. Bank repossesses the housing asset
  2. Recovery value = current market price of the house (p_h at time of default)
  3. If recovery >= outstanding mortgage balance: bank is made whole, surplus returned to household
  4. If recovery < outstanding mortgage balance: bank absorbs loss = balance - recovery (negative equity)
  5. Loss absorbed first by loan loss reserve, then by bank equity
  6. Repossessed housing enters unsold housing stock (increases vacancy rate, pushes prices down)
  7. Household loses housing asset from balance sheet; housing wealth drops to zero
```

This creates the realistic **doom loop** feedback mechanism:
- Defaults → bank repossession → more unsold housing → lower prices → more negative equity → more defaults
- Banks absorb losses → capital ratio worsens → tighter lending standards → fewer new mortgages → lower demand → lower prices

The doom loop is self-reinforcing and produces the kind of housing crises observed in 2007-09. The player's policy levers (LTV requirements, interest rates, fiscal stimulus) determine whether and how severely this cycle plays out.

#### SFC Accounting for Housing Transactions

A housing purchase creates the following balance sheet changes:

```
Household buys a house for $200,000 with 20% down payment, 80% mortgage:

Household balance sheet:
  Assets:  -$40,000 deposits (down payment)
           +$200,000 housing stock (at purchase price)
  Liabilities: +$160,000 mortgage debt
  Net worth change: +$0 (assets up $160k, liabilities up $160k)

Bank balance sheet:
  Assets:  +$160,000 mortgage receivable
  Liabilities: +$160,000 deposits (credited to seller — Construction sector firm)
  Net worth change: $0

Construction sector firm:
  Assets:  +$200,000 deposits ($40k from buyer's savings + $160k new money from mortgage)
  Inventory: -1 housing unit (delivered to buyer)
```

The mortgage creates $160,000 of new deposit money (endogenous money creation). The down payment transfers $40,000 of existing deposits. The Construction firm receives $200,000 total, which it uses to pay wages, buy materials, and earn profit — feeding back into the circular flow.

Monthly mortgage payments:
- Interest portion → bank revenue (not destroyed)
- Principal portion → money destruction (deposits and loan shrink symmetrically)

This is identical to the loan mechanics described in [Household Borrowing](#household-borrowing), with the addition of the collateral relationship.

#### Future Extensions

The following are explicitly deferred and not part of the current specification:

- **Variable-rate mortgages:** Interest rate adjusts with CB policy rate. Would make monetary policy transmission more direct — rate hikes immediately squeeze existing borrowers, not just new ones. Significant policy lever.
- **Refinancing:** Households replace existing mortgage with new one at current rates. Introduces rate-sensitivity in the existing stock of mortgages, not just new origination.
- **Secondary housing market:** Households sell housing to other households (resale). Currently, households only buy new housing from Construction. A secondary market would decouple house prices from construction costs and enable speculative dynamics.
- **Home equity lines of credit (HELOC):** Borrowing against housing equity for consumption. Would create an additional channel from house prices to aggregate demand, amplifying wealth effects.

## Government Bonds: Auction-Based

The government issues bonds following real-world institutional practice. Bond issuance is a reserve management operation, not a means of "financing" the deficit (see MMT Context below).

### MMT Context
In MMT, bond issuance is a reserve-draining operation, not "financing" for government spending. The government spends first (creating reserves) and then issues bonds (swapping reserves for bonds). Bonds are a monetary policy tool, not a fiscal necessity.

The game models the real-world procedure (auctions) while allowing players to observe that the central bank can always ensure bonds are sold (making the interest rate a policy choice).

### Auction Mechanics
1. Government announces bond issuance (amount and maturity)
2. Commercial banks submit bids (price/yield)
3. Bonds allocated to highest bidders (lowest yield demanded)
4. If demand is insufficient, interest rate rises to attract buyers
5. **Central bank as buyer of last resort:** the central bank can buy any unsold bonds, effectively setting a ceiling on rates

### Bond Properties
- **Face value:** fixed denomination
- **Coupon rate:** determined at auction (interest paid to holder)
- **Maturity:** time until the government repays the face value. MVP uses a single fixed maturity (12 months). The full game introduces multiple maturities (e.g., 3-month, 1-year, 5-year, 10-year) with an emergent yield curve reflecting term premia and fiscal expectations.
- **Secondary market:** bond holders can sell bonds to other agents

### Interest Payments
- Government pays interest on outstanding bonds each period
- This is government spending (creates new currency)
- Interest payments flow to bond holders (banks, and households via secondary market)
- This creates a distributional dynamic: deficit -> bonds -> interest -> flows to bond holders

**Note on the fiscal channel of interest rate policy:** Higher interest rates increase government bond interest payments, which are themselves government spending (currency creation). This means higher rates have a stimulative fiscal effect — more income flows to bond holders — that partially offsets the contractionary effect on borrowing. This is a key reason MMT prefers fiscal policy (spending and taxation) over monetary policy (interest rates) for demand management. See Mosler (1993, 1995), Fullwiler (2006), Wray (2015) Ch. 5.

### No Sovereign Default
The government, as currency issuer, always pays bond interest and principal when due. Bond payments are government spending (currency creation). Sovereign default on domestic-currency-denominated debt is a political choice, never an operational necessity. The game does not model voluntary default.

## Investment

### Public Investment (Government)
Government spending on **infrastructure** builds public capital that benefits all sectors:
- Improves transportation -> reduces costs for all firms
- Improves utilities -> increases productive capacity
- Improves education/health (public services) -> increases labor productivity over time

Public capital depreciates slowly and requires maintenance spending.

### Private Investment (Firms)
Firms invest in capital goods to maintain and expand productive capacity. The investment decision uses a simple accelerator-profit model (Fazzari et al. 1988, Godley & Lavoie 2012 Ch. 11):

**Investment decision:**

```
DesiredInvestment = ReplacementInvestment + ExpansionInvestment

ReplacementInvestment = depreciationRate × CurrentCapital
ExpansionInvestment = accelerator × max(0, ExpectedDemand - CurrentCapacity)
```

Where:
- `depreciationRate` is the sector-specific geometric depreciation rate (ADR-0012)
- `accelerator` is a data-driven parameter (e.g., 0.5) controlling how aggressively firms expand
- `ExpectedDemand` comes from the demand estimation rule (see Demand Estimation above)
- `CurrentCapacity` is the firm's current maximum output given its capital stock

**Funding and rationing:**

```
Shortfall = max(0, DesiredInvestment - RetainedProfits)
MaxLoan = largest loan where DSCR ≥ DSCR_threshold (see Creditworthiness Assessment)
ActualBorrowing = min(Shortfall, MaxLoan)
ActualInvestment = RetainedProfits + ActualBorrowing
```

If `DesiredInvestment > RetainedProfits`, the firm applies for a bank loan for the shortfall. The bank computes the maximum loan the firm can service while maintaining DSCR ≥ 1.25 (see [Creditworthiness Assessment](#creditworthiness-assessment)). If even with borrowing the firm cannot fund desired investment, it invests what it can afford. This creates a realistic credit constraint channel where tight lending standards or high interest rates reduce investment.

**Funding sources:**
1. Internal funds (retained profits — used first)
2. Bank loans (credit creation — new money is created to fund the remaining investment)

Bank lending is the primary enabler of expansion investment. A firm's decision to invest is based on expected demand, capacity utilization, and borrowing costs — not on the pre-existence of savings. When investment is bank-funded, the loan creates new deposits (which become someone's saving), so at the macro level investment creates saving, not the reverse.

**Capital dynamics:**
- Capital goods depreciate over time (require replacement investment)
- New investment expands productive capacity
- Manufacturing sector produces capital goods for all sectors (including itself)

## Time and Lags

### Tick Rate
Each simulation tick represents **one month**. Game speed is adjustable (1x, 2x, 5x).

### Policy Lags
Policy changes do not take effect instantaneously. Short lags with clear visual feedback:

| Policy change | Lag | Rationale |
|---|---|---|
| Tax rate change | 1 tick (1 month) | Tax collection adjusts next period |
| Spending level change | 1-2 ticks | Budget allocation and disbursement takes time |
| Spending reallocation | 2-3 ticks | Redirecting spending to new targets takes longer |
| Infrastructure effects | 6-12 ticks | Building takes time; benefits are gradual |
| Public services effects | 12-24 ticks | Education/health improvements are slow |

### Economic Lags
Natural economic processes also have delays:

| Process | Lag | Rationale |
|---|---|---|
| Wage adjustment | 1-3 ticks | Wages are sticky, contracts renegotiate slowly |
| Price adjustment | 1-2 ticks | Firms adjust prices based on recent cost/demand data |
| Hiring/firing | 1-2 ticks | Recruitment and layoffs take time |
| Investment to capacity | 3-6 ticks | Building new capacity takes months |
| Household spending adjustment | 1 tick | Households adjust spending quickly to income changes |

### Visual Feedback
The UI shows pending changes in a pipeline:
- "Government spending increase: $X arriving in 2 months"
- "Infrastructure project: 40% complete, capacity boost in 4 months"
- Color-coded indicators for "policy enacted" -> "in pipeline" -> "taking effect"

## Simulation Tick Sequence

Each monthly tick processes in this order:

```
1. GOVERNMENT PHASE
   a. Execute spending: pay for infrastructure, public services, transfers
      (new reserves created, flow to bank deposits)
   b. Pay interest on outstanding bonds
      (new reserves created, flow to bank deposits)
   c. Compute government resource demand from spending allocation:
      - Register public sector job postings with labor market
        (infrastructure workers, public service employees)
      - Register procurement demand with sectors
        (construction output, manufacturing materials)
      - Direct transfers: no resource demand, income flows to households
   d. Collect taxes from households and firms
      (deposits destroyed, reserves returned to treasury)
   e. Issue new bonds if needed (auction)
      (drains excess reserves from banking system)

2. PRODUCTION PHASE
   a. Firms assess demand (based on previous period sales + expectations)
   b. Firms set production targets
   c. Firms post job openings with wages
   d. Government posts job openings with public pay scale
   e. Households evaluate ALL job postings (firm + government),
      accept/reject based on reservation wage, prefer higher pay
   f. Production occurs using available labor + materials + capital
   g. Goods enter inventory

3. MARKET PHASE
   a. Firms set prices (cost-plus markup with demand adjustment)
   b. ConsumptionEngine computes AIDS budget shares per household class
      using current prices and disposable income
   c. Nominal demand per sector = budget shares x disposable income
   d. Government procurement demand added to sector demand
   e. Firms sell from inventory to meet combined demand
   f. Unsold goods remain in inventory (potential waste/depreciation)

4. FINANCIAL PHASE
   a. Banks assess loan applications
   b. Approved loans create new deposits (endogenous money)
   c. Borrowers make debt service payments (deposits destroyed)
   d. Defaults processed if borrowers cannot pay
   e. Banks earn interest income, pay interest on deposits

5. ACCOUNTING PHASE
   a. All balance sheets updated
   b. SFC consistency check (all balances must sum to zero)
   c. Economic indicators calculated (employment, inflation, GDP, etc.)
   d. UI updated with new data points
```

## Key Economic Indicators

These are tracked and displayed to the player:

| Indicator | Definition |
|---|---|
| **Employment rate** | Workers employed / Total labor force |
| **Unemployment rate** | Workers seeking work but not employed / Total labor force |
| **Public sector employment share** | Public sector workers / Total employed workers |
| **Inflation rate** | % change in average price level from previous period |
| **GDP (Output)** | Total value of goods and services produced |
| **Government balance** | Spending - Tax revenue (positive = net currency creation, negative = net currency withdrawal) |
| **Private sector balance** | Total private income - Total private spending - Taxes |
| **Private sector net financial assets** | Total government bonds held by private sector + net deposits (mirror image of government bonds outstanding) |
| **Government bonds outstanding / GDP** | Total government bonds outstanding / GDP |
| **Capacity utilization** | Actual output / Maximum possible output (per sector) |
| **Unit labor costs** | Wages / Productivity (per sector) |
| **Private debt level** | Total household + firm loans outstanding |
| **Bank reserves** | Total reserves in the banking system |
| **Savings rate** | Household savings / Household income |
| **Wage growth** | % change in average wages from previous period |
| **Interest rates** | CB policy rate, bank lending rates, bond yields |

For MVP scoping and simplifications, see [MVP-SCOPE.md](../requirements/MVP-SCOPE.md).
