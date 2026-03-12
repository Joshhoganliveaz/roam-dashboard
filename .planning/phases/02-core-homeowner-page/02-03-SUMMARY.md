---
phase: 02-core-homeowner-page
plan: 03
subsystem: ui
tags: [recharts, appreciation-chart, sp500-comparison, data-visualization]

# Dependency graph
requires:
  - phase: 02-core-homeowner-page
    plan: 02
    provides: "Six data sections wired in narrative order"
provides:
  - "AppreciationChart with olive-to-gold gradient, SP500Callout card, complete 9-section page"
affects: [03-narrative-experience]

# Tech tracking
tech-stack:
  added: [recharts]
  patterns: ["Recharts AreaChart with SVG gradient fill and ReferenceDot annotations"]

key-files:
  created:
    - src/components/homeowner/AppreciationChart.tsx
    - src/components/homeowner/SP500Callout.tsx
  modified:
    - src/components/homeowner/HomeownerPage.tsx
    - package.json

key-decisions:
  - "Chart uses area chart (not line) for visual weight and gradient fill"
  - "S&P shown as callout card below chart (not second line on chart) for clarity"
  - "Gradient uses olive top to gold bottom matching brand palette"
  - "Biweekly data points for short ownership (3-12 months) for smoother curves"

patterns-established:
  - "Recharts AreaChart pattern: ResponsiveContainer > AreaChart > defs gradient > Area with fill url reference"
  - "Graceful degradation: simplified highlight card for insufficient data (under 3 months ownership)"

requirements-completed: [DATA-03, DATA-06]

# Metrics
duration: 5min
completed: 2026-03-12
---

# Phase 2 Plan 03: Appreciation Chart and S&P Callout Summary

**Recharts area chart with olive-to-gold gradient, annotated purchase/today markers, S&P 500 callout card, and graceful sub-3-month fallback**

## Performance

- **Duration:** 5 min (estimated from single atomic commit)
- **Started:** 2026-03-12T13:50:00Z
- **Completed:** 2026-03-12T13:55:00Z
- **Tasks:** 1 (code task; visual verification was checkpoint)
- **Files modified:** 5

## Accomplishments
- AppreciationChart (218 lines) renders Recharts AreaChart with olive-to-gold SVG linear gradient fill
- Annotated ReferenceDot markers at purchase point and today with formatted price labels
- Biweekly data point generation for 3-12 month ownership produces smoother curves
- Graceful fallback for under-3-month ownership: standalone highlight card instead of empty chart
- SP500Callout (38 lines) renders as standalone comparison card below the chart
- HomeownerPage now renders complete 9-section narrative (hero through net proceeds with chart in position 7)
- Recharts installed and wired into the project

## Task Commits

Each task was committed atomically:

1. **Task 1: Install Recharts and build appreciation chart + S&P callout** - `e0eee9b` (feat)

**Note:** Task 2 was a checkpoint:human-verify (visual verification by user). This was completed during the original execution.

## Files Created/Modified
- `src/components/homeowner/AppreciationChart.tsx` - Recharts AreaChart with gradient fill, annotated markers, biweekly/monthly data generation
- `src/components/homeowner/SP500Callout.tsx` - S&P 500 comparison callout card with honest outperform/underperform display
- `src/components/homeowner/HomeownerPage.tsx` - Updated to wire AppreciationChart and SP500Callout into section 7
- `package.json` - Added recharts dependency
- `package-lock.json` - Lock file updated for recharts

## Decisions Made
- Chart uses area chart (not line chart) for greater visual impact with gradient fill
- S&P 500 comparison shown as a separate callout card below the chart rather than as a second line on the chart -- cleaner separation and easier to read on mobile
- Gradient uses olive (#5C6B4F, opacity 0.8) at top fading to gold (#C9953E, opacity 0.1) at bottom, matching brand palette
- Biweekly data points for short ownership (3-12 months) instead of monthly, producing smoother curves
- Under 3 months ownership shows a highlight card instead of a near-empty chart -- graceful degradation

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Complete 9-section homeowner page ready for Phase 3 narrative experience layer
- All DATA requirements (01-07) now satisfied in code
- Chart section provides the visual centerpiece for scroll animation in Phase 3

---
*Phase: 02-core-homeowner-page*
*Completed: 2026-03-12*
