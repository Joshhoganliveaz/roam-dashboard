---
phase: 09-scenario-cards-rework
plan: 02
subsystem: ui
tags: [react, typescript, scenario-cards, mortgage-calculator, horizon-tabs, date-fns, sb7]

# Dependency graph
requires:
  - phase: 09-scenario-cards-rework
    plan: 01
    provides: "HorizonYears type, HorizonDate interface, getHorizonDates, SELL_COMMISSION_RATE, SELL_CLOSING_RATE, calcMortgagePayment, calcRemainingBalance, calcNetProceeds, StayAndBuildProjection, SellAndMoveUpProjection"
provides:
  - "HorizonTabs component with date-based labels and mobile fallback"
  - "MiniMortgageCalculator with local state, down payment presets, rate override"
  - "StayAndBuildCard showing equity growth at 4 horizons"
  - "SellAndMoveUpCard with net proceeds, 6%+1% sell costs, mini calculator, and lender CTA"
  - "ScenarioCard updated to accept HorizonYears with optional horizons and initialHorizon props"
affects: [09-03, scenario-cards, homeowner-page]

# Tech tracking
tech-stack:
  added: []
  patterns: [date-based-horizon-tabs, mini-mortgage-calculator-local-state, scenario-card-children-pattern]

key-files:
  created:
    - src/components/homeowner/MiniMortgageCalculator.tsx
    - src/components/homeowner/StayAndBuildCard.tsx
    - src/components/homeowner/SellAndMoveUpCard.tsx
  modified:
    - src/components/homeowner/HorizonTabs.tsx
    - src/components/homeowner/ScenarioCard.tsx
    - src/components/homeowner/ScenarioCards.tsx
    - src/components/homeowner/ComboScenario.tsx
    - src/components/homeowner/ScenarioCards.test.tsx

key-decisions:
  - "StayAndBuildCard defaults to 5yr tab (not Now) per research pitfall #6 -- value is in future horizons"
  - "MiniMortgageCalculator owns all state locally via useState per user decision -- never lifted"
  - "ScenarioCard horizonContent uses Partial<Record<HorizonYears, HorizonContent>> for backward compat with legacy 5/10/15 cards"
  - "SellAndMoveUpCard CTA uses inline tel link to Josh (480-369-8280) per SB7 warm pattern from Phase 7"

patterns-established:
  - "Date-based HorizonTabs with sm:inline/inline sm:hidden responsive fallback for mobile year-only labels"
  - "MiniMortgageCalculator pattern: marketRate prop as default, local useState for overrides, onPaymentChange callback for parent integration"
  - "ScenarioCard initialHorizon prop allows each card to choose its default tab"

requirements-completed: [SCNR-01, SCNR-02, SCNR-03, SCNR-04, SCNR-05, SCNR-06]

# Metrics
duration: 6min
completed: 2026-03-14
---

# Phase 9 Plan 02: Shared Components & First Two Scenario Cards Summary

**Date-based HorizonTabs with mobile fallback, MiniMortgageCalculator with 4 down payment presets and local state, StayAndBuildCard with equity trajectory, SellAndMoveUpCard with 6%+1% net proceeds and lender CTA**

## Performance

- **Duration:** 6 min
- **Started:** 2026-03-14T06:03:55Z
- **Completed:** 2026-03-14T06:10:09Z
- **Tasks:** 2
- **Files modified:** 8

## Accomplishments
- HorizonTabs refactored from numeric "5yr/10yr/15yr" labels to date-based "Mar '26/Mar '31/Mar '36/Mar '41" with year-only fallback on mobile via CSS hidden/sm:inline responsive classes
- MiniMortgageCalculator component with fully local state: home price currency input, 4 down payment preset buttons (5/10/15/20%), rate override with reset-to-market link, and estimated P&I output
- StayAndBuildCard showing equity growth across 4 horizons with SB7 narrative ("The most powerful wealth-building tool? Time.")
- SellAndMoveUpCard showing net proceeds with 6% commission + 1% closing costs, "cost to sell*" footnote, embedded MiniMortgageCalculator, and lender connect CTA via inline tel link
- Backward-compatible ScenarioCard update supporting both legacy (5/10/15) and new (0/5/10/15) horizon patterns

