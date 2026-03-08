# Full Document Review -- Consolidated Report

**Date:** 2026-03-07
**Scope:** All Macrostate project documents
**Method:** Five specialist agents (Economic Accuracy, Cross-Document Consistency, Completeness & Gaps, Technical Feasibility, Internal Consistency) reviewed independently; findings merged by integration agent.

---

## 1. Executive Summary

| Severity     | Raw Findings | After Dedup |
|-------------|-------------|-------------|
| Critical     | 3           | 2           |
| Major        | 28          | 20          |
| Minor        | 27          | 21          |
| Observation  | 22          | 10          |
| **Total**    | **80**      | **53**      |

**Key themes:**

1. **ARCHITECTURE.md post-ADR drift** -- Several ADR decisions (ADR-0002, ADR-0011) were never back-ported into ARCHITECTURE.md, creating contradictions between the authoritative decision record and the architecture spec.
2. **Console spec is critically misaligned** -- CONSOLE.md path roots, casing conventions, and entity names diverge from ARCHITECTURE.md on nearly every entity.
3. **Data file specs are skeletal** -- JSON schemas, example files, and parameter homes for post-ADR additions (Omega coefficients, mobility parameters, AIDS structure) are missing or scattered.
4. **Core behavioral functions left as placeholders** -- Firm demand estimation and private investment decisions are unspecified, blocking implementation.
5. **Stale references persist** -- Removed documents and resolved findings are still referenced.

---

## 2. Consolidated Findings

Each finding has a unique ID (FDR-NNN), severity, affected locations, description, suggested fix, and discovering agents.

### Critical

#### FDR-001: Console path schema critically mismatched across documents [FIXED]
- **Severity:** Critical
- **Locations:** CONSOLE.md (all query/set examples), ARCHITECTURE.md (Section 3.9 state schema)
- **Agents:** Agent 2 (C-002, C-003, C-004, C-005), Agent 4 (T-015), Agent 5 (I-011)
- **Description:** CONSOLE.md uses root paths and property names that do not exist in ARCHITECTURE.md:
  - `banks.*` vs `bank.*` (plural vs singular)
  - `economy.*` vs `indicators.*` (different root entirely)
  - `cb.*` vs `centralbank.*` (abbreviation vs full name)
  - `government.tax_rate` (snake_case) vs `government.taxRate` (camelCase)
  - `government.balance` has no matching path (should be `centralbank.treasuryAccountBalance`)
  - `banks.loans.total` has no matching path (should be `bank.loansOutstanding`)
- **Suggestion:** Rewrite CONSOLE.md query examples to match ARCHITECTURE.md schema exactly. Adopt camelCase throughout. Pick singular `bank` and full `centralbank` as canonical roots.

#### FDR-002: No JSON data files or formal schemas exist [FIXED]
- **Severity:** Critical
- **Locations:** `data/base/` (empty), MODDING.md, PARAMETERS-COMMENTARY.md, PRD FR-GMD-002
- **Agents:** Agent 3 (G-014, G-015, G-016, G-017, G-023)
- **Description:** The `data/base/` directory contains only PARAMETERS-COMMENTARY.md. No JSON files exist for any of the 12 expected data files. No formal JSON schema definitions exist. The consumption.json structure (AIDS parameters) is completely unspecified. The relationship between production.json and sectors.json is unclear (sectors.json already includes inputCoefficients). Scenario JSON has no example despite being required by FR-GMD-002.
- **Suggestion:** Before Phase 1, create a `data/schemas/` directory with JSON Schema definitions for all data files. Create minimal but complete example JSON files in `data/base/`. Resolve the production.json vs sectors.json overlap by deciding whether input-output coefficients live in sectors.json alone.

### Major

#### FDR-003: ARCHITECTURE.md Buffer 2 uses capacity-utilization language but inventory-ratio formula [FIXED]
- **Severity:** Major
- **Locations:** ARCHITECTURE.md Section 3.10 (lines 562-563 prose, line 573 CapacityThreshold, line 586 formula), ADR-0011
- **Agents:** Agent 1 (E-002), Agent 2 (C-009), Agent 4 (T-012), Agent 5 (I-006)
- **Description:** The prose and parameter name (CapacityThreshold) describe a capacity-utilization approach, but the formula correctly uses TargetInventoryRatio/ActualInventoryRatio per ADR-0011. This internal contradiction was found by all four relevant agents, giving highest confidence.
- **Suggestion:** Rewrite Buffer 2 prose to use inventory language. Rename CapacityThreshold to InventoryThreshold or remove it in favor of the TargetInventoryRatio parameter already in the formula.

#### FDR-004: Policy pipeline queue-then-overwrite model contradicts ADR-0002 [FIXED]
- **Severity:** Major
- **Locations:** ARCHITECTURE.md Section 3.8, ADR-0002, PRD FR-TIM-001
- **Agents:** Agent 2 (C-010), Agent 4 (T-001), Agent 5 (I-004, I-007)
- **Description:** ARCHITECTURE.md describes a queue-then-overwrite model for overlapping policy changes. ADR-0002 specifies replace-in-place semantics. PRD FR-TIM-001 aligns with ADR-0002. ARCHITECTURE.md is the outlier.
- **Suggestion:** Rewrite ARCHITECTURE.md Section 3.8 to match ADR-0002's replace-in-place model.

