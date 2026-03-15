---
phase: 09-scenario-cards-rework
plan: 03
subsystem: ui
tags: [react, typescript, scenario-cards, effective-interest-rate, heloc, rental-cash-flow, mini-mortgage-calculator, sb7, str-analyzer]

# Dependency graph
requires:
  - phase: 09-scenario-cards-rework
    plan: 01
    provides: "calcEffectiveInterestRate, getHorizonDates, HorizonYears, StayAndInvestProjection, MoveAndKeepProjection, SELL_COMMISSION_RATE, SELL_CLOSING_RATE"
  - phase: 09-scenario-cards-rework
    plan: 02
    provides: "HorizonTabs, MiniMortgageCalculator, StayAndBuildCard, SellAndMoveUpCard, ScenarioCard with horizons/initialHorizon props"
provides:
  - "StayAndInvestCard with HELOC projections, tax advantages callout, STR Analyzer link, Josh & Jacqui CTA"
  - "MoveAndKeepAsRentalCard with effective interest rate hero metric, rental cash flow, MiniMortgageCalculator"
  - "Fully refactored ScenarioCards.tsx rendering all 4 new scenario cards with date-based horizons"
  - "Updated ScenarioCards tests covering new card titles, effective rate, tax callout, STR link"
affects: [homeowner-page, scenario-cards]

# Tech tracking
tech-stack:
  added: []
  patterns: [effective-rate-hero-metric, heloc-investment-framing, rental-cashflow-offset, str-analyzer-cross-sell]

key-files:
  created:
    - src/components/homeowner/StayAndInvestCard.tsx
    - src/components/homeowner/MoveAndKeepAsRentalCard.tsx
  modified:
    - src/components/homeowner/ScenarioCards.tsx
    - src/components/homeowner/ScenarioCards.test.tsx

key-decisions:
  - "Effective rate only shown when rental cash flow is positive (rent > old mortgage) -- negative cash flow gets advisory message instead"
  - "StayAndInvestCard defaults to 5yr tab per pattern from StayAndBuildCard"
  - "MoveAndKeepAsRentalCard defaults to 0yr (Now) tab -- effective rate concept works best grounded in today's numbers"
  - "STR Analyzer URL stored as constant (STR_ANALYZER_URL) for easy updates -- points to Vercel primary deploy"
  - "ComboScenario import removed from ScenarioCards.tsx but file kept as dead code per project convention"

patterns-established:
  - "Effective rate pattern: rental cash flow offsets new mortgage, bisection solver finds equivalent rate on loan amount"
  - "Three effective rate states: computed (positive cash flow), fully-covered (rental covers new payment), negative-cashflow (advisory)"
  - "Cross-sell pattern: internal tool links (STR Analyzer) allowed in scenario cards, external links removed"

requirements-completed: [SCNR-01, SCNR-07, SCNR-08, SCNR-09, SCNR-10]

# Metrics
duration: 4min
completed: 2026-03-14
---

# Phase 9 Plan 03: Final Two Scenario Cards & ScenarioCards Rework Summary

**StayAndInvestCard with HELOC/tax/STR Analyzer framing, MoveAndKeepAsRentalCard with effective interest rate hero metric and rental cash flow offset, all four new cards wired into ScenarioCards.tsx replacing legacy cards**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-14T06:14:07Z
- **Completed:** 2026-03-14T06:18:18Z
- **Tasks:** 2
- **Files modified:** 4

## Accomplishments
- StayAndInvestCard showing HELOC available at 4 date-based horizons with investment property framing, OBBBA tax advantages callout, STR Analyzer cross-sell link, and Josh & Jacqui warm CTA
- MoveAndKeepAsRentalCard with rental cash flow (rent minus old mortgage), embedded MiniMortgageCalculator for new home, and the effective interest rate hero metric that reframes the "rates are too high" objection
- Effective rate computation handles three states: computed rate (when rental cash flow is positive), fully-covered (rental covers entire new payment showing 0.00%), and negative cash flow (advisory message)
- ScenarioCards.tsx fully refactored from 341 lines of legacy projection code to 107 lines rendering four new self-contained cards with shared date-based horizons
- All 239 tests pass including 9 ScenarioCards tests covering new card titles, horizon tabs, effective rate section, tax depreciation callout, and STR Analyzer link

## Task Commits

Each task was committed atomically:

1. **Task 1: StayAndInvestCard and MoveAndKeepAsRentalCard** - `a4b032c` (feat)
2. **Task 2: Wire all four cards into ScenarioCards.tsx and update tests** - `3289722` (feat)

## Files Created/Modified
- `src/components/homeowner/StayAndInvestCard.tsx` - Stay & Invest scenario card with HELOC projections, tax callout, STR Analyzer link, warm CTA
- `src/components/homeowner/MoveAndKeepAsRentalCard.tsx` - Move & Keep as Rental card with effective interest rate hero metric, rental cash flow, MiniMortgageCalculator
- `src/components/homeowner/ScenarioCards.tsx` - Refactored to render 4 new cards, removed all old projection code and ComboScenario import
- `src/components/homeowner/ScenarioCards.test.tsx` - Updated for new card titles, effective rate section, tax callout, STR Analyzer link verification

## Decisions Made
- Effective rate only displayed when rental cash flow is positive (rent exceeds old mortgage) -- when negative, an advisory message encourages the homeowner to talk about strategies rather than showing a misleading higher-than-market rate
- MoveAndKeepAsRentalCard defaults to Now (year 0) tab because the effective rate concept is most compelling grounded in current numbers, unlike the other cards which default to 5yr
- STR Analyzer URL stored as `const STR_ANALYZER_URL = "https://stranalyzer.vercel.app"` placeholder constant -- Vercel is the primary deploy per CLAUDE.md
- ComboScenario.tsx file kept as dead code per project convention (same as SP500Callout.tsx and RentVsOwn.tsx from Phase 6), only the import was removed from ScenarioCards.tsx

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Phase 9 (Scenario Cards Rework) is fully complete with all 3 plans executed
- All four scenario cards render with date-based horizons, market rate slider, mini mortgage calculators, effective interest rate, and contextual CTAs
- The homeowner page now has four distinct decision paths: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental

## Self-Check: PASSED

- All 4 component files verified present on disk
- Both commits (a4b032c, 3289722) verified in git log

---
*Phase: 09-scenario-cards-rework*
*Completed: 2026-03-14*
