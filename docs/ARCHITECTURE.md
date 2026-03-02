# Architecture Document

## 1. System Overview

The game is structured as three independent layers connected through clean interfaces. The critical architectural decision is that the **simulation engine is a pure C# library with no Godot dependencies**, making it testable, portable, and modular.

```
┌──────────────────────────────────────────────────────────────────┐
│                        GODOT SCENE TREE                          │
│  ┌────────────┐  ┌────────────┐  ┌──────────┐  ┌─────────────┐  │
│  │ Map View   │  │ Charts     │  │ Policy   │  │ Console     │  │
│  │            │  │            │  │ Panel    │  │             │  │
│  └─────┬──────┘  └─────┬──────┘  └────┬─────┘  └──────┬──────┘  │
│        └───────────┬───┴──────────────┴────────────────┘         │
│                    │ reads state / sends commands                 │
│              ┌─────▼─────┐                                       │
│              │ Game       │                                       │
│              │ Controller │ (Godot node, bridges UI ↔ Sim)       │
│              └─────┬──────┘                                       │
└────────────────────┼─────────────────────────────────────────────┘
                     │
        ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─  Godot boundary
                     │
┌────────────────────▼─────────────────────────────────────────────┐
│                    SIMULATION ENGINE (pure C#)                    │
│                                                                  │
│  ┌──────────────┐  ┌───────────────┐  ┌───────────────────────┐  │
│  │ Tick Engine  │  │ Agent System  │  │ Accounting System     │  │
│  │              │  │               │  │ (SFC double-entry)    │  │
│  │ Orchestrates │  │ Government    │  │                       │  │
│  │ monthly tick │  │ Central Bank  │  │ Balance sheets        │  │
│  │ phases       │  │ Banks         │  │ Transaction log       │  │
│  │              │  │ Households    │  │ Consistency checks    │  │
│  │              │  │ Firms         │  │                       │  │
│  └──────┬───────┘  └───────┬───────┘  └───────────┬───────────┘  │
│         │                  │                      │              │
│         └──────────┬───────┴──────────────────────┘              │
│                    │                                             │
│              ┌─────▼──────┐                                      │
│              │ Data Layer │                                      │
│              │            │                                      │
│              │ JSON loader│                                      │
│              │ Schemas    │                                      │
│              │ Validation │                                      │
│              └────────────┘                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## 2. Layer Breakdown

### 2.1 Simulation Engine (Pure C# — No Godot Dependencies)

This is the heart of the game. It contains all economic logic, agent behavior, and accounting. It is a standalone C# class library that can be:

- Unit tested without Godot
- Run headlessly (for automated testing, benchmarking, or future server use)
- Developed and debugged in any C# IDE

**Key principle:** This layer knows nothing about rendering, UI, or Godot. It exposes state and accepts commands through clean interfaces.

#### Components

**Tick Engine**
- Orchestrates the monthly tick sequence (Government → Production → Market → Financial → Accounting)
- Manages simulation time (current month/year)
- Handles tick scheduling and phase ordering

**Agent System**
- Base interfaces for all economic agents
- Concrete implementations: Government, CentralBank, CommercialBank, Household, Firm
- Agent registry for looking up and iterating agents
- Agent behavior logic (production decisions, consumption, hiring, lending)

**Accounting System (SFC)**
- Double-entry transaction recording
- Balance sheet management for every agent/sector
- Two-circuit ledger: reserves ledger + deposits ledger
- Transaction validation (source and destination must exist, amounts must balance)
- Consistency checks (run post-tick, verify all balances sum to zero)
- Transaction log for debugging and console inspection

**Markets**
- Labor market: wage posting, job matching
- Goods market: pricing, buying, selling, inventory
- Bond market: auction mechanism, bidding

**Data Layer**
- JSON file loading and parsing
- Schema validation
- Parameter registry (all economic parameters accessible by path, e.g., `firms.agriculture.baseMarkup`)
- Data-driven agent initialization (sectors, classes, goods defined in data files)

### 2.2 Game Controller (Bridge Layer)

A thin Godot node that bridges the simulation engine and the UI. This is the only component that has dependencies on both Godot and the simulation.

**Responsibilities:**
- Initialize the simulation engine with loaded data
- Advance the simulation (call tick) based on game speed and pause state
- Expose simulation state to UI nodes (read-only)
- Route player commands (policy changes) from UI to simulation
- Route console commands to simulation
- Manage game modes (sandbox, scenario) and win/lose detection
- Handle save/load

**Does NOT contain:**
- Economic logic
- Agent behavior
- Rendering or UI layout

### 2.3 Presentation Layer (Godot Nodes)

Godot scenes and nodes that render the game and handle player input. These nodes read from the simulation state (via Game Controller) and send commands back.

**Components:**

**Policy Panel** — Control nodes (sliders, buttons) for player policy inputs

**Chart System** — Line charts for economic indicators, updated per tick

**Balance Sheet View** — Table display of sector balance sheets

**Map View** — 2D map rendering (cosmetic in MVP)

**Console** — Text input/output overlay, command parsing, help system

**HUD** — Time display, speed controls, mode indicator

**Scenario UI** — Objective display, progress tracking, win/lose feedback

## 3. Key Interfaces

### 3.1 Simulation State (Read)

The simulation exposes a read-only state object that the UI can query:

```csharp
public interface ISimulationState
{
    int CurrentMonth { get; }
    int CurrentYear { get; }

