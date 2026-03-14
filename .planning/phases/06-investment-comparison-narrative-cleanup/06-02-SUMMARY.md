---
phase: 06-investment-comparison-narrative-cleanup
plan: 02
subsystem: ui
tags: [react, typescript, framer-motion, sb7-narrative, leverage-roi, investment-comparison]

# Dependency graph
requires:
  - phase: 06-investment-comparison-narrative-cleanup
    provides: "calcLeverageROI, calcRateLockSavings, currentMarketRate prop, LeverageROI/RateLockSavings types"
provides:
  - "InvestmentComparison component merging S&P 500 and rent-vs-own into leverage-first narrative"
  - "investmentComparisonCopy with 3 tenure-adapted voice variants"
  - "Scenario cards without cliff questions, multi-expand capability"
  - "Updated HomeownerPage section order with InvestmentComparison at cascadeDelay=1000"
affects: [06-03, homeowner-page-layout]

# Tech tracking
tech-stack:
  added: []
  patterns: [conditional section rendering based on ownership tenure and appreciation, leverage hero number as screenshot-worthy centerpiece]

key-files:
  created:
    - src/components/homeowner/InvestmentComparison.tsx
  modified:
    - src/components/homeowner/HomeownerPage.tsx
    - src/components/homeowner/ScenarioCard.tsx
    - src/components/homeowner/ScenarioCards.tsx
    - src/lib/narrative.ts
    - src/app/admin/[id]/preview/page.tsx
    - src/components/homeowner/CurrentValue.tsx
    - src/components/homeowner/EarningsReframe.tsx
    - src/components/homeowner/EquityGained.tsx
    - src/components/homeowner/PurchaseStory.tsx
    - src/components/homeowner/RentVsOwn.tsx
    - src/components/homeowner/ScenarioCards.test.tsx

key-decisions:
  - "S&P one-liner conditional on home outperforming S&P (hidden when S&P wins)"
  - "Rent contrast conditional on sub-24-month ownership"
  - "Rate lock callout conditional on negative appreciation AND rate gap >= 0.5%"
  - "Kept SP500Callout.tsx and RentVsOwn.tsx as dead code for reference"

patterns-established:
  - "Conditional sub-section rendering: guard booleans at top of component, render blocks conditionally"
  - "Leverage hero number pattern: text-5xl+ CountUpNumber with pull-quote callout beneath"
  - "SB7 voice-adapted narrative copy via investmentComparisonCopy Record<NarrativeVoice, SectionCopy>"

requirements-completed: [INVS-01, INVS-02, INVS-03, INVS-04, SCEN-11]

# Metrics
duration: 5min
completed: 2026-03-14
---

# Phase 6 Plan 2: Investment Comparison Component + Narrative Cleanup Summary

**Merged S&P 500 and Rent-vs-Own into a single leverage-first InvestmentComparison section with conditional sub-sections, tenure-adapted SB7 narrative copy, and cliff-question-free scenario cards**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-14T00:38:00Z
- **Completed:** 2026-03-14T01:30:52Z
- **Tasks:** 3 (2 auto + 1 checkpoint)
- **Files modified:** 12

## Accomplishments
- New InvestmentComparison component with leverage hero number as bold centerpiece, personalized callout showing exact dollar amounts, and three conditional sub-sections (S&P one-liner, rent contrast, rate lock callout)
- HomeownerPage section order updated: CurrentValue -> Equity -> InvestmentComparison -> Earnings -> Chart -> NetProceeds -> Scenarios -> CTA
- Scenario cards cleaned up: cliff questions removed, full details shown on expand, multiple cards can be open simultaneously
- Three SB7 voice variants (recent/midterm/longterm) for investment comparison narrative copy

## Task Commits

Each task was committed atomically:

1. **Task 1: InvestmentComparison component + narrative copy** - `ad55a38` (feat) - New component with leverage hero, conditional S&P/rent/rate-lock; investmentComparisonCopy added to narrative.ts
2. **Task 2: Page wiring, section reorder, scenario card cleanup** - `fcc5c96` (feat) - InvestmentComparison wired into HomeownerPage, SP500Callout/RentVsOwn removed from render, cliff questions removed from ScenarioCard
3. **Task 3: Visual verification checkpoint** - Approved by user, no code changes

