---
phase: 07-future-scenarios-rework
plan: 01
subsystem: calculations
tags: [typescript, tdd, vitest, projections, amortization, heloc, appreciation]

# Dependency graph
requires:
  - phase: 02-core-homeowner-page
    provides: calcMortgagePayment, calcRemainingBalance, calcNetProceeds, calcEquityPlay base functions
provides:
  - calcHoldAndRentProjections function (5/10/15yr Hold & Rent projections)
  - calcMoveUpProjections function (5/10/15yr Move-Up projections with purchasing power)
  - calcEquityPlayProjections function (5/10/15yr HELOC projections)
  - HorizonProjection, HoldAndRentProjection, MoveUpProjection, EquityPlayProjection type interfaces
  - DEFAULTS.rentGrowthRate constant (0.03)
  - PROJECTION_HORIZONS constant ([5, 10, 15])
affects: [07-02 (scenario cards UI), 07-03 (combo scenario)]

# Tech tracking
tech-stack:
  added: []
  patterns: [multi-horizon projection with configurable market rate, refi logic for rate comparison]

key-files:
  created: []
  modified:
    - src/lib/types.ts
    - src/lib/defaults.ts
    - src/lib/calculations.ts
    - src/lib/calculations.test.ts

key-decisions:
  - "Refi in Hold & Rent recalculates remaining balance at marketRate for new 30yr term"
  - "Projection functions accept ownershipMonths to track cumulative amortization correctly"
  - "Move-Up clamps negative net proceeds to 0 purchasing power"

patterns-established:
  - "Projection input objects: typed interfaces per scenario keeping functions pure and testable"
  - "PROJECTION_HORIZONS const for consistent 5/10/15yr across all scenarios"

requirements-completed: [SCEN-04, SCEN-08, SCEN-10]

# Metrics
duration: 2min
completed: 2026-03-14
---

# Phase 7 Plan 1: Multi-Horizon Projection Calculations Summary

**TDD-built 5/10/15-year projection functions for Hold & Rent (rent compounding + refi logic), Move-Up (purchasing power at future equity), and Equity Play (HELOC growth with interest-only payments)**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-14T02:07:22Z
- **Completed:** 2026-03-14T02:09:44Z
- **Tasks:** 1 feature (TDD: RED + GREEN)
- **Files modified:** 4

## Accomplishments
- Three pure projection functions that power all Phase 7 scenario cards
- Refi logic in Hold & Rent: when marketRate drops below homeowner's locked rate, mortgage recalculated on remaining balance
- All functions use DEFAULTS.appreciationRate (4%) for forward projections, not historical CAGR
- 17 new tests covering compound math, refi scenarios, cash purchases, and edge cases
- Full test suite: 196 tests across 14 files, zero regressions

## Task Commits

Each task was committed atomically:

1. **RED: Failing projection tests** - `445cc29` (test)
2. **GREEN: Implement projection functions** - `fd3c97d` (feat)

_TDD plan: RED wrote 17 failing tests, GREEN implemented all three functions + interfaces + defaults._

## Files Created/Modified
- `src/lib/types.ts` - Added HorizonProjection, HoldAndRentProjection, MoveUpProjection, EquityPlayProjection interfaces
- `src/lib/defaults.ts` - Added DEFAULTS.rentGrowthRate = 0.03
- `src/lib/calculations.ts` - Added calcHoldAndRentProjections, calcMoveUpProjections, calcEquityPlayProjections, PROJECTION_HORIZONS
- `src/lib/calculations.test.ts` - 17 new tests for projection functions

## Decisions Made
- Refi in Hold & Rent recalculates remaining balance at marketRate for a new 30-year term (not remaining years on original loan)
- Projection functions accept ownershipMonths parameter to compute cumulative amortization correctly (ownershipMonths + horizon*12)
- Move-Up clamps negative net proceeds: purchasingPower = 0 when netProceeds <= 0

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- All three projection functions exported and ready for UI consumption in Plan 02 (scenario cards)
- Type interfaces ready for component props
- PROJECTION_HORIZONS constant available for consistent horizon rendering

---
*Phase: 07-future-scenarios-rework*
*Completed: 2026-03-14*