#### FDR-005: "Financial intermediation" terminology contradicts endogenous money framework [FIXED]
- **Severity:** Major
- **Locations:** ECONOMIC-MODEL.md (line 200), ADR-0005 (line 36)
- **Agents:** Agent 1 (E-001, E-008)
- **Description:** "Financial intermediation" implies the loanable-funds view (banks as middlemen), contradicting the endogenous money / credit creation framework the model explicitly adopts. The document correctly describes credit creation mechanics elsewhere.
- **Suggestion:** Replace "financial intermediation" with "credit creation and payments sector" in both documents.

#### FDR-006: Firm demand estimation mechanism completely unspecified [FIXED]
- **Severity:** Major
- **Locations:** ECONOMIC-MODEL.md, ARCHITECTURE.md, PRD
- **Agents:** Agent 3 (G-034), Agent 4 (T-013)
- **Description:** No document specifies how firms estimate demand for production planning. This is a core behavioral rule that blocks Phase 4 implementation. Without it, the production quantity decision has no basis.
- **Suggestion:** Add a demand estimation rule (e.g., adaptive expectations: expected demand = weighted average of recent sales) to ECONOMIC-MODEL.md and derive an FR in the PRD.

#### FDR-007: Private investment decision function is a placeholder [FIXED]
- **Severity:** Major
- **Locations:** ECONOMIC-MODEL.md (f(...) placeholder), ARCHITECTURE.md
- **Agents:** Agent 3 (G-035)
- **Description:** The investment decision function is left as `f(...)` with no specified form. This is one of the most consequential behavioral rules in the simulation -- it drives capital accumulation, credit demand, and growth dynamics.
- **Suggestion:** Specify a concrete investment function (e.g., accelerator model based on capacity utilization and profit rate) in ECONOMIC-MODEL.md and create an ADR if the choice is non-obvious.

#### FDR-008: MODDING.md sectors.json missing required fields from ADRs [FIXED]
- **Severity:** Major
- **Locations:** MODDING.md sectors.json example, ADR-0011, ADR-0013, ADR-0014
- **Agents:** Agent 2 (C-006, C-008), Agent 3 (G-020)
- **Description:** The sectors.json example in MODDING.md is missing:
  - `targetInventoryRatio` (required by ADR-0011 and FR-PRC-001)
  - Omega wage-equation coefficients (from ADR-0013)
  - Labor mobility parameters (from ADR-0014)
- **Suggestion:** Update the MODDING.md sectors.json example to include all required fields. Also update PARAMETERS-COMMENTARY.md with these parameters.

#### FDR-009: Omega wage coefficients have no documented home file [FIXED]
- **Severity:** Major
- **Locations:** ADR-0013, PRD FR-LBR-001 (define the coefficients), no JSON file hosts them
- **Agents:** Agent 3 (G-018)
- **Description:** The wage equation Omega coefficients are defined in ADR-0013 and referenced in PRD but have no designated JSON data file. They are absent from both the sectors.json example and PARAMETERS-COMMENTARY.md.
- **Suggestion:** Add Omega coefficients to sectors.json (per-sector wage parameters) and document them in PARAMETERS-COMMENTARY.md.

#### FDR-010: Labor mobility parameters have no documented home file [FIXED]
- **Severity:** Major
- **Locations:** ADR-0014 (defines parameters), no JSON file hosts them
- **Agents:** Agent 3 (G-019)
- **Description:** Labor mobility parameters defined in ADR-0014 have no designated JSON data file.
- **Suggestion:** Add mobility parameters to sectors.json and document in PARAMETERS-COMMENTARY.md.

#### FDR-011: No FR for household borrowing behavior [FIXED]
- **Severity:** Major
- **Locations:** PRD (missing FR), ECONOMIC-MODEL.md
- **Agents:** Agent 3 (G-004)
- **Description:** Firms have investment decision functions but households have no equivalent borrowing decision. No FR specifies when or why households borrow, yet the banking system supports household loans.
- **Suggestion:** Either add an FR for household borrowing decisions or explicitly defer household borrowing to post-MVP and document this in MVP-SCOPE.md.

#### FDR-012: A-series ambiguities A8-A13 still open
- **Severity:** Major
- **Locations:** prd-review.md, PRD, ECONOMIC-MODEL.md
- **Agents:** Agent 3 (G-011)
- **Description:** Six ambiguities, three now resolved:
  - ~~A8: Bond auction type (uniform vs discriminatory)~~ → Resolved: ADR-0017 (uniform price)
  - ~~A9: Bond maturity structure~~ → Resolved: single fixed 12-month (MVP), multiple maturities (full game)
  - ~~A10: Creditworthiness DTI thresholds~~ → Resolved: DSCR ≥ 1.25 for firms, DTI ≤ 0.40 for households
  - ~~A11: Loan durations and amortization schedule~~ → Resolved: fixed amortizing installments; households 60 ticks, firms matched to capital life (capped 120 ticks)
  - A12: Price level weighting method
  - A13: Population counts and household reclassification triggers
- **Suggestion:** Create ADRs for A8-A13, or document explicit "deferred to implementation" decisions with sensible defaults.

