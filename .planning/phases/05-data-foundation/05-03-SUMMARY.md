---
phase: 05-data-foundation
plan: 03
subsystem: ui, animation, charts
tags: [condition-picker, counter-roll, motion-animate, recharts, cromford, cascading-animation]

# Dependency graph
requires:
  - phase: 05-data-foundation/01
    provides: "Condition type, ConditionVariants, conditionValues, HomeownerPage with useState condition"
  - phase: 05-data-foundation/02
    provides: "generateShapedCurve, applyCurveConditionFactor, getConditionFactor from cromford.ts"
provides:
  - "ConditionPicker component with three card-style condition options"
  - "useCounterRoll hook for odometer-style numeric value transitions"
  - "CountUpNumber with cascadeDelay support for staggered counter-roll"
  - "AppreciationChart with Cromford data support and condition-aware curve shifting"
  - "Cascading animation wiring across all data sections (0-2500ms delays)"
affects: [05-04, 05-05, homeowner-page-visual-verification]

# Tech tracking
tech-stack:
  added: []
  patterns: [counter-roll-animation, cascading-delay-props, condition-picker-inline, cromford-chart-integration]

key-files:
  created:
    - src/components/homeowner/ConditionPicker.tsx
    - src/lib/hooks/useCounterRoll.ts
  modified:
    - src/components/homeowner/CountUpNumber.tsx
    - src/components/homeowner/CurrentValue.tsx
    - src/components/homeowner/HomeownerPage.tsx
    - src/components/homeowner/AppreciationChart.tsx
    - src/components/homeowner/EquityGained.tsx
    - src/components/homeowner/EarningsReframe.tsx
    - src/components/homeowner/NetProceeds.tsx
    - src/components/homeowner/SP500Callout.tsx
    - src/components/homeowner/RentVsOwn.tsx

key-decisions:
  - "useCounterRoll uses Motion animate() imperative API with easeInOut for odometer feel"
  - "ConditionPicker hides values on buttons (selecting reveals value in CountUpNumber)"
  - "Cascade timing: 0/500/1000/1500/2000/2500ms across 6 sections for ~3s total cascade"
  - "AppreciationChart uses key={condition} for smooth remount morph animation"
  - "Shaped fallback curve from generateShapedCurve replaces old linear generateChartData"

patterns-established:
  - "cascadeDelay prop pattern: pass delay in ms to data section components for staggered animation"
  - "Counter-roll after count-up: useCountUp for initial scroll reveal, useCounterRoll for subsequent changes"
  - "Condition picker hides when all three condition values are identical (backward compat)"

requirements-completed: [VALU-04, CROM-03, CROM-04]

# Metrics
duration: 4min
completed: 2026-03-13
---

# Phase 5 Plan 03: Condition Picker and Cascading Animation Summary

**Inline condition picker with cascading counter-roll animation and Cromford-powered appreciation chart with condition-aware curve shifting**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-13T15:14:57Z
- **Completed:** 2026-03-13T15:19:00Z
- **Tasks:** 2 of 3 (Task 3 is checkpoint:human-verify)
- **Files modified:** 11

## Accomplishments
- Created ConditionPicker with three branded card-style buttons (original/updated/remodeled) that stacks vertically on mobile
- Built useCounterRoll hook using Motion animate() for smooth odometer-style numeric transitions between condition values
- Wired cascading delays (0-2500ms) across all data sections for theatrical 3-second animation cascade
- Updated AppreciationChart to use real Cromford data when available or shaped fallback curve with seasonal variation
- Chart morphs smoothly on condition change via key={condition} remount with Recharts animation

## Task Commits

Each task was committed atomically:

1. **Task 1: Condition picker, counter-roll hook, and cascading animation wiring** - `1538428` (feat)
2. **Task 2: Appreciation chart with Cromford data and condition-aware curve shifting** - `ed61aa8` (feat)
3. **Task 3: Visual verification** - checkpoint:human-verify (pending)

## Files Created/Modified
- `src/components/homeowner/ConditionPicker.tsx` - Three card-style condition selector with olive/canyon branded styling
- `src/lib/hooks/useCounterRoll.ts` - Hook using Motion animate() for smooth numeric transitions with delay support
- `src/components/homeowner/CountUpNumber.tsx` - Added cascadeDelay prop and useCounterRoll for post-initial-animation changes
- `src/components/homeowner/CurrentValue.tsx` - Added inline ConditionPicker, condition/setCondition props, conditional rendering
- `src/components/homeowner/HomeownerPage.tsx` - Wired cascading delays, passes condition state to CurrentValue and chart
- `src/components/homeowner/AppreciationChart.tsx` - Cromford data support, shaped fallback, condition-aware curve shifting
- `src/components/homeowner/EquityGained.tsx` - Added cascadeDelay prop passed to CountUpNumber
- `src/components/homeowner/EarningsReframe.tsx` - Added cascadeDelay prop (accepted but not yet wired to CountUpNumber)
- `src/components/homeowner/NetProceeds.tsx` - Added cascadeDelay prop passed to CountUpNumber
- `src/components/homeowner/SP500Callout.tsx` - Added cascadeDelay prop (accepted for interface consistency)
- `src/components/homeowner/RentVsOwn.tsx` - Added cascadeDelay prop (accepted for interface consistency)

## Decisions Made
- useCounterRoll skips animation on first render (returns target directly) to avoid conflicting with useCountUp's initial scroll reveal
- ConditionPicker is hidden when all three condition values are identical (backward compatibility with pages that don't have value ranges set)
- AppreciationChart converts Cromford pricePsf data to home values by multiplying by homeSqft and condition factor
- Shaped fallback curve replaces old linear generateChartData -- provides seasonal variation and market cycle noise for more realistic appearance
- EarningsReframe, SP500Callout, and RentVsOwn accept cascadeDelay prop for interface consistency even though they use inline formatting rather than CountUpNumber

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- Pre-existing TypeScript errors in test files (calculations.test.ts, ScenarioCards.test.tsx) -- these are from earlier plans and not related to this plan's changes

## Next Phase Readiness
- Condition picker and cascading animation are functional and ready for visual verification (Task 3 checkpoint)
- AppreciationChart integrates with Cromford pipeline from Plan 02
- All data sections accept cascadeDelay for future animation tuning
- Task 3 (human-verify checkpoint) awaiting Josh's visual review

---
*Phase: 05-data-foundation*
*Completed: 2026-03-13 (Tasks 1-2; Task 3 pending verification)*