    IGovernmentState Government { get; }
    ICentralBankState CentralBank { get; }
    IBankingState Banks { get; }
    IReadOnlyList<IHouseholdClassState> HouseholdClasses { get; }
    IReadOnlyList<IFirmSectorState> FirmSectors { get; }

    IEconomicIndicators Indicators { get; }
    IReadOnlyList<IBalanceSheet> AllBalanceSheets { get; }

    // For console: query any value by path
    object QueryByPath(string path);
}
```

### 3.2 Simulation Commands (Write)

The UI sends commands to the simulation through a command interface:

```csharp
public interface ISimulationCommands
{
    void SetSpendingLevel(decimal amount);
    void SetSpendingAllocation(SpendingAllocation allocation);
    void SetTaxRate(decimal rate);

    // Console commands
    void ExecuteConsoleCommand(string command);

    // Time
    void Tick();
    void Tick(int count);
}
```

### 3.3 Agent Interfaces

```csharp
public interface IAgent
{
    string Id { get; }
    string Type { get; }
    IBalanceSheet BalanceSheet { get; }
}

public interface IFirm : IAgent
{
    string SectorId { get; }
    decimal PostedWage { get; }
    decimal CurrentPrice { get; }
    decimal CapacityUtilization { get; }
    decimal Productivity { get; }
    decimal UnitLaborCost { get; }
    int EmployeeCount { get; }
    decimal Inventory { get; }
}

public interface IHouseholdClass : IAgent
{
    string ClassId { get; }
    int Population { get; }
    int Employed { get; }
    decimal AverageIncome { get; }
    decimal ConsumptionSpending { get; }
    decimal SavingsBalance { get; }
    decimal DebtBalance { get; }
}
```

### 3.4 Accounting Interfaces

```csharp
public interface IBalanceSheet
{
    string OwnerId { get; }
    IReadOnlyDictionary<string, decimal> Assets { get; }
    IReadOnlyDictionary<string, decimal> Liabilities { get; }
    decimal NetPosition { get; } // Assets - Liabilities
}

public interface ITransaction
{
    string Id { get; }
    int Tick { get; }
    string FromAccount { get; }
    string ToAccount { get; }
    decimal Amount { get; }
    string Category { get; } // "spending", "taxation", "wages", "purchase", "lending", etc.
    string Description { get; }
}

public interface ILedger
{
    void RecordTransaction(string from, string to, decimal amount, string category, string description);
    IReadOnlyList<ITransaction> GetTransactions(int tick);
    IReadOnlyList<ITransaction> GetTransactions(string accountId);
    bool CheckConsistency(); // Verify SFC
}
```

### 3.5 Data Loading Interface

```csharp
public interface IDataProvider
{
    T LoadData<T>(string path);                    // Load and deserialize a JSON file
    bool TryLoadData<T>(string path, out T data);  // Non-throwing variant
    IEnumerable<string> ListFiles(string directory);
    string ResolvePath(string relativePath);        // Resolve through data/base/ (and later, mod overrides)
}
```

## 4. Data Flow

### 4.1 Simulation Tick

```
Game Controller
    │
    ├── calls Tick() on Simulation Engine
    │
    ▼
Tick Engine
    │
    ├── 1. Government Phase
    │   ├── Collect taxes → Ledger.RecordTransaction (deposits → reserves → treasury)
    │   ├── Execute spending → Ledger.RecordTransaction (treasury → reserves → deposits)
    │   ├── Pay bond interest → Ledger.RecordTransaction
    │   └── Bond auction → BondMarket.RunAuction()
    │
    ├── 2. Production Phase
    │   ├── Firms estimate demand
    │   ├── Firms set production targets
    │   ├── Firms post wages → LaborMarket.PostJobs()
    │   ├── Households accept jobs → LaborMarket.MatchJobs()
    │   └── Production occurs → Firms produce output
    │
    ├── 3. Market Phase
    │   ├── Firms set prices (cost-plus markup)
    │   ├── Households purchase goods (hierarchical needs)
    │   └── Transactions recorded → Ledger.RecordTransaction
    │
    ├── 4. Financial Phase
    │   ├── Banks process loan applications → Credit creation
    │   ├── Borrowers make debt service payments → Money destruction
    │   └── Defaults processed
    │
    └── 5. Accounting Phase
        ├── Balance sheets updated
        ├── SFC consistency check → Ledger.CheckConsistency()
        ├── Economic indicators calculated
        └── State snapshot published → Game Controller reads new state
