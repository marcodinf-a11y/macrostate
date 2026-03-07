# UI Testing Roadmap

**Status:** Future side project — not required for MVP
**Context:** ADR-0007 (manual UI testing with scoped testability contract)

## Current State (MVP)

Presentation-layer FR groups (FR-UI, FR-CON, FR-GMD, FR-CTL-002) use manual verification for Godot-specific behavior. Automated tests exist only where logic is separable from the engine:

| What | How | Phase |
|---|---|---|
| Command parsing | Unit test: string → command mapping (pure C#) | Phase 12 |
| Scenario win/lose detection | Headless via `SimulationTestHarness` | Phase 13 |
| Policy signal wiring | Console-based integration scripts | Phase 10 |

This is sufficient for MVP because the simulation engine — where all economic logic lives — is fully TDD-tested (Phases 2–8). The UI is a thin read/write layer over `ISimulationState` and `ISimulationCommands`.

## What Full Automated UI Testing Requires

### 1. Headless Godot Runtime

Godot can run scenes without rendering via `--headless` mode. This is the foundation for all automated UI testing.

**What's needed:**
- CI runner with Godot installed (or a Docker image like `barichello/godot-ci`)
- A test entry point scene that bootstraps the test runner without opening the game
- Verification that `--headless` mode correctly processes input events, signals, and scene tree operations

**Risks:**
- Some Godot UI nodes behave differently in headless mode (e.g., `Control` layout may not resolve without a viewport)
- Shader compilation and rendering-dependent code will not run

### 2. GdUnit4 (Godot Unit Testing Framework)

[GdUnit4](https://mikeschulze.github.io/gdUnit4/) is the most mature C# testing framework for Godot 4.x.

**What it provides:**
- Scene runner: instantiate scenes, simulate input, assert on node state
- Mocking and spying for Godot nodes
- Async test support (wait for signals, timeouts)
- Integration with `dotnet test` and CI

**What's needed:**
- Add `gdUnit4Api` NuGet package to `tests/Game.Tests/Game.Tests.csproj`
- Create `Game.Tests.csproj` (listed in Architecture Section 5 but deferred — see M9 resolution)
- Configure GdUnit4 test runner in the Godot project
- Write a few proof-of-concept tests to validate the setup works in headless mode

### 3. Scene Tree Mocking / Test Doubles

UI tests need to run panels in isolation without the full game scene tree.

**What's needed:**
- A `TestSceneRoot` that provides minimal required parent nodes (e.g., a `Control` root for UI panels)
- Mock implementations of `ISimulationState` and `ISimulationCommands` that return canned data
- A pattern for injecting mock simulation state into UI nodes (either via exported properties or a service locator)

**Example test structure:**
```csharp
[TestCase]
public async Task PolicyPanel_SetTaxRate_CallsSimulationCommand()
{
    var mockSim = new MockSimulationCommands();
    var panel = SceneRunner.Load<PolicyPanel>("res://scenes/PolicyPanel.tscn");
    panel.SimCommands = mockSim;

    panel.TaxRateSlider.Value = 0.25;
    panel.ApplyButton.EmitSignal("pressed");

    Assert.That(mockSim.LastTaxRate).IsEqual(0.25m);
}
```

### 4. CI Integration

**What's needed:**
- GitHub Actions workflow (or equivalent) that:
  1. Installs Godot (headless export template)
  2. Runs `dotnet test` for simulation tests (already planned)
  3. Runs GdUnit4 tests via Godot `--headless` mode
- Test result reporting (GdUnit4 supports JUnit XML output)
- Separate CI job for UI tests (they're slower and may be flaky)

### 5. What to Test Automatically

Not everything needs automated tests. Priority order:

| Priority | What | Why |
|---|---|---|
| High | Console command parser | Pure logic, easy to test, high regression risk |
| High | Scenario win/lose detection | Headless-testable, critical game logic |
| High | Policy panel → simulation command wiring | Catches broken signal connections |
| Medium | Chart data binding correctness | Verify charts read correct `ISimulationState` properties |
| Medium | Console query output formatting | String output, easy to assert |
| Medium | Balance sheet panel sector mapping | Verify `SectorBalanceSheets` keys map to correct UI elements |
| Low | Map zoom/pan | Hard to test meaningfully in headless mode |
| Low | Visual styling, layout | Better caught by manual review |

### 6. Estimated Scope

This is a side project, not a prerequisite for MVP. Rough breakdown:

- **Setup (GdUnit4 + headless CI):** Get one test running end-to-end in CI
- **Console tests:** Command parser unit tests + query output tests
- **Signal wiring tests:** One test per UI panel verifying it calls the right simulation method
- **Scenario tests:** Win/lose detection under headless simulation
- **Scene integration tests:** Panels render and update when simulation state changes

## References

- [GdUnit4 documentation](https://mikeschulze.github.io/gdUnit4/)
- [Godot headless mode](https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_dedicated_servers.html)
- [barichello/godot-ci Docker image](https://github.com/abarichello/godot-ci)
- Architecture Section 5: `Game.Tests.csproj` (deferred per M9)
- ADR-0007: Manual UI testing with scoped testability contract