#### FDR-013: Edge cases A18-A21, A23 still open
- **Severity:** Major
- **Locations:** prd-review.md, PRD
- **Agents:** Agent 3 (G-013)
- **Description:** Five edge cases have no specified handling:
  - A18: All firms in a sector bankrupt
  - A19: Bank at zero reserves
  - A20: 100% unemployment
  - A21: Zero government spending
  - A23: Economic collapse definition / game-over trigger
- **Suggestion:** Document handling for each edge case in the PRD or create a dedicated ADR for edge-case behavior.

#### FDR-014: Zero inventory / zero sales division-by-zero in DemandAdjustmentFactor [FIXED]
- **Severity:** Major
- **Locations:** ARCHITECTURE.md (DemandAdjustmentFactor formula), ADR-0011
- **Agents:** Agent 3 (G-028)
- **Description:** The DemandAdjustmentFactor formula divides by ActualInventoryRatio, which can be zero if inventory is zero. No guard clause or edge-case handling is specified.
- **Suggestion:** Add a guard clause: if ActualInventoryRatio = 0, set DemandAdjustmentFactor to its maximum value (treat as maximum excess demand).

#### FDR-015: No scenario win/lose detection architecture [FIXED]
- **Severity:** Major
- **Locations:** ARCHITECTURE.md (missing component), PRD FR-GMD-002
- **Agents:** Agent 4 (T-016)
- **Description:** FR-GMD-002 requires scenario win/lose conditions as an MVP feature, but ARCHITECTURE.md has no IScenarioEngine interface or equivalent component. No architectural home for this logic.
- **Suggestion:** Add a ScenarioEngine component to ARCHITECTURE.md that evaluates win/lose predicates each tick against the simulation state.

#### FDR-016: A15 wage/procurement spending split needs defaults
- **Severity:** Major
- **Locations:** prd-review.md A15, PRD FR-CTL-001
- **Agents:** Agent 3 (G-012)
- **Description:** Government spending allocation between wages and procurement has no documented default split. FR-CTL-001 provides the policy lever but no initial value.
- **Suggestion:** Add a default wage/procurement split ratio to the government parameters in PARAMETERS-COMMENTARY.md.

#### FDR-017: CapacityThreshold parameter has no default value
- **Severity:** Major
- **Locations:** ARCHITECTURE.md, PARAMETERS-COMMENTARY.md
- **Agents:** Agent 3 (G-008)
- **Description:** The CapacityThreshold parameter (which should become an inventory-related threshold per FDR-003) has no default value or empirical basis documented anywhere.
- **Suggestion:** After renaming per FDR-003, provide a default value with empirical justification in PARAMETERS-COMMENTARY.md.

#### FDR-018: ECONOMIC-MODEL.md "two channels" but lists four items [FIXED]
- **Severity:** Major
- **Locations:** ECONOMIC-MODEL.md (money creation/destruction section)
- **Agents:** Agent 5 (I-001)
- **Description:** Text says "two legitimate channels" but then lists four items (2 creation + 2 destruction). The count is wrong or the framing is misleading.
- **Suggestion:** Change to "two creation channels and two destruction channels" or restructure as "two paired channels (creation and its reverse)."

#### FDR-019: Replay performance degrades linearly with game length
- **Severity:** Major
- **Locations:** ARCHITECTURE.md (save/load via replay), ADR-0006
- **Agents:** Agent 4 (T-008)
- **Description:** Save/load via replay (recording policy inputs per tick) scales linearly: tick 1000 takes ~1s, tick 5000 ~5s, tick 12000 ~12s. The claim that "replaying without rendering is fast" is overly optimistic for long games. This is post-MVP but the architectural choice has long-term implications.
- **Suggestion:** Document the linear scaling limitation. Consider checkpoint-based saves (periodic full state snapshots + replay from last checkpoint) as a mitigation for post-MVP.

#### FDR-020: FR-AGT-001a sovereign currency invariants lack dedicated tests
- **Severity:** Major
- **Locations:** PRD FR-AGT-001a (INV-GOV-001, INV-GOV-002), MVP-IMPLEMENTATION-PLAN.md
- **Agents:** Agent 3 (G-026)
- **Description:** The sovereign currency invariants (government never involuntarily defaults, spending creates money) are core MMT assertions but have no dedicated test entries in the implementation plan.
- **Suggestion:** Add explicit invariant tests to the relevant implementation phase (likely Phase 3 or Phase 7).

#### FDR-021: Bank parameters unspecified in data files
- **Severity:** Major
- **Locations:** PARAMETERS-COMMENTARY.md, expected banks.json
- **Agents:** Agent 3 (G-022)
- **Description:** Bank parameters (interest rate spreads, DTI thresholds, reserve requirements, etc.) have no example JSON and no entries in PARAMETERS-COMMENTARY.md.
- **Suggestion:** Add bank parameters to PARAMETERS-COMMENTARY.md and create a banks.json example.

#### FDR-022: consumption.json structure completely unspecified [FIXED]
- **Severity:** Major
- **Locations:** ECONOMIC-MODEL.md (AIDS model), expected consumption.json
- **Agents:** Agent 3 (G-016)
- **Description:** The AIDS model requires alpha, beta, and gamma parameters per household class per sector. No JSON structure is specified for these.
- **Suggestion:** Define the consumption.json schema with AIDS parameters organized by household class.