```

### 4.2 Player Input

```
Player adjusts slider (Godot UI)
    │
    ▼
Policy Panel node emits signal
    │
    ▼
Game Controller receives signal
    │
    ▼
Game Controller calls ISimulationCommands.SetSpendingLevel(newValue)
    │
    ▼
Simulation Engine queues policy change with appropriate lag
    │
    ▼
N ticks later: policy change takes effect in Government Phase
```

### 4.3 Console Command

```
Player types command in console
    │
    ▼
Console node parses command
    │
    ├── Query command → calls ISimulationState.QueryByPath() → displays result
    │
    └── Write command → calls ISimulationCommands.ExecuteConsoleCommand()
                            │
                            ▼
                        Simulation Engine executes through proper channels
                            │
                            ▼
                        Result returned to Console → displays output
```

## 5. Project Structure

```
game/
├── game.sln                          # C# solution file
│
├── src/
│   ├── Simulation/                   # Pure C# class library (no Godot deps)
│   │   ├── Simulation.csproj
│   │   ├── Core/
│   │   │   ├── TickEngine.cs         # Tick orchestration
│   │   │   ├── SimulationState.cs    # Central state container
│   │   │   └── SimulationCommands.cs # Command handler
│   │   ├── Accounting/
│   │   │   ├── Ledger.cs             # Double-entry ledger
│   │   │   ├── BalanceSheet.cs       # Balance sheet per agent
│   │   │   ├── Transaction.cs        # Transaction record
│   │   │   └── SfcChecker.cs         # Consistency validation
│   │   ├── Agents/
│   │   │   ├── IAgent.cs             # Agent interface
│   │   │   ├── Government.cs
│   │   │   ├── CentralBank.cs
│   │   │   ├── CommercialBank.cs
│   │   │   ├── Household.cs
│   │   │   └── Firm.cs
│   │   ├── Markets/
│   │   │   ├── LaborMarket.cs        # Wage posting and matching
│   │   │   ├── GoodsMarket.cs        # Buying and selling
│   │   │   └── BondMarket.cs         # Bond auction
│   │   ├── Economics/
│   │   │   ├── PricingEngine.cs      # Cost-plus markup calculation
│   │   │   ├── ProductionEngine.cs   # Production function
│   │   │   └── IndicatorCalculator.cs # GDP, inflation, etc.
│   │   └── Data/
│   │       ├── IDataProvider.cs      # Data loading interface
│   │       ├── JsonDataProvider.cs   # JSON file loader
│   │       └── Models/              # Data models for JSON deserialization
│   │           ├── SectorData.cs
│   │           ├── HouseholdData.cs
│   │           ├── GoodsData.cs
│   │           └── ScenarioData.cs
│   │
│   └── Game/                         # Godot project
│       ├── project.godot             # Godot project file
│       ├── Game.csproj               # Godot C# project (references Simulation)
│       ├── Scenes/
│       │   ├── Main.tscn             # Main scene
│       │   ├── UI/
│       │   │   ├── PolicyPanel.tscn
│       │   │   ├── Charts.tscn
│       │   │   ├── BalanceSheetView.tscn
│       │   │   ├── Console.tscn
│       │   │   ├── HUD.tscn
│       │   │   └── ScenarioUI.tscn
│       │   └── Map/
│       │       └── MapView.tscn
│       ├── Scripts/
│       │   ├── GameController.cs     # Bridge: Simulation ↔ Godot
│       │   ├── UI/
│       │   │   ├── PolicyPanel.cs
│       │   │   ├── ChartPanel.cs
│       │   │   ├── BalanceSheetPanel.cs
│       │   │   ├── ConsolePanel.cs
│       │   │   ├── HUD.cs
│       │   │   └── ScenarioUI.cs
│       │   └── Map/
│       │       └── MapView.cs
│       └── Assets/
│           ├── Themes/               # Godot UI themes
│           ├── Fonts/
│           └── Textures/
│
├── tests/
│   ├── Simulation.Tests/            # Unit tests for simulation engine
│   │   ├── Simulation.Tests.csproj
│   │   ├── Accounting/
│   │   │   ├── LedgerTests.cs
│   │   │   ├── SfcCheckerTests.cs
│   │   │   └── BalanceSheetTests.cs
│   │   ├── Agents/
│   │   │   ├── GovernmentTests.cs
│   │   │   ├── HouseholdTests.cs
│   │   │   ├── FirmTests.cs
│   │   │   └── BankTests.cs
│   │   ├── Markets/
│   │   │   ├── LaborMarketTests.cs
│   │   │   ├── GoodsMarketTests.cs
│   │   │   └── BondMarketTests.cs
│   │   └── Integration/
│   │       ├── TickIntegrationTests.cs
│   │       └── MoneyFlowTests.cs
│   └── Game.Tests/                   # Integration tests that may need Godot
│       └── Game.Tests.csproj
│
├── data/
│   └── base/                         # Base game data (JSON files)
│       ├── economy/
│       │   ├── sectors.json
│       │   ├── goods.json
│       │   ├── production.json
│       │   └── parameters.json
│       ├── agents/
│       │   ├── households.json
│       │   ├── firms.json
│       │   ├── banks.json
│       │   └── government.json
│       ├── scenarios/
│       │   ├── sandbox.json
│       │   └── full-employment.json
│       ├── map/
│       │   └── default-map.json
│       └── config/
│           ├── simulation.json
│           └── ui.json
│
├── docs/                             # Project documentation
│   ├── DESIGN.md
│   ├── ECONOMIC-MODEL.md
│   ├── MVP.md
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── MODDING.md
│   └── CONSOLE.md
│
└── mods/                             # Mod directory (empty in base game)
    └── .gitkeep
