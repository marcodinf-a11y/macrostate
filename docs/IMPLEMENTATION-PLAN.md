# Implementation Plan

This document breaks the MVP into concrete, ordered implementation phases. Each phase has specific deliverables, dependencies, and a definition of done.

## Phase Overview

```
Phase 0: Project Setup
    │
    ▼
Phase 1: Accounting Foundation
    │
    ▼
Phase 2: Economic Agents (Core)
    │
    ▼
Phase 3: Markets & Pricing
    │
    ▼
Phase 4: Banking & Credit
    │
    ▼
Phase 5: Government Bonds
    │
    ▼
Phase 6: Investment & Lags
    │
    ▼
Phase 7: Tick Engine Integration
    │
    ▼
Phase 8: Godot Project & Game Controller
    │
    ▼
Phase 9: UI — Policy Panel & Time Controls
    │
    ▼
Phase 10: UI — Charts & Data Display
    │
    ▼
Phase 11: Console
    │
    ▼
Phase 12: Game Modes (Sandbox & Scenario)
    │
    ▼
Phase 13: Map View
    │
    ▼
Phase 14: Data Files & Validation
    │
    ▼
Phase 15: Balancing & Polish
```

---

## Phase 0: Project Setup

**Goal:** Repository structure, build system, and tooling ready.

**Tasks:**
1. Create .NET solution file (`game.sln`)
2. Create Simulation class library project (`src/Simulation/Simulation.csproj`)
3. Create Godot project (`src/Game/` with `project.godot` and `Game.csproj`)
4. Configure `Game.csproj` to reference `Simulation.csproj`
5. Create test project (`tests/Simulation.Tests/Simulation.Tests.csproj`)
6. Create `data/base/` directory structure with placeholder JSON files
7. Create `mods/.gitkeep`
8. Add `.gitignore` for C#, Godot, and .NET artifacts
9. Verify: solution builds, tests run, Godot project opens

**Dependencies:** None

**Definition of done:** `dotnet build` succeeds, `dotnet test` runs (with zero tests), Godot opens the project without errors.

---

## Phase 1: Accounting Foundation

**Goal:** The SFC double-entry bookkeeping system works. Transactions can be recorded and verified.

**Tasks:**
1. Implement `Transaction` record (from, to, amount, category, tick, description)
2. Implement `BalanceSheet` (assets dictionary, liabilities dictionary, net position)
3. Implement `Account` (id, balance, type: asset/liability)
4. Implement `Ledger` with two sub-ledgers:
   - Reserves ledger (central bank money)
   - Deposits ledger (bank deposits)
5. Implement `RecordTransaction()` — validates and records double-entry
6. Implement `SfcChecker` — verifies all balances sum to zero
7. Write tests:
   - Transaction recording updates both accounts
   - Balance sheet assets - liabilities = net position
   - SFC check passes after valid transactions
   - SFC check fails after invalid/manual balance edit
   - Government spending creates reserves and deposits correctly
   - Taxation destroys deposits and reserves correctly

**Dependencies:** Phase 0

**Definition of done:** All accounting tests pass. A sequence of government spending + taxation results in correct balances and passes SFC check.

---

## Phase 2: Economic Agents (Core)

**Goal:** All agent types exist with balance sheets and basic state. No behavior yet.

**Tasks:**
1. Define `IAgent` interface (id, type, balance sheet)
2. Implement `Government` agent (treasury account, spending level, allocation, tax rate)
3. Implement `CentralBank` agent (reserve accounts, policy rate)
4. Implement `CommercialBank` agent (reserves, deposits, loans, bonds, equity)
5. Implement `Household` agent (class id, population, deposits, debts, employment status, reservation wage)
6. Implement `Firm` agent (sector id, deposits, capital, employees, inventory, wage, price, productivity)
7. Implement `AgentRegistry` — stores and retrieves agents by type/id
8. Initialize agents from hardcoded defaults (move to JSON in Phase 14)
9. Wire agents to accounting system (each agent owns accounts in the ledger)
10. Write tests:
    - Each agent type can be created with correct initial state
    - Agent balance sheets are registered in the ledger
    - AgentRegistry can look up agents by type and id