### Minor

#### FDR-023: Stale "F10" reference in ECONOMIC-MODEL.md [FIXED]
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md, formerly mmt-accuracy.md
- **Agents:** Agent 2 (C-013), Agent 3 (G-001), Agent 5 (I-002)
- **Description:** ECONOMIC-MODEL.md references a finding "F10" from the removed mmt-accuracy.md. The reference is now dangling.
- **Suggestion:** Remove the F10 reference. If the CB standing facility / corridor system concept is worth preserving, add it as a post-MVP note.

#### FDR-024: ARCHITECTURE.md project structure lists removed mmt-accuracy.md [FIXED]
- **Severity:** Minor
- **Locations:** ARCHITECTURE.md (project structure section)
- **Agents:** Agent 2 (C-014), Agent 5 (I-008)
- **Description:** The file tree in ARCHITECTURE.md still lists `docs/reviews/mmt-accuracy.md`, which was removed.
- **Suggestion:** Remove the entry. Also add MVP-SCOPE.md and UI-TESTING-ROADMAP.md which are missing from the tree (per I-009).

#### FDR-025: ARCHITECTURE.md project structure missing MVP-SCOPE.md and UI-TESTING-ROADMAP.md [FIXED]
- **Severity:** Minor
- **Locations:** ARCHITECTURE.md (project structure section)
- **Agents:** Agent 5 (I-009)
- **Description:** Two existing documents are not listed in the project structure tree.
- **Suggestion:** Add both to the file tree.

#### FDR-026: "Mid income" vs "Middle income" inconsistency [FIXED]
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md (deposit diagram)
- **Agents:** Agent 2 (C-001)
- **Description:** The deposit diagram uses "Mid income" while all other documents use "Middle income."
- **Suggestion:** Change to "Middle income" for consistency.

#### FDR-027: ADR-0009 uses `inputWeights` but MODDING.md uses `inputCoefficients`
- **Severity:** Minor
- **Locations:** ADR-0009, MODDING.md
- **Agents:** Agent 2 (C-007)
- **Description:** Stale terminology in ADR-0009. The canonical name is `inputCoefficients`.
- **Suggestion:** Update ADR-0009 to use `inputCoefficients`, or add a note that the term was renamed.

#### FDR-028: ECONOMIC-MODEL.md markup formula missing partial adjustment step
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md (markup pricing section)
- **Agents:** Agent 1 (E-003)
- **Description:** ECONOMIC-MODEL.md presents the markup formula without the partial adjustment step (`CurrentMarkup += speed * (TargetMarkup - CurrentMarkup)`). PRD, ADR-0011, and ARCHITECTURE.md all include it. Without the adjustment step, the `markupUpwardSpeed` / `markupDownwardSpeed` parameters are purposeless.
- **Suggestion:** Add the partial adjustment step to ECONOMIC-MODEL.md.

#### FDR-029: G&L wage equation absent from ECONOMIC-MODEL.md
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md (should contain it), PRD FR-LBR-001, ADR-0013
- **Agents:** Agent 1 (E-005)
- **Description:** The wage equation is defined in PRD and ADR-0013 but missing from ECONOMIC-MODEL.md, the primary economic design document.
- **Suggestion:** Add the wage equation to ECONOMIC-MODEL.md's labor market section.

#### FDR-030: ARCHITECTURE.md depreciation formula missing +I(t) investment term
- **Severity:** Minor
- **Locations:** ARCHITECTURE.md (capital accumulation formula)
- **Agents:** Agent 1 (E-006)
- **Description:** Shows only decay (`Capital_t+1 = Capital_t * (1-d)`) without the investment addition term present in ADR-0012 and PRD.
- **Suggestion:** Add `+ I(t)` to the formula.

#### FDR-031: ECONOMIC-MODEL.md tick sequence missing investment and depreciation steps
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md (tick sequence)
- **Agents:** Agent 5 (I-003)
- **Description:** The tick sequence outline omits investment and depreciation, though ARCHITECTURE.md includes them.
- **Suggestion:** Add investment and depreciation to the tick sequence.

#### FDR-032: Save/load commands in CONSOLE.md lack post-MVP annotation [FIXED]
- **Severity:** Minor
- **Locations:** CONSOLE.md (save/load commands)
- **Agents:** Agent 5 (I-012)
- **Description:** Console save/load commands are documented without noting they are post-MVP, contradicting ADR-0006's deferral.
- **Suggestion:** Add a `[post-MVP]` annotation to save/load commands in CONSOLE.md.

#### FDR-033: PRD testability contract does not cover FR-CTL-001
- **Severity:** Minor
- **Locations:** PRD (testability section), MVP-IMPLEMENTATION-PLAN.md Phase 8
- **Agents:** Agent 5 (I-005)
- **Description:** FR-CTL-001 (policy levers) is tested in Phase 8 but not listed in the PRD's testability contract.
- **Suggestion:** Add FR-CTL-001 to the testability contract.