```

## 6. Key Design Patterns

### 6.1 Simulation as Pure Library

The simulation engine is a pure C# class library with **zero Godot dependencies**. This is enforced at the project level: `Simulation.csproj` does not reference Godot assemblies.

Benefits:
- Unit testable with standard C# test frameworks (xUnit, NUnit)
- Can run headlessly for benchmarking or automated testing
- Could be reused in other frontends (web, different engine) in the future
- Clean separation forces good architecture

### 6.2 Command Pattern for Player Actions

All player actions go through a command interface rather than directly mutating state. This:
- Centralizes all state mutations
- Makes it easy to add undo/redo later
- Makes it easy to record/replay sessions
- Enables the console to use the same commands as the UI

### 6.3 Observer Pattern for UI Updates

The simulation publishes state changes. UI nodes subscribe to changes they care about. This avoids the UI polling the simulation and keeps the coupling one-directional.

```csharp
public interface ISimulationEvents
{
    event Action<int> OnTickCompleted;            // tick number
    event Action<string, decimal> OnIndicatorChanged; // indicator name, new value
    event Action<ITransaction> OnTransactionRecorded;
    event Action<string> OnPolicyEnacted;         // policy description
}
```

### 6.4 Data-Driven Agent Configuration

Agents are not hardcoded. Their types, parameters, and behavior thresholds are loaded from JSON data files. The code provides the behavior framework; the data files configure it.

```
Code defines:  "A Firm has inputs, outputs, a markup, and produces goods"
Data defines:  "Agriculture takes labor+land, outputs food, has 15% markup"
```

This means mods can add new sectors, goods, and household classes without touching code (Tier 1 modding).

### 6.5 Path-Based State Access

All simulation state is queryable by a dot-separated path (e.g., `firms.agriculture.price`). This is used by:
- The console (`query firms.agriculture.price`)
- The data binding system (charts can reference `economy.inflation_rate`)
- Future mod scripts

Implemented via a state tree or registry that maps paths to getters.

## 7. Technology Stack

| Component | Technology |
|---|---|
| Game engine | Godot 4.x (.NET variant) |
| Simulation language | C# (.NET 8+) |
| UI scripting | C# |
| Data format | JSON |
| Unit testing | xUnit (or NUnit) |
| Build system | .NET SDK / MSBuild |
| Version control | Git |

## 8. Constraints and Decisions

### 8.1 Why Simulation is a Separate Library

The most important architectural decision. Alternatives considered:

| Approach | Rejected because |
|---|---|
| All code in Godot project | Can't unit test without Godot. Tight coupling. Hard to refactor. |
| Simulation as Godot autoload | Still coupled to Godot. Can't run headlessly. |
| **Separate C# library** (chosen) | Clean separation. Testable. Portable. Slightly more project setup. |

### 8.2 Why Not ECS (Entity Component System)

Godot and some game engines push ECS. We use a more traditional OOP agent model because:
- Economic agents have complex behavior that maps well to objects
- The SFC accounting system is inherently relational (balance sheets, transactions)
- Population groups are not entities with interchangeable components
- ECS adds complexity without clear benefit for this simulation type

### 8.3 Single-Threaded Simulation

The simulation runs on a single thread. Multi-threading adds complexity and is unnecessary for the MVP:
- Monthly ticks are sequential by nature (phases depend on previous phases)
- Population groups (not individual agents) keep the computation manageable
- Godot's main thread handles rendering; simulation can run on a background thread if needed later
