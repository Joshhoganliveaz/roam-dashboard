---
phase: 07-future-scenarios-rework
plan: 01
subsystem: ui
tags: [hero, paycheck-hook, page-layout, section-reorder, sbr-narrative]

# Dependency graph
requires:
  - phase: 02-core-homeowner-page
    provides: HeroSection, HomeownerPage layout, metrics calculations
provides:
  - Weekly paycheck hook as standalone PaycheckHook component below hero
  - Reordered page sections (scenarios before data/proof)
  - Removed NetProceeds standalone section and ConditionCompLinks external link
affects: [09-scenario-cards-rework]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "PaycheckHook as standalone section with solid background (not overlaid on hero photo)"

key-files:
  created:
    - src/components/homeowner/PaycheckHook.tsx
  modified:
    - src/components/homeowner/HeroSection.tsx
    - src/components/homeowner/HomeownerPage.tsx

key-decisions:
  - "PaycheckHook as separate component below hero with charcoal background for readability (user design feedback)"
  - "HeroSection kept clean: photo + name + address + branding only"
  - "Cream text on charcoal background for paycheck hook (brand-consistent, high contrast)"

patterns-established:
  - "Hero photo sections stay clean -- no number overlays on images"
  - "Emotional hook numbers get their own solid-background section for readability"

requirements-completed: [PAGE-01, PAGE-02, PAGE-04, PAGE-06]

# Metrics
duration: 1min
completed: 2026-03-14
---

# Phase 7 Plan 1: Hero Paycheck Hook & Page Reorder Summary

**Weekly paycheck hook moved to standalone charcoal-background section below hero photo, page sections reordered with scenarios before data/proof sections**

## Performance

- **Duration:** 1 min
- **Started:** 2026-03-14T14:07:43Z
- **Completed:** 2026-03-14T14:08:52Z
- **Tasks:** 2 (1 auto + 1 checkpoint with design revision)
- **Files modified:** 3

## Accomplishments
- Weekly paycheck hook ("Your home earned you $X/week") displayed as first compelling number below hero photo on solid background
- Page sections reordered: hero -> paycheck hook -> note -> value+equity -> scenarios -> chart -> comparison -> earnings detail -> purchase story -> CTA
- NetProceeds standalone section removed (file preserved for Phase 9 reuse)
- ConditionCompLinks external link removed (no off-page links)
- All 239 existing tests pass

## Task Commits

Each task was committed atomically:

1. **Task 1: Add weekly paycheck to HeroSection and reorder page sections** - `7b5083d` (feat) -- initial implementation with overlay
2. **Task 2: Move paycheck hook below hero per user feedback** - `460fa86` (feat) -- design revision: standalone PaycheckHook component

**Plan metadata:** pending (docs: complete plan)

## Files Created/Modified
- `src/components/homeowner/PaycheckHook.tsx` - New standalone paycheck hook section with charcoal background, positive/negative earnings display
- `src/components/homeowner/HeroSection.tsx` - Cleaned up: removed earningsPerWeek/isNegative props and overlay, kept photo + name + address + branding
- `src/components/homeowner/HomeownerPage.tsx` - Added PaycheckHook import/render, removed earningsPerWeek/isNegative props from HeroSection

## Decisions Made
- **PaycheckHook as separate component:** User feedback indicated the paycheck number overlaid on the hero photo was hard to read. Moved to its own section with a solid charcoal background for maximum contrast and readability.
- **Cream on charcoal color scheme:** Uses project branding colors (cream text on charcoal background) for the paycheck hook section, providing high contrast without clashing with the hero photo.
- **HeroSection kept minimal:** Hero shows only the home photo with name, address, and "Live AZ Co" branding -- no numbers, keeping it visually clean.

## Deviations from Plan

### Design Revision (User Feedback at Checkpoint)

**1. Paycheck hook moved from hero overlay to standalone section**
- **Found during:** Task 2 checkpoint review
- **Issue:** User reported the weekly paycheck hook overlaid on the hero photo was hard to read
- **Fix:** Created PaycheckHook component with solid charcoal background, removed earnings overlay from HeroSection
- **Files created:** src/components/homeowner/PaycheckHook.tsx
- **Files modified:** src/components/homeowner/HeroSection.tsx, src/components/homeowner/HomeownerPage.tsx
- **Committed in:** 460fa86

---

**Total deviations:** 1 design revision (user-directed)
**Impact on plan:** Improved readability per user feedback. Same functional outcome (paycheck hook is first number after hero), better visual execution.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Page layout finalized with scenarios in prominent position
- PaycheckHook pattern established for future reference
- Ready for remaining Phase 7 plans and Phase 9 scenario card work

## Self-Check: PASSED

All files exist, all commits verified, SUMMARY.md created.

---
*Phase: 07-future-scenarios-rework*
*Completed: 2026-03-14*