#### FDR-034: IGovernmentState and ICentralBankState do not extend IAgent
- **Severity:** Minor
- **Locations:** ARCHITECTURE.md (interface definitions)
- **Agents:** Agent 4 (T-002)
- **Description:** Government and Central Bank state interfaces don't extend IAgent, meaning AllBalanceSheets aggregation needs special-casing. Workable but should be documented as intentional.
- **Suggestion:** Either extend IAgent or add a comment explaining why these are excluded.

#### FDR-035: Phase 7 tests assume GoodsMarket handles firm-to-firm purchases
- **Severity:** Minor
- **Locations:** MVP-IMPLEMENTATION-PLAN.md Phase 4 and Phase 7
- **Agents:** Agent 4 (T-004)
- **Description:** Phase 4 defines GoodsMarket for household/government demand only, but Phase 7 tests assume it handles intermediate goods (firm-to-firm) purchases too.
- **Suggestion:** Clarify in Phase 4 or Phase 7 whether intermediate goods flow through GoodsMarket or are handled separately (e.g., via Leontief production function pre-market).

#### FDR-036: ITimeControl defined in simulation layer but implemented in Godot layer
- **Severity:** Minor
- **Locations:** ARCHITECTURE.md (layer boundaries)
- **Agents:** Agent 4 (T-009)
- **Description:** Tests will need a mock/TestTimeControl since the real implementation lives in the Godot layer.
- **Suggestion:** Note in the implementation plan that a TestTimeControl stub is needed for simulation-layer tests.

#### FDR-037: InMemoryDataProvider bypasses JSON deserialization
- **Severity:** Minor
- **Locations:** MVP-IMPLEMENTATION-PLAN.md (test strategy)
- **Agents:** Agent 4 (T-011)
- **Description:** Using InMemoryDataProvider in tests means JSON mapping bugs (field name mismatches, missing fields) are only caught if JsonDataProvider tests exist separately.
- **Suggestion:** Ensure at least one integration test per data file uses JsonDataProvider to validate deserialization.

#### FDR-038: Phase 4 government demand tests may need isolation
- **Severity:** Minor
- **Locations:** MVP-IMPLEMENTATION-PLAN.md Phase 4
- **Agents:** Agent 4 (T-006)
- **Description:** Testing government demand in isolation (without a full tick) may require direct method invocation, which isn't noted in the test plan.
- **Suggestion:** Add a note about unit-testing government demand logic independently of the tick engine.

#### FDR-039: No FR for bond secondary market, technological change, or CB corridor
- **Severity:** Minor
- **Locations:** PRD (gaps), ECONOMIC-MODEL.md (mentions these concepts)
- **Agents:** Agent 3 (G-001, G-002, G-003)
- **Description:** Three concepts mentioned in ECONOMIC-MODEL.md have no FRs even as deferred items: bond secondary market, technological change / coefficient drift, CB standing facility.
- **Suggestion:** Add post-MVP placeholder FRs or explicitly note these as out-of-scope.

#### FDR-040: Leontief zero-output cascade unaddressed
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md (production function), ARCHITECTURE.md
- **Agents:** Agent 3 (G-032)
- **Description:** Leontief production means zero availability of any input zeroes output entirely. No recovery mechanics or minimum production guarantee exists.
- **Suggestion:** Consider a minimum production floor or document that cascading collapse is intended behavior (with the edge case handled by rationing).

#### FDR-041: Mod validation missing cross-file referential integrity checks
- **Severity:** Minor
- **Locations:** MODDING.md (validation section)
- **Agents:** Agent 3 (G-033)
- **Description:** Mod validation checks individual file schemas but doesn't verify cross-file consistency (e.g., a new sector added without corresponding AIDS parameters in consumption.json).
- **Suggestion:** Add cross-file referential integrity checks to the mod validation spec.

#### FDR-042: Negative bank equity handling unspecified
- **Severity:** Minor
- **Locations:** ECONOMIC-MODEL.md, ARCHITECTURE.md
- **Agents:** Agent 3 (G-030)
- **Description:** With a single aggregate bank, negative equity can't trigger "closure." No handling is specified.
- **Suggestion:** Document that negative bank equity triggers a CB recapitalization or is flagged as a scenario-failure condition.

#### FDR-043: Map data schema is a placeholder
- **Severity:** Minor
- **Locations:** MODDING.md (map data section)
- **Agents:** Agent 3 (G-024)
- **Description:** Map data has a placeholder with no example content.
- **Suggestion:** Fill in a minimal map data example or explicitly mark as post-MVP.

### Observations

#### FDR-044: MinimumMarkup floor in PRD/ARCHITECTURE but not ADR-0011
- **Agents:** Agent 1 (E-004)
- **Description:** Not an error -- ADR defines the core mechanism; PRD/ARCHITECTURE add a safeguard floor. Consistent layering.

#### FDR-045: prd-review.md items C4, A15, A16, A22 appear resolved but not updated
- **Agents:** Agent 2 (C-015, C-016, C-017, C-018)
- **Description:** Several prd-review.md items are resolved by subsequent ADRs and PRD updates but their status hasn't been updated in the review document.
- **Suggestion:** Update prd-review.md statuses or archive the document.

#### FDR-046: ISimulationCommands.Tick() vs ITimeControl.Tick() naming overlap
- **Agents:** Agent 4 (T-003)
- **Description:** Two interfaces with Tick() methods at different layers. Not a bug but invites confusion. Consider renaming one (e.g., `AdvanceTick()` vs `Tick()`).