## Task Commits

Each task was committed atomically:

1. **Task 1: Refactor HorizonTabs to date-based + create MiniMortgageCalculator** - `4ebc353` (feat)
2. **Task 2: StayAndBuildCard and SellAndMoveUpCard components** - `af45b13` (feat)

## Files Created/Modified
- `src/components/homeowner/HorizonTabs.tsx` - Refactored to accept HorizonDate[] with date labels and mobile year fallback
- `src/components/homeowner/MiniMortgageCalculator.tsx` - New reusable mini mortgage calculator with local state
- `src/components/homeowner/ScenarioCard.tsx` - Updated to accept HorizonYears, optional horizons and initialHorizon props
- `src/components/homeowner/StayAndBuildCard.tsx` - New "Stay & Build Wealth" scenario card
- `src/components/homeowner/SellAndMoveUpCard.tsx` - New "Sell & Move Up" scenario card with mini calculator and lender CTA
- `src/components/homeowner/ScenarioCards.tsx` - Updated to use HorizonYears type for backward compat
- `src/components/homeowner/ComboScenario.tsx` - Updated to use HorizonYears type for backward compat
- `src/components/homeowner/ScenarioCards.test.tsx` - Updated test for new tab behavior

## Decisions Made
- StayAndBuildCard defaults to 5yr tab instead of Now per research pitfall #6 -- "Now" grounds reality but the value is in future projections
- ScenarioCard.horizonContent uses Partial<Record<HorizonYears, ...>> to support both legacy (5/10/15 keys) and new (0/5/10/15 keys) cards
- SellAndMoveUpCard MiniMortgageCalculator defaultHomePrice left as 0 -- homeowner enters their target price
- Legacy ScenarioCards test updated to reflect that old cards without horizons prop no longer render date-based tabs (old cards will be replaced by Plan 03)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Updated ComboScenario.tsx for HorizonYears type compatibility**
- **Found during:** Task 1 (HorizonTabs refactor)
- **Issue:** ComboScenario.tsx used local `type Horizon = 5 | 10 | 15` and `Record<Horizon, HorizonContent>` which was incompatible with ScenarioCard's updated `Partial<Record<HorizonYears, HorizonContent>>` prop type
- **Fix:** Imported `HorizonYears` from types, updated Record type and cast, added `initialHorizon={5}`
- **Files modified:** src/components/homeowner/ComboScenario.tsx
- **Verification:** TypeScript compiles cleanly
- **Committed in:** 4ebc353 (Task 1 commit)

**2. [Rule 1 - Bug] Updated ScenarioCards.test.tsx for new tab behavior**
- **Found during:** Task 1 (HorizonTabs refactor)
- **Issue:** Test expected "5yr/10yr/15yr" text in horizon tabs, but legacy cards no longer pass horizons prop so date-based tabs are not rendered
- **Fix:** Updated test to verify card expansion and detail content instead of tab labels
- **Files modified:** src/components/homeowner/ScenarioCards.test.tsx
- **Verification:** All 239 tests pass
- **Committed in:** 4ebc353 (Task 1 commit)

---

**Total deviations:** 2 auto-fixed (2 bug fixes for backward compat)
**Impact on plan:** Both fixes necessary for type safety and test suite integrity. No scope creep.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- HorizonTabs, MiniMortgageCalculator, and ScenarioCard are ready for Plan 03's two remaining cards (StayAndInvestCard, MoveAndKeepAsRentalCard)
- MiniMortgageCalculator's onPaymentChange callback is ready for effective rate integration in MoveAndKeepAsRentalCard
- Plan 03 can wire all four new cards into ScenarioCards.tsx, replacing the legacy cards

## Self-Check: PASSED

- All 5 component files verified present on disk
- Both commits (4ebc353, af45b13) verified in git log

---
*Phase: 09-scenario-cards-rework*
*Completed: 2026-03-14*
