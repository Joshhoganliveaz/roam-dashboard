---
phase: 11-hero-logo-dead-code-cleanup
plan: 01
subsystem: ui
tags: [hero-section, logo, dead-code, cleanup]

requires:
  - phase: 07.1-hero-section-redesign
    provides: "Frosted glass hero layout with text logo placeholder"
provides:
  - "PNG logo rendering in HeroSection on both breakpoints"
  - "331 lines of dead code removed (4 component files)"
affects: []

tech-stack:
  added: []
  patterns: ["eslint-disable for plain img over next/image for brand assets"]

key-files:
  created: []
  modified:
    - src/components/homeowner/HeroSection.tsx
    - src/components/homeowner/HeroSection.test.tsx
  deleted:
    - src/components/homeowner/SP500Callout.tsx
    - src/components/homeowner/RentVsOwn.tsx
    - src/components/homeowner/ComboScenario.tsx
    - src/components/homeowner/NetProceeds.tsx

key-decisions:
  - "PNG logo uses plain img with eslint-disable (not next/image) per project convention"

patterns-established: []

requirements-completed: [HERO-06]

duration: 1min
completed: 2026-03-15
---

# Phase 11 Plan 01: Hero Logo Fix & Dead Code Cleanup Summary

**PNG logo-white.png replaces text "LIVE AZ CO" in HeroSection on both breakpoints, 4 dead component files deleted (331 lines)**

## Performance

- **Duration:** 1 min
- **Started:** 2026-03-15T01:48:07Z
- **Completed:** 2026-03-15T01:49:19Z
- **Tasks:** 2
- **Files modified:** 6 (2 modified, 4 deleted)

## Accomplishments
- HeroSection now renders /logo-white.png as img on desktop (28px) and mobile (24px) with opacity and drop-shadow matching original text styling
- Updated test from text assertion to img element assertion (HERO-06)
- Deleted SP500Callout.tsx, RentVsOwn.tsx, ComboScenario.tsx, NetProceeds.tsx -- all superseded by Phase 6 and Phase 9 components
- All 245 tests pass with zero regressions

## Task Commits

Each task was committed atomically:

1. **Task 1: Replace text logo with PNG in HeroSection + update test** - `77c8330` (feat)
2. **Task 2: Delete 4 dead code component files** - `d5fbe9e` (chore)

## Files Created/Modified
- `src/components/homeowner/HeroSection.tsx` - Desktop and mobile text logos replaced with img elements
- `src/components/homeowner/HeroSection.test.tsx` - Logo test updated from getAllByText to getAllByAltText
- `src/components/homeowner/SP500Callout.tsx` - Deleted (52 lines, replaced by InvestmentComparison)
- `src/components/homeowner/RentVsOwn.tsx` - Deleted (85 lines, replaced by InvestmentComparison)
- `src/components/homeowner/ComboScenario.tsx` - Deleted (82 lines, replaced by scenario cards)
- `src/components/homeowner/NetProceeds.tsx` - Deleted (112 lines, replaced by SellAndMoveUpCard)

## Decisions Made
None - followed plan as specified

## Deviations from Plan
None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- HERO-06 requirement closed
- v1.1 milestone cleanup complete -- all requirements satisfied
- Project ready for final milestone audit

## Self-Check: PASSED

- HeroSection.tsx: FOUND
- HeroSection.test.tsx: FOUND
- SP500Callout.tsx: CONFIRMED DELETED
- RentVsOwn.tsx: CONFIRMED DELETED
- ComboScenario.tsx: CONFIRMED DELETED
- NetProceeds.tsx: CONFIRMED DELETED
- Commit 77c8330: FOUND
- Commit d5fbe9e: FOUND
- Test suite: 245/245 PASS

---
*Phase: 11-hero-logo-dead-code-cleanup*
*Completed: 2026-03-15*