**Dependencies:** Phase 1

**Definition of done:** All agent types instantiate with balance sheets wired to the ledger. Registry lookups work.

---

## Phase 3: Markets & Pricing

**Goal:** Firms produce goods, set prices, households buy goods. Labor market matches workers to jobs.

**Tasks:**
1. Implement `LaborMarket`:
   - Firms post jobs with wages
   - Households sorted by reservation wage
   - Matching: households accept best-paying jobs above reservation wage
   - Track employed/unemployed counts
2. Implement `ProductionEngine`:
   - Firms determine production target (profit-driven: based on recent demand)
   - Check available inputs (labor, materials, capital)
   - Produce output up to capacity
   - Output goes to firm inventory
3. Implement `PricingEngine`:
   - Calculate unit labor cost = wages / output
   - Calculate unit material cost
   - Apply markup with demand adjustment
   - Demand adjustment: markup rises when demand > capacity, falls when slack
4. Implement `GoodsMarket`:
   - Households calculate disposable income
   - Households purchase by hierarchical needs (survival → shelter → comfort → luxury)
   - Price elasticity per need level (inelastic necessities, elastic luxuries)
   - Firms sell from inventory
   - Transactions recorded in ledger (household deposits → firm deposits)
5. Write tests:
   - Labor market matches correct number of workers to jobs
   - Workers reject jobs below reservation wage
   - Production uses available inputs correctly
   - Unit labor cost calculated correctly (wages/productivity, not raw wages)
   - Prices respond to cost changes
   - Prices respond to demand pressure
   - Households buy survival goods before comfort goods
   - Low income households spend higher share on necessities
   - All purchases recorded as proper ledger transactions

**Dependencies:** Phase 2

**Definition of done:** A full cycle works: firms hire → produce → price → households buy. All transactions in ledger. SFC check passes.

---

## Phase 4: Banking & Credit

**Goal:** Banks create money through lending. Debt service and defaults work.

**Tasks:**
1. Implement creditworthiness assessment:
   - Check borrower income
   - Check debt-to-income ratio
   - Check collateral (for secured loans)
   - Calculate risk premium
2. Implement loan creation:
   - Create new deposit in borrower account (money creation)
   - Create loan receivable on bank balance sheet
   - Record in ledger (no reserves involved — pure deposit creation)
   - Set lending rate: CB rate + spread + risk premium
3. Implement loan types:
   - Consumer loans (short-term, unsecured)
   - Mortgages (long-term, secured)
   - Business loans (for firms)
4. Implement debt service:
   - Monthly payment calculation (principal + interest)
   - Payment destroys deposit money (principal) and transfers interest to bank
   - Record in ledger
5. Implement defaults:
   - Detect when borrower cannot make payment
   - Write off loan (bank loss)
   - Remove debt from borrower
6. Write tests:
   - Loan creation increases both deposits and loan receivables
   - Total money stock increases after lending (endogenous money)
   - Debt service payment reduces money stock (principal portion)
   - Interest portion stays in system (bank revenue)
   - SFC check passes after lending cycle
   - Creditworthy borrowers get loans, non-creditworthy don't
   - Defaults correctly write off loans

**Dependencies:** Phase 2

**Definition of done:** Banks can lend, money is created, debt is serviced, defaults are handled. SFC consistency maintained throughout.

---

## Phase 5: Government Bonds

**Goal:** Government issues bonds via auction. Interest is paid. Central bank backstops.

**Tasks:**
1. Implement bond data structure (face value, coupon rate, maturity, holder)
2. Implement `BondMarket.RunAuction()`:
   - Government announces issuance amount
   - Banks and high-income households submit bids
   - Bonds allocated to highest bidders (lowest yield)
   - Central bank buys unsold bonds