#### FDR-047: Console speed values differ from other docs
- **Agents:** Agent 5 (I-013, I-016)
- **Description:** Console supports "1, 2, 5, 10, 50, 100" while other docs say "1x, 2x, 5x." Extended range may be intentional for console power users. Worth a clarifying note.

#### FDR-048: Academic citations used consistently
- **Agents:** Agent 1 (E-007)
- **Description:** All academic citations are used correctly and in appropriate contexts. No issues.

#### FDR-049: AIDS negative budget shares correctly handled
- **Agents:** Agent 3 (G-031)
- **Description:** Clamped to [0,1] and renormalized. Has tests. No issues.

#### FDR-050: Simulation layer properly Godot-free
- **Agents:** Agent 4 (T-010)
- **Description:** Clean boundary between simulation and engine layers. No issues.

#### FDR-051: No circular within-tick dependencies
- **Agents:** Agent 4 (T-014)
- **Description:** Tick phase ordering has no circular dependencies. No issues.

#### FDR-052: Phase numbering and test counts consistent
- **Agents:** Agent 5 (I-010)
- **Description:** Implementation plan phase numbering and test counts are internally consistent.

#### FDR-053: I-O coefficients in MODDING example qualitatively consistent
- **Agents:** Agent 5 (I-015)
- **Description:** The input-output coefficients in the MODDING.md example are qualitatively consistent with ECONOMIC-MODEL.md's sector descriptions.

---

## 3. Systemic Patterns

### Pattern 1: ARCHITECTURE.md not updated after ADR decisions
**Affected findings:** FDR-003, FDR-004, FDR-030
**Root cause:** ADRs are written and approved but ARCHITECTURE.md is not back-ported. The architecture doc becomes the most stale document despite being the primary implementation reference.
**Recommendation:** Establish a checklist: every ADR that changes architecture must include an "Architecture update" section, and the ADR is not considered complete until ARCHITECTURE.md is updated.

### Pattern 2: Console spec developed in isolation
**Affected findings:** FDR-001, FDR-032, FDR-047
**Root cause:** CONSOLE.md appears to have been written without cross-referencing ARCHITECTURE.md's state schema. It uses different naming conventions, different entity roots, and includes features (save/load) that are post-MVP.
**Recommendation:** Rewrite CONSOLE.md from scratch using ARCHITECTURE.md's schema as the source of truth.

### Pattern 3: Missing data file specifications
**Affected findings:** FDR-002, FDR-008, FDR-009, FDR-010, FDR-021, FDR-022
**Root cause:** New parameters were added via ADRs (wage coefficients, mobility, inventory ratios) but never assigned to specific data files. No central registry of "which parameter lives in which JSON file."
**Recommendation:** Create a parameter-to-file mapping table in PARAMETERS-COMMENTARY.md or a new `data/README.md`. Define JSON schemas before Phase 1.

### Pattern 4: Placeholder behavioral functions
**Affected findings:** FDR-006, FDR-007, FDR-011
**Root cause:** The economic model specifies structural equations (AIDS, Leontief, SFC) but leaves agent decision functions as placeholders. These are the hardest to specify but most impactful for simulation behavior.
**Recommendation:** Prioritize specifying firm demand estimation (FDR-006) and investment decisions (FDR-007) before implementation begins. These require ADRs with literature references.

### Pattern 5: Stale cross-references
**Affected findings:** FDR-023, FDR-024, FDR-025, FDR-027, FDR-045
**Root cause:** Documents reference other documents or findings that have been removed or renamed, and no cleanup pass was done.
**Recommendation:** Do a single cleanup pass to remove all stale references. Consider adding a "last verified" date to each document header.

---

## 4. Priority Action Groups

### Group 1: Fix before implementation begins
These findings would cause incorrect code or block early phases if not addressed.

| ID | Title | Effort |
|----|-------|--------|
| FDR-001 | Console path schema overhaul | Medium |
| FDR-002 | Create JSON schemas and example data files | High |
| FDR-003 | Fix Buffer 2 capacity vs inventory language | Low |
| FDR-004 | Fix policy pipeline to match ADR-0002 | Low |
| FDR-005 | Fix "financial intermediation" terminology | Low |
| FDR-006 | Specify firm demand estimation | High |
| FDR-007 | Specify private investment function | High |
| FDR-008 | Update sectors.json example with all required fields | Low |
| FDR-014 | Add division-by-zero guard to DemandAdjustmentFactor | Low |
| FDR-015 | Add ScenarioEngine to architecture | Medium |
| FDR-018 | Fix "two channels" count | Low |
| FDR-022 | Define consumption.json structure | Medium |

### Group 2: Fix during Phase 0 setup
These are important for implementation correctness but don't block initial architecture work.

| ID | Title | Effort |
|----|-------|--------|
| FDR-009 | Assign Omega coefficients to a data file | Low |
| FDR-010 | Assign mobility parameters to a data file | Low |
| ~~FDR-011~~ | ~~Decide on household borrowing scope~~ | ~~Medium~~ |
| FDR-016 | Add wage/procurement split defaults | Low |
| FDR-017 | Add CapacityThreshold default value | Low |
| FDR-020 | Add sovereign invariant tests | Low |
| FDR-021 | Specify bank parameters | Medium |
| FDR-028 | Add partial adjustment step to ECONOMIC-MODEL.md | Low |
| FDR-029 | Add wage equation to ECONOMIC-MODEL.md | Low |

