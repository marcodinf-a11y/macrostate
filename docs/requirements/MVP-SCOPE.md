# MVP Scope

This document centralizes all MVP scoping decisions. The main design and requirements documents describe the **full project vision**; this document defines which subset is built first.

For the phased build sequence, see [MVP-IMPLEMENTATION-PLAN.md](../technical/MVP-IMPLEMENTATION-PLAN.md).

## Definition of Done

The MVP is complete when:

1. **SFC accounting works** — all transactions are double-entry; balances always sum to zero
2. **Two money circuits visible** — reserves and deposits tracked and displayable
3. **Policy controls work** — player can adjust spending, allocation, and tax rate
4. **Money circuit flows** — government spending creates currency; taxation destroys it
5. **Production runs** — firms produce based on profit-driven decisions; households consume via AIDS demand system
6. **Labor market functions** — firms post wages, households accept/reject, unemployment emerges naturally
7. **Bank lending works** — banks create money via loans; repayments destroy it
8. **Bond auctions work** — government issues bonds; banks bid; CB backstops
9. **Inflation responds correctly** — driven by unit labor costs and demand pressure, not raw spending
10. **Charts update live** — all key indicators chart in real-time
11. **Scenario playable** — one scenario with win/lose conditions completable
12. **Sandbox works** — free experimentation mode with full dashboard
13. **Pause and speed controls work** — player can control simulation flow

## MVP Simplifications

The MVP implements the full economic model with these simplifications:

| Aspect | MVP simplification | Full vision | Reference |
|---|---|---|---|
| Household classes | 3 fixed classes | 5+ classes or continuous spectrum | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Households |
| Sector granularity | 4 top-level sectors | Sub-sector hierarchy (8-15 sectors) via parentId | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Firms |
| Foreign sector | None (closed economy) | Multiple AI nations with trade | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) SFC Framework |
| Bank competition | Single aggregate bank | Multiple competing banks | [ARCHITECTURE.md](../technical/ARCHITECTURE.md) Banking |
| Bond secondary market | No resale | Full secondary market | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Bonds |
| Central bank policy rate | Fixed at 0 | Player-adjustable or rule-based | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Central Bank |
| Provinces | Single province | Multiple geographic provinces | [GAME-DESIGN.md](../design/GAME-DESIGN.md) Map |
| Firm heterogeneity | One representative firm per sector | Multiple competing firms per sector | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Firms |
| Wage negotiation | Simple posting/acceptance | Collective bargaining, contracts | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) Labor Market |
| Government spending types | 3 (infrastructure, services, transfers) | More granular spending categories | [PRD.md](PRD.md) FR-CTL-001 |
| AIDS parameters | US-based estimates | Country-specific parameter sets | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) AIDS |
| Currency demand | Implicit — single currency, no alternative exists | Explicit tax-driven currency demand for exchange rate dynamics | [ECONOMIC-MODEL.md](../design/ECONOMIC-MODEL.md) |
| Map | Cosmetic single-province map | Geographic gameplay with provinces and resources | [PRD.md](PRD.md) FR-UI-005 |

## Out of Scope

The following are explicitly deferred beyond the MVP:

| Feature | Reason for deferral |
|---|---|
| Job Guarantee program | Build basic employment dynamics first |
| Foreign sector / trade / sectoral balances (3-sector) | Requires foreign sector |
| Multiple economies / AI nations | Focus on domestic dynamics first |
| Multiple provinces / geographic map gameplay | One province proves the economic model |
| Individual agent simulation (household classes only) | Household classes sufficient initially |
| Monetary policy (CB rate changes) | CB rate fixed at 0 initially |
| Multiple tax types | Single income tax; progressive/sales/wealth taxes later |
| Bond secondary market | No resale initially |
| Multiple competing banks | Single aggregate bank initially |
| Bank insolvency / systemic crisis mechanics | Defaults tracked but no systemic crisis modeling |
| Collective bargaining / wage contracts | Simple wage posting only |
| 5+ household classes or continuous income spectrum | 3 classes initially; continuous spectrum later |
| Sub-sector hierarchy | 4 top-level sectors initially; sub-sector expansion later |
| Mod loader / mod management UI | Data-driven JSON modding sufficient initially |
| Plugin API for code mods | Data-driven JSON modding sufficient initially |
| Multiplayer | Single-player focus |
| Tutorial / guided onboarding | Sandbox and scenario modes provide sufficient onboarding |
| Save/load game state | Deterministic simulation + short sessions; full state serialization deferred (high priority) |
| Sound / music | Gameplay mechanics first; polish later |
| Localization | English only initially |

## Modding

The MVP modding infrastructure is limited to the **data-driven foundation**:

**Included:**
- All game data loaded from JSON files in `data/base/`
- No hardcoded economic parameters, sector definitions, or scenario data
- Clean separation between data and logic in the codebase
- Data schema validated on load
- Base game structured as if it were a mod (same file format, same loading path)

**Deferred:**
- Mod loader (modders can edit `data/base/` directly to test)
- Mod manifest system
- Override/merge mechanics
- Dependency resolution
- Plugin API and event hooks
- Mod management UI

The key MVP deliverable is the **architecture**: the game must be designed so that adding the mod loader later is a matter of adding a new data source, not restructuring the entire codebase.

See [MODDING.md](../technical/MODDING.md) for the full modding architecture.

## Console

The MVP console includes:

**Included:**
- Console UI (toggle with `~`, text input, scrollable output)
- `query` commands for all simulation state
- `set` commands for government policy (spending, allocation, tax rate)
- `spend` and `tax` one-time operation commands
- `balance` commands for all sectors
- `sfc check` for consistency verification
- `pause`, `resume`, `speed`, `tick` time controls
- `help` system
- `run` for basic script execution
- Command history (up/down arrow)

**Deferred:**
- Tab completion
- Wildcard queries (`*`, `**`)
- Agent spawn/remove commands
- CSV export
- Mod command registration
- `--force` flag for unsafe writes
- `sfc repair`

See [CONSOLE.md](../technical/CONSOLE.md) for the full console specification.

## UI Testing

UI testing is a future side project. See [UI-TESTING-ROADMAP.md](../technical/UI-TESTING-ROADMAP.md) and [ADR-0007](../decisions/ADR-0007-manual-ui-testing.md) for details.

## Build Sequence

See [MVP-IMPLEMENTATION-PLAN.md](../technical/MVP-IMPLEMENTATION-PLAN.md) for the phased build plan (16 phases, TDD methodology).

## Related ADRs

- [ADR-0003](../decisions/ADR-0003-banks-only-bond-auctions.md) — Banks-only bond auctions
- [ADR-0005](../decisions/ADR-0005-aids-consumption-model.md) — AIDS consumption model
- [ADR-0007](../decisions/ADR-0007-manual-ui-testing.md) — Manual UI testing with scoped testability contract
- [ADR-0008](../decisions/ADR-0008-bond-yields-definition.md) — Bond yields definition
- [ADR-0009](../decisions/ADR-0009-leontief-io-production-function.md) — Leontief I-O production function
- [ADR-0010](../decisions/ADR-0010-asymmetric-markup-sellers-inflation.md) — Asymmetric markup and seller's inflation