3. Implement bond accounting:
   - Bond purchase: buyer deposits → reserves → treasury (swaps deposits for bonds)
   - Interest payment: treasury → reserves → deposits (government spending)
   - Maturity: treasury → reserves → deposits (government spending to repay face value)
4. Implement central bank bond operations:
   - CB can buy bonds at auction (buyer of last resort)
   - CB bond purchases credit bank reserves
5. Write tests:
   - Bond auction allocates to highest bidders
   - Central bank buys unsold bonds
   - Interest payments flow correctly
   - Bond purchase/interest/maturity all maintain SFC consistency

**Dependencies:** Phase 2, Phase 4 (banks need to exist)

**Definition of done:** Bond auctions run, interest is paid, CB backstops. All SFC-consistent.

---

## Phase 6: Investment & Lags

**Goal:** Public and private investment affect capacity. Policy changes have realistic delays.

**Tasks:**
1. Implement public investment:
   - Infrastructure spending increases capacity over time
   - Public services spending increases productivity over time
   - Public capital depreciation
2. Implement private investment:
   - Firms decide to invest based on demand expectations and capacity utilization
   - Funded from retained profits and/or bank loans
   - Capital goods purchased from industry sector
   - Capital increases productive capacity
   - Capital depreciation
3. Implement policy lag system:
   - Queue for pending policy changes with tick delay
   - Pipeline tracking (what's pending, when it takes effect)
   - Different lag durations for different policy types
4. Implement economic lags:
   - Sticky wages (slow adjustment)
   - Gradual price adjustment
   - Hiring/firing delays
5. Write tests:
   - Infrastructure investment increases capacity after correct lag
   - Productivity improves from public services after correct lag
   - Capital depreciates over time
   - Policy changes take effect after specified delay
   - Policy pipeline correctly tracks pending changes

**Dependencies:** Phase 3 (production system needs to exist), Phase 4 (firms borrow for investment)

**Definition of done:** Investment changes capacity over time. Policy lags work correctly. Pipeline tracks pending changes.

---

## Phase 7: Tick Engine Integration

**Goal:** All phases run together in a complete monthly tick. The economy simulates end-to-end.

**Tasks:**
1. Implement `TickEngine`:
   - Orchestrate phases in order: Government → Production → Market → Financial → Accounting
   - Call each subsystem in sequence
   - Run SFC check after each tick
2. Implement `IndicatorCalculator`:
   - Employment rate
   - Inflation rate (price level change from previous tick)
   - GDP (total output value)
   - Government balance
   - Private sector balance
   - Capacity utilization per sector
   - Unit labor costs per sector
   - Private debt level
   - All other indicators from PRD
3. Implement `SimulationState`:
   - Central read-only state container
   - Exposes all indicators and agent states
   - Path-based query system (`QueryByPath`)
4. Implement `SimulationCommands`:
   - Policy change handlers
   - Console command routing
5. Run integration tests:
   - Multi-tick simulation produces sensible economic dynamics
   - Government spending increases private sector balances
   - Taxation decreases private sector balances
   - Overspending causes inflation
   - Underspending causes unemployment
   - SFC identity holds every tick
   - Indicators calculate correctly

**Dependencies:** Phases 1–6

**Definition of done:** A 120-tick (10 year) simulation runs end-to-end with sensible economic dynamics and SFC consistency every tick.

---

## Phase 8: Godot Project & Game Controller

**Goal:** Godot project runs and connects to the simulation engine.

**Tasks:**
1. Set up Godot project with C# support
2. Configure project references (Game → Simulation)
3. Implement `GameController` node:
   - Initialize simulation engine
   - Manage game loop (advance ticks based on speed, handle pause)
   - Expose simulation state to child nodes
   - Route commands from UI to simulation
4. Implement time management:
   - Pause/resume state
   - Speed settings (1x, 2x, 5x)
   - Map game time to real time (e.g., 1x = 1 tick per second)
5. Create main scene with GameController as root
6. Verify simulation runs inside Godot

**Dependencies:** Phase 7

**Definition of done:** Godot runs, simulation ticks advance, speed and pause controls work.

---

## Phase 9: UI — Policy Panel & Time Controls

**Goal:** Player can control the simulation through the UI.

**Tasks:**
1. Create `PolicyPanel` scene and script:
   - Slider for government spending level
   - Three sliders for spending allocation (infrastructure, services, transfers) — constrained to sum to 100%
   - Slider for tax rate
   - Labels showing current values
   - Signals connected to GameController
2. Create `HUD` scene and script:
   - Current date display (month/year)
   - Play/pause button
   - Speed selector (1x, 2x, 5x)
   - Tick advance button (when paused)
3. Wire policy changes from UI → GameController → SimulationCommands
4. Visual feedback for pending policy changes (in pipeline indicator)

**Dependencies:** Phase 8

**Definition of done:** Player can adjust all policy levers and time controls via UI. Changes flow to simulation correctly.

---

## Phase 10: UI — Charts & Data Display

**Goal:** Player can see the economy's state through charts and data panels.

**Tasks:**
1. Evaluate/select a chart solution (Godot native, custom drawing, or lightweight library)
2. Implement `ChartPanel`:
   - Line chart component with scrolling time window
   - Support multiple data series per chart
   - Create charts for: employment rate, inflation rate, GDP, government balance, private savings, sector output
3. Implement `BalanceSheetPanel`:
   - Table display for each sector's balance sheet
   - Assets, liabilities, net position columns
   - Updated each tick
4. Implement resource utilization bars:
   - Per-sector capacity utilization bar
   - Color coded: green (slack), yellow (moderate), red (near capacity)
5. Layout all panels in the main scene

**Dependencies:** Phase 8

**Definition of done:** All key indicators visible in charts. Balance sheets displayable. Utilization bars update with simulation.

---

## Phase 11: Console

**Goal:** In-game console with query, write, and time commands.

**Tasks:**
1. Create `ConsolePanel` scene:
   - Toggle visibility with `~` key
   - Text input field at bottom
   - Scrollable output history above
   - Semi-transparent overlay
2. Implement command parser:
   - Split command into verb + arguments
   - Route to appropriate handler
3. Implement query commands:
   - `query <path>` — reads from SimulationState.QueryByPath()
   - `balance <sector>` — formats and displays balance sheet
   - `sfc check` — runs consistency check
4. Implement write commands:
   - `set government.spending <amount>`
   - `set government.allocation.<target> <pct>`
   - `set government.tax_rate <rate>`
   - `spend <amount>` / `tax <amount>` — one-time operations
5. Implement time commands:
   - `pause`, `resume`, `speed <n>`, `tick`, `tick <n>`
6. Implement `help` system
7. Implement `run <filename>` script execution
8. Implement command history (up/down arrow)

**Dependencies:** Phase 8, Phase 7 (needs SimulationState query system)

**Definition of done:** Console opens/closes, all MVP commands work, scripts execute, help displays.

---

## Phase 12: Game Modes (Sandbox & Scenario)

**Goal:** Both sandbox and scenario modes are playable.

**Tasks:**
1. Implement sandbox mode:
   - No win/lose conditions
   - Score dashboard with all key metrics
   - Free play with all controls available
2. Define scenario data format in JSON:
   - Starting economic state (initial balances, employment, etc.)
   - Objectives (target indicators, thresholds)
   - Time limit
   - Fail conditions (indicator thresholds)
3. Implement scenario loader:
   - Read scenario JSON
   - Initialize simulation to scenario starting state
   - Track objective progress
4. Implement win/lose detection:
   - Check objectives each tick
   - Check fail conditions each tick
   - Trigger appropriate UI feedback
5. Create `ScenarioUI`:
   - Objective display and progress
   - Time remaining
   - Win/lose screen
6. Create one complete scenario:
   - "Full Employment Challenge": achieve 95% employment, keep inflation below 5%, 10 year time limit
   - Fail: hyperinflation (>25%) or employment collapse (<50%)

**Dependencies:** Phase 9, Phase 10 (UI needs to exist)

**Definition of done:** Player can start sandbox or scenario mode. Scenario can be won or lost. Score dashboard works in sandbox.

---

## Phase 13: Map View

**Goal:** Cosmetic single-province map establishing visual direction.

**Tasks:**
1. Create simple 2D map asset (single province, modern/contemporary style)
2. Implement `MapView` scene:
   - Display map texture
   - Province boundary rendering
   - Basic zoom/pan
3. Add data overlays (optional MVP stretch):
   - Employment heat map coloring
   - Economic activity indicator
4. Integrate into main scene layout

**Dependencies:** Phase 8

**Definition of done:** Map displays in game. Basic visual representation of the single province.

---

## Phase 14: Data Files & Validation

**Goal:** All game data loaded from JSON. No hardcoded parameters remain.

**Tasks:**
1. Create all JSON data files in `data/base/`:
   - `economy/sectors.json` — sector definitions
   - `economy/goods.json` — goods and resources
   - `economy/production.json` — production chains
   - `economy/parameters.json` — global economic parameters
   - `agents/households.json` — household class definitions
   - `agents/firms.json` — firm behavior parameters
   - `agents/banks.json` — banking parameters
   - `agents/government.json` — government defaults
   - `scenarios/sandbox.json` — sandbox configuration
   - `scenarios/full-employment.json` — scenario definition
   - `config/simulation.json` — tick rate, lags, speeds
   - `config/ui.json` — UI configuration
2. Implement `JsonDataProvider`:
   - Load and deserialize JSON files
   - Resolve file paths
   - Schema validation with clear error messages
3. Refactor all agent initialization to use data provider instead of hardcoded values
4. Verify: changing a JSON value changes the simulation behavior without code changes
5. Write tests:
   - Data files load correctly
   - Missing required fields produce clear errors
   - Invalid values produce clear errors
   - Simulation initializes correctly from data files

**Dependencies:** Phase 7 (simulation must work before migrating to data-driven)

**Definition of done:** Zero hardcoded economic parameters. All data loaded from JSON. Changing JSON changes behavior.

---

## Phase 15: Balancing & Polish

**Goal:** The economic model produces realistic, playable dynamics.

**Tasks:**
1. Tune economic parameters:
   - Starting values for all sectors, classes, resources
   - Markup rates, productivity values, depreciation rates
   - Consumption propensities, reservation wages
   - Investment thresholds, lending criteria
2. Run extended simulations (100+ years) and check for:
   - Stability (does the economy reach a steady state without intervention?)
   - Responsiveness (do policy changes have visible effects?)
   - Realism (do dynamics match MMT predictions?)
   - Edge cases (what happens at extreme settings?)
3. UI/UX improvements:
   - Chart readability
   - Panel layout optimization
   - Clear labeling and tooltips
   - Visual polish
4. Bug fixing from playtesting
5. Write console scripts for common test scenarios

**Dependencies:** All previous phases

**Definition of done:** A new player can start, understand the controls, observe meaningful economic dynamics, and complete the scenario. Extended simulations are stable. Key MMT predictions hold (deficit = private surplus, spending into slack doesn't cause inflation, etc.).

---

## Risk Areas

| Risk | Mitigation |
|---|---|
| Economic model instability (runaway values, crashes) | Extensive unit and integration testing. SFC consistency check every tick catches accounting errors immediately. |
| Balancing difficulty (unrealistic dynamics) | Start with simple parameter values. Use console scripts to test systematically. Iterate on parameters with data files (no code changes needed). |
| Godot C# integration issues | Keep simulation as pure C# library. Minimize Godot-specific code. Test simulation independently. |
| Scope creep | Strict MVP scope defined. Features deferred to post-MVP are documented. |
| Performance (slow ticks at scale) | Population groups (not individual agents) keep computation manageable. Profile early if tick time exceeds 100ms. |
| Chart rendering in Godot | Evaluate chart solutions early in Phase 10. Custom drawing with Godot's `_Draw()` is always a fallback. |