### Group 3: Fix during relevant implementation phase
These can be resolved when the developer reaches the relevant code.

| ID | Title | Phase |
|----|-------|-------|
| FDR-012 | Resolve A8-A13 ambiguities | Phase 3-5 |
| FDR-013 | Define edge case handling | Phase 7 |
| FDR-023 | Remove stale F10 reference | Any |
| FDR-024 | Fix project structure listing | Any |
| FDR-025 | Add missing docs to project structure | Any |
| FDR-026 | Fix "Mid income" typo | Any |
| FDR-027 | Fix inputWeights vs inputCoefficients | Any |
| FDR-030 | Add +I(t) to depreciation formula | Phase 5 |
| FDR-031 | Add investment/depreciation to tick sequence | Phase 5 |
| FDR-032 | Add post-MVP annotation to save/load | Any |
| FDR-033 | Add FR-CTL-001 to testability contract | Phase 8 |
| FDR-034 | Document IAgent extension decision | Phase 2 |
| FDR-035 | Clarify firm-to-firm market flow | Phase 4 |
| FDR-036 | Note TestTimeControl need | Phase 1 |
| FDR-037 | Add JsonDataProvider integration tests | Phase 1 |
| FDR-038 | Note government demand test isolation | Phase 4 |
| FDR-039 | Add post-MVP placeholder FRs | Any |
| FDR-040 | Document Leontief cascade behavior | Phase 4 |
| FDR-041 | Add cross-file mod validation | Phase 9 |
| FDR-042 | Document negative bank equity handling | Phase 5 |
| FDR-043 | Fill in map data schema | Phase 9 |

### Group 4: Track for later
No implementation risk; improvements for documentation quality.

| ID | Title |
|----|-------|
| FDR-019 | Document replay scaling limitation |
| FDR-044 | MinimumMarkup floor layering (no action needed) |
| FDR-045 | Update prd-review.md statuses |
| FDR-046 | Consider Tick() method rename |
| FDR-047 | Clarify console speed range |

---

## 5. Document Health Matrix

| Document | Accuracy | Completeness | Consistency | Implementability | Notes |
|----------|----------|-------------|-------------|-----------------|-------|
| GAME-DESIGN.md | Good | Good | Good | Good | No findings against it |
| ECONOMIC-MODEL.md | Fair | Fair | Fair | Fair | Stale terminology (FDR-005), missing formulas (FDR-028, FDR-029, FDR-031), placeholder functions (FDR-006, FDR-007), counting error (FDR-018), stale ref (FDR-023) |
| PRD.md | Good | Fair | Good | Good | Missing FRs for household borrowing (FDR-011), testability gap (FDR-033), open A-series items (FDR-012, FDR-013) |
| MVP-SCOPE.md | Good | Good | Good | Good | No findings against it |
| ARCHITECTURE.md | Fair | Fair | Poor | Fair | Post-ADR drift (FDR-003, FDR-004), stale file tree (FDR-024, FDR-025), missing components (FDR-015), formula gaps (FDR-030, FDR-014) |
| MODDING.md | Good | Fair | Fair | Fair | Missing required fields (FDR-008), stale terminology (FDR-027), placeholder sections (FDR-043), missing validation (FDR-041) |
| CONSOLE.md | Poor | Fair | Poor | Poor | Path schema critically wrong (FDR-001), post-MVP features unmarked (FDR-032), speed values differ (FDR-047) |
| MVP-IMPLEMENTATION-PLAN.md | Good | Good | Good | Fair | Test isolation concerns (FDR-035, FDR-038), mock needs (FDR-036, FDR-037) |
| UI-TESTING-ROADMAP.md | Good | Good | Good | Good | No findings against it |
| PARAMETERS-COMMENTARY.md | Good | Poor | Fair | Fair | Missing many post-ADR parameters (FDR-009, FDR-010, FDR-017, FDR-021) |
| ADRs 0001-0014 | Good | Good | Good | Good | Minor terminology issue in ADR-0005 (FDR-005) and ADR-0009 (FDR-027); otherwise solid |
| decisions/README.md | Good | Good | Good | Good | No findings against it |
| prd-review.md | Fair | Good | Fair | N/A | Stale statuses (FDR-045); useful as tracking document but needs update pass |
| docs/README.md | Good | Good | Good | Good | No findings against it |

**Rating definitions:**
- **Good:** No critical or major findings; minor issues only
- **Fair:** One or more major findings but document is largely usable
- **Poor:** Critical findings or multiple major issues that undermine the document's purpose

---

## 6. A-Series Ambiguity Status

Complete status of all A-series items from prd-review.md:

