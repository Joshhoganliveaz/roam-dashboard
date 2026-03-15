---
phase: 09-scenario-cards-rework
plan: 01
subsystem: calculations
tags: [typescript, tdd, bisection, date-fns, mortgage-math, effective-rate]

# Dependency graph
requires:
  - phase: 07-future-scenarios-rework
    provides: "calcMortgagePayment, PROJECTION_HORIZONS, existing projection types"
provides:
  - "calcEffectiveInterestRate bisection solver"
  - "getHorizonDates utility with formatted date labels"
  - "HorizonDate and HorizonYears types"
  - "Four new projection types: StayAndBuild, SellAndMoveUp, StayAndInvest, MoveAndKeep"
  - "SELL_COMMISSION_RATE (6%), SELL_CLOSING_RATE (1%), SELL_TOTAL_COST_RATE (7%)"
  - "NEW_PROJECTION_HORIZONS = [0, 5, 10, 15]"
affects: [09-02, 09-03, scenario-cards]

# Tech tracking
tech-stack:
  added: []
  patterns: [bisection-method-solver, date-based-horizon-tabs]

key-files:
  created:
    - src/lib/horizon-dates.ts
    - src/lib/horizon-dates.test.ts
  modified:
    - src/lib/types.ts
    - src/lib/calculations.ts
    - src/lib/calculations.test.ts

key-decisions:
  - "Card-local sell costs (6%+1%) separate from global DEFAULTS (5%+2%) per user decision"
  - "Bisection tolerance of $0.01 with 100 max iterations for calcEffectiveInterestRate"
  - "HorizonYears is 0|5|10|15 type union, NEW_PROJECTION_HORIZONS includes year 0"

patterns-established:
  - "Phase 9 projection types use years: HorizonYears + date: Date (vs Phase 7 year: number)"
  - "Card-local constants exported from domain module, not added to global DEFAULTS"

requirements-completed: [SCNR-02, SCNR-06, SCNR-09]

# Metrics
duration: 5min
completed: 2026-03-14
---

# Phase 9 Plan 01: Foundation Types & Calculations Summary

**Bisection-method calcEffectiveInterestRate, getHorizonDates with date-fns for 4 scenario card horizons, and 4 new projection type contracts with TDD coverage**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-14T05:56:03Z
- **Completed:** 2026-03-14T06:01:00Z
- **Tasks:** 2
- **Files modified:** 5

## Accomplishments
- calcEffectiveInterestRate bisection solver with round-trip accuracy within $0.01 for the Move & Keep "effective rate" hero metric
- getHorizonDates producing "Mar '26 | Mar '31 | Mar '36 | Mar '41" formatted labels for scenario card tabs
- Four new projection types (StayAndBuild, SellAndMoveUp, StayAndInvest, MoveAndKeep) establishing contracts for all downstream scenario card components
- 25 new tests (14 horizon + 11 effective rate) all passing alongside 90 existing calculation tests

## Task Commits

Each task was committed atomically:

1. **Task 1: Types, constants, and getHorizonDates with tests** - `2c21269` (test)
2. **Task 2: calcEffectiveInterestRate with comprehensive tests** - `d5c36f0` (feat)

## Files Created/Modified
- `src/lib/horizon-dates.ts` - getHorizonDates utility, HorizonDate type, sell cost constants
- `src/lib/horizon-dates.test.ts` - 14 tests covering all horizons, labels, and constants
- `src/lib/types.ts` - 4 new projection interfaces + HorizonYears type
- `src/lib/calculations.ts` - calcEffectiveInterestRate function + NEW_PROJECTION_HORIZONS constant
- `src/lib/calculations.test.ts` - 11 new tests for effective interest rate

## Decisions Made
- Card-local sell costs (SELL_COMMISSION_RATE=6%, SELL_CLOSING_RATE=1%) kept separate from global DEFAULTS (agentCommissionRate=5%, closingCostRate=2%) per user decision in 09-CONTEXT.md
- Bisection method with $0.01 tolerance and 100 max iterations provides convergence for all practical rates (0% to 30%)
- NEW_PROJECTION_HORIZONS = [0, 5, 10, 15] added alongside legacy PROJECTION_HORIZONS = [5, 10, 15] for backward compatibility

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- All types, calculation functions, and date utilities are exported and tested
- Phase 9 Plans 02 and 03 can import these directly to build scenario card components
- calcEffectiveInterestRate ready for Move & Keep as Rental hero metric display

## Self-Check: PASSED

- All 3 files verified present on disk
- Both commits (2c21269, d5c36f0) verified in git log

---
*Phase: 09-scenario-cards-rework*
*Completed: 2026-03-14*
