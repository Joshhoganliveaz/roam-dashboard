---
phase: 07-future-scenarios-rework
plan: 02
subsystem: ui
tags: [react, typescript, slider, projections, scenario-cards, horizon-tabs, useMemo]

# Dependency graph
requires:
  - phase: 07-future-scenarios-rework
    provides: calcHoldAndRentProjections, calcMoveUpProjections, calcEquityPlayProjections, HorizonProjection types
  - phase: 02-core-homeowner-page
    provides: ScenarioCards, ScenarioCard, HomeownerPage base components
provides:
  - MarketRateSlider component (range input with rate lock indicator)
  - HorizonTabs component (5yr/10yr/15yr pill selector)
  - Enhanced ScenarioCard supporting both static and horizon-tabbed content
  - ScenarioCards orchestrator with slider state driving all projections
  - scenarioCardsCopy voice-adapted narrative text
  - Sell-to-buy education section in Move-Up card (4 paths)
  - Rate lock advantage and refi note callouts in Hold & Rent card
affects: [07-03 (combo scenario)]

# Tech tracking
tech-stack:
  added: []
  patterns: [useMemo-driven projection recalculation on slider change, horizon-keyed content maps, children prop for custom expanded content]

key-files:
  created:
    - src/components/homeowner/MarketRateSlider.tsx
    - src/components/homeowner/HorizonTabs.tsx
  modified:
    - src/components/homeowner/ScenarioCard.tsx
    - src/components/homeowner/ScenarioCards.tsx
    - src/components/homeowner/ScenarioCards.test.tsx
    - src/components/homeowner/HomeownerPage.tsx
    - src/lib/narrative.ts

key-decisions:
  - "ScenarioCards owns slider state via useState, not lifted to HomeownerPage"
  - "Horizon content maps built via useMemo keyed on projection arrays"
  - "ScenarioCard uses children prop for custom expanded content (rate lock callout, sell-to-buy education)"
  - "HorizonTabs buttons use stopPropagation to prevent card collapse on tab click"

patterns-established:
  - "Horizon content pattern: Record<5|10|15, HorizonContent> for multi-horizon card data"
  - "Slider-driven recalculation: useState + useMemo for the single client-side calculation point"

requirements-completed: [SCEN-04, SCEN-05, SCEN-06, SCEN-08, SCEN-09, SCEN-10]

# Metrics
duration: 4min
completed: 2026-03-14
---

# Phase 7 Plan 2: Interactive Scenario Cards with Market Rate Slider and Multi-Horizon Projections Summary

**Market rate slider driving three scenario cards with 5/10/15yr horizon tabs, rate lock advantage callout, sell-to-buy education, and HELOC cost at slider rate**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-14T02:11:32Z
- **Completed:** 2026-03-14T02:15:24Z
- **Tasks:** 2
- **Files modified:** 7

## Accomplishments
- MarketRateSlider with 3-10% range in 0.25% steps, rate lock advantage indicator
- HorizonTabs with olive/charcoal pill buttons for 5/10/15yr selection
- ScenarioCard refactored for dual mode: static (backward compat) and horizon-tabbed content
- ScenarioCards orchestrates slider state with useMemo-driven projections for all three scenarios
- Hold & Rent: rate lock callout when homeowner rate beats market, refi note when market drops below
- Move-Up: purchasing power at each horizon with sell-to-buy education (4 paths)
- Equity Play: HELOC available with interest-only payment updating at slider rate
- 199 tests passing across 14 test files, zero regressions

## Task Commits

Each task was committed atomically:

1. **Task 1: MarketRateSlider, HorizonTabs, and enhanced ScenarioCard** - `70334e8` (feat)
2. **Task 2: ScenarioCards orchestrator with slider, projections, and enhanced cards** - `dc4d3aa` (feat)

## Files Created/Modified
- `src/components/homeowner/MarketRateSlider.tsx` - Range slider with rate display and rate lock indicator
- `src/components/homeowner/HorizonTabs.tsx` - 5yr/10yr/15yr tab selector component
- `src/components/homeowner/ScenarioCard.tsx` - Enhanced card supporting horizon tabs, children prop, backward compat
- `src/components/homeowner/ScenarioCards.tsx` - Refactored orchestrator with slider state, projections, enhanced cards
- `src/components/homeowner/ScenarioCards.test.tsx` - Updated tests for new props (9 tests)
- `src/components/homeowner/HomeownerPage.tsx` - Passes record and currentMarketRate to ScenarioCards
- `src/lib/narrative.ts` - Added scenarioCardsCopy with voice-adapted intro text

## Decisions Made
- ScenarioCards owns slider state via useState -- not lifted to HomeownerPage because only scenario cards need it
- Horizon content maps built via useMemo keyed on projection arrays for efficient recalculation
- ScenarioCard uses children prop for custom expanded content (rate lock callout, sell-to-buy education) rather than adding more config props
- HorizonTabs buttons use stopPropagation to prevent card collapse when switching horizons

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Updated existing ScenarioCards tests for new props**
- **Found during:** Task 2 (ScenarioCards refactor)
- **Issue:** Existing tests passed only metrics and voice; new component requires record and currentMarketRate
- **Fix:** Added mockRecord fixture, updated all test renders to pass new required props, updated assertions to match new UI (horizon tabs instead of static details)
- **Files modified:** src/components/homeowner/ScenarioCards.test.tsx
- **Verification:** All 199 tests pass
- **Committed in:** dc4d3aa (Task 2 commit)

---

**Total deviations:** 1 auto-fixed (1 bug fix)
**Impact on plan:** Test update was necessary consequence of component API change. No scope creep.

## Issues Encountered
- DOM nesting warning: button-in-button (HorizonTabs buttons inside ScenarioCard button element). Does not affect functionality. Candidate for future cleanup (change outer button to div with role="button").

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- All three scenario cards fully wired to projection functions with slider-driven recalculation
- Ready for Plan 03 (combo scenario) which composes these projection functions
- HorizonContent pattern established for reuse in combo scenario card

---
*Phase: 07-future-scenarios-rework*
*Completed: 2026-03-14*