| ID | Topic | Status | Resolved By | Notes |
|----|-------|--------|-------------|-------|
| A1 | Lag value selection | Resolved | PRD FR-TIM-001 | -- |
| A2 | Depreciation specification | Resolved | ADR-0012 | -- |
| A3 | Markup adjustment mechanism | Resolved | ADR-0011 | -- |
| A4 | Buffer exhaustion thresholds | Partially resolved | ADR-0011 | CapacityThreshold has no default value (FDR-017) |
| A5 | Wage stickiness | Resolved | ADR-0013 | -- |
| A6 | Sector mobility | Resolved | ADR-0014 | -- |
| A7 | (not referenced in findings) | Presumed resolved | -- | -- |
| A8 | Bond auction type | Resolved | ADR-0017 | Uniform price (Dutch) auction |
| A9 | Bond maturity structure | Resolved | MVP-SCOPE.md, ECONOMIC-MODEL.md, PRD | Single fixed 12-month maturity (MVP); multiple maturities with yield curve (full game) |
| A10 | Creditworthiness DTI thresholds | Resolved | ECONOMIC-MODEL.md, PRD FR-BNK-002, PARAMETERS-COMMENTARY.md | Firms: DSCR ≥ 1.25; Households: DTI ≤ 0.40 |
| A11 | Loan durations / amortization | Resolved | ECONOMIC-MODEL.md, PARAMETERS-COMMENTARY.md | Households: 60-tick fixed amortizing; Firms: `min(1/depRate, 120)` ticks; Mortgages: 360-tick (full game, draft) |
| A12 | Price level weighting method | **Open** | -- | Needs ADR or documented default |
| A13 | Population counts / reclassification | **Open** | -- | Needs ADR or documented default |
| A14 | (not referenced) | Resolved | -- | -- |
| A15 | Wage/procurement spending split | Partially resolved | FR-CTL-001 | Lever exists but no default values (FDR-016) |
| A16 | CB balance sheet in SFC | Resolved | PRD FR-SIM-001 | prd-review.md not updated |
| A17 | (not referenced) | Resolved | -- | -- |
| A18 | All firms bankrupt | **Open** | -- | Edge case, needs handling spec |
| A19 | Bank at zero reserves | **Open** | -- | Edge case, needs handling spec |
| A20 | 100% unemployment | **Open** | -- | Edge case, needs handling spec |
| A21 | Zero government spending | **Open** | -- | Edge case, needs handling spec |
| A22 | (resolved) | Resolved | ADR-0004 | prd-review.md not updated |
| A23 | Economic collapse definition | **Open** | -- | Edge case, needs handling spec |

**Summary:** 9 resolved, 2 partially resolved, 11 open (6 under-specified mechanics + 5 edge cases).

---

## 7. Follow-up Recommendation

**A second review pass is recommended** after Group 1 fixes are applied, scoped narrowly to:

1. **CONSOLE.md** -- verify the rewrite aligns with ARCHITECTURE.md schema (most critical)
2. **ARCHITECTURE.md** -- verify ADR back-ports (FDR-003, FDR-004) and new ScenarioEngine component (FDR-015)
3. **Data file schemas** -- verify JSON schemas cover all parameters and are cross-consistent
4. **FDR-006 and FDR-007** -- verify the new behavioral function specs are economically sound and implementable

This follow-up should be a single-agent review focused on these four areas, not a full five-agent pass. Estimated scope: 2-3 documents, ~30 minutes.

No further review is needed for: GAME-DESIGN.md, MVP-SCOPE.md, UI-TESTING-ROADMAP.md, ADRs (except ADR-0005 terminology fix), decisions/README.md, or docs/README.md. These documents are in good shape.

---

## 8. Orchestrator Recommendation

The document suite is in strong shape for a pre-implementation project — the economic theory is correctly represented, the SFC accounting is sound, tick ordering is acyclic, and the Godot/simulation boundary is clean. The problems are concentrated in two areas: **drift** (ARCHITECTURE.md and CONSOLE.md falling behind ADR decisions) and **incompleteness** (data file specs and agent behavioral functions).

**Recommended action sequence:**

1. **Immediate (before any code):** Fix the 12 Group 1 items. The two highest-leverage tasks are:
   - Specify firm demand estimation (FDR-006) and the investment function (FDR-007) — these are the last unspecified core behavioral rules and will need ADRs with literature backing.
   - Rewrite CONSOLE.md query examples from ARCHITECTURE.md's schema (FDR-001) — this is purely mechanical but touches every example in the document.

2. **During Phase 0:** Create the JSON data file schemas (FDR-002, FDR-022) and assign all orphaned parameters to their home files (FDR-008, FDR-009, FDR-010). This naturally resolves the "missing data specs" systemic pattern.

3. **Process change:** Adopt a rule that no ADR is considered complete until its decisions are back-ported to ARCHITECTURE.md and any affected example JSON in MODDING.md. This prevents the drift pattern (Pattern 1) from recurring.

4. **After Group 1 fixes:** Run the narrow follow-up review described in Section 7. A single-agent pass over CONSOLE.md, ARCHITECTURE.md, data schemas, and the two new behavioral specs should suffice — no need for the full five-agent treatment again.

The 11 open A-series ambiguities (FDR-012, FDR-013) do not need to block implementation start — they affect Phases 3-7 and can be resolved as ADRs when each phase begins. The Group 3 minor fixes are safe for developers to resolve inline during implementation.

---

*Report generated 2026-03-07 by integration agent (Agent 6) synthesizing findings from five specialist review agents.*