### Auto-fix commits:
- **Fix missing currentMarketRate in admin preview** - `b3bb716` (fix) - Admin preview page was missing the currentMarketRate prop added in Plan 01
- **Polish narrative copy and UI refinements** - `4945cb6` (refactor) - Tightened SB7 copy across 6 components, UI polish pass

## Files Created/Modified
- `src/components/homeowner/InvestmentComparison.tsx` - NEW: Merged investment comparison section with leverage hero, conditional S&P, rent contrast, rate lock callout
- `src/components/homeowner/HomeownerPage.tsx` - Updated section order, replaced SP500Callout/RentVsOwn with InvestmentComparison, computes leverageROI and rateLockSavings
- `src/components/homeowner/ScenarioCard.tsx` - Removed cliffQuestion prop, added chevron expand indicator
- `src/components/homeowner/ScenarioCards.tsx` - Removed cliffQuestion values from card instances
- `src/lib/narrative.ts` - Added investmentComparisonCopy with 3 voice variants
- `src/app/admin/[id]/preview/page.tsx` - Added missing currentMarketRate prop
- `src/components/homeowner/CurrentValue.tsx` - Narrative copy polish
- `src/components/homeowner/EarningsReframe.tsx` - Narrative copy polish
- `src/components/homeowner/EquityGained.tsx` - Minor copy refinement
- `src/components/homeowner/PurchaseStory.tsx` - Minor copy refinement
- `src/components/homeowner/RentVsOwn.tsx` - Minor copy refinement (retained as dead code)
- `src/components/homeowner/ScenarioCards.test.tsx` - Updated tests for removed cliffQuestion

## Decisions Made
- S&P one-liner is conditional: only appears when home outperformed S&P 500 -- avoids showing unfavorable comparison that undermines the hero narrative
- Rent contrast only for sub-24-month owners -- recent buyers need validation that buying was the right move; longer-term owners already know
- Rate lock callout triple-gated: negative appreciation AND rate gap >= 0.5% AND rate lock data available -- silver lining narrative only when truly relevant
- Kept SP500Callout.tsx and RentVsOwn.tsx files as dead code rather than deleting -- allows reference during future maintenance, zero runtime cost since not imported

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed missing currentMarketRate in admin preview**
- **Found during:** Task 2 (page wiring)
- **Issue:** Admin preview page at `/admin/[id]/preview` was not passing currentMarketRate to HomeownerPage after Plan 01 added the prop
- **Fix:** Added currentMarketRate fetch and prop to admin preview server component
- **Files modified:** src/app/admin/[id]/preview/page.tsx
- **Committed in:** `b3bb716`

**2. [Rule 2 - Polish] Narrative copy and UI refinements**
- **Found during:** Post-Task 2 review
- **Issue:** SB7 narrative copy across multiple components needed tightening for voice consistency and impact
- **Fix:** Polished copy in 6 component files for stronger SB7 tone
- **Files modified:** src/lib/narrative.ts, 6 component files
- **Committed in:** `4945cb6`

---

**Total deviations:** 2 auto-fixed (1 bug, 1 polish)
**Impact on plan:** Bug fix was essential for admin preview functionality. Polish pass improved narrative quality. No scope creep.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Investment Comparison section is live and rendering correctly in all ownership tenure scenarios
- Phase 6 core deliverables complete (Plans 01 + 02)
- Ready for Plan 03 if additional narrative cleanup or polish is planned
- All tests pass, TypeScript compiles cleanly

## Self-Check: PASSED

- All 4 task commits verified in project repo (ad55a38, fcc5c96, b3bb716, 4945cb6)
- SUMMARY.md created at expected path
- All 5 key files confirmed present in project

---
*Phase: 06-investment-comparison-narrative-cleanup*
*Completed: 2026-03-14*
