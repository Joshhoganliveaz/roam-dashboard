---
phase: 03-narrative-experience
plan: 01
subsystem: ui
tags: [motion, animation, scroll-trigger, narrative-voice, milestones, countup, sb7]

# Dependency graph
requires:
  - phase: 02-core-homeowner-page
    provides: Section components (CurrentValue, EquityGained, EarningsReframe, etc.), SectionWrapper, HomeownerPage orchestrator
  - phase: 02.1-financial-data-audit-edge-cases
    provides: Hardened formatCurrency/formatPercent, negative value handling, SB7 underwater narratives
provides:
  - AnimatedSection wrapper with stagger/unit modes for scroll-triggered reveals
  - CountUpNumber component with ease-in rAF animation
  - MilestoneCallout inline badge component
  - getNarrativeVoice() with 3 ownership duration tiers and SB7 copy maps for 6 sections
  - selectMilestones() with anniversary, equity threshold, and appreciation candidates
  - useCountUp hook with requestAnimationFrame and easeInQuad easing
  - All existing sections wired with scroll animations, adaptive narrative, and milestone display
affects: [03-02 scenario cards and CTA]

# Tech tracking
tech-stack:
  added: [motion ^12.x, @testing-library/react (dev), @testing-library/dom (dev)]
  patterns: [whileInView scroll triggers, staggered child variants, useInView for chart draw, voice-keyed copy maps]

key-files:
  created:
    - src/lib/narrative.ts
    - src/lib/narrative.test.ts
    - src/lib/milestones.ts
    - src/lib/milestones.test.ts
    - src/lib/hooks/useCountUp.ts
    - src/lib/hooks/useCountUp.test.ts
    - src/components/homeowner/AnimatedSection.tsx
    - src/components/homeowner/CountUpNumber.tsx
    - src/components/homeowner/MilestoneCallout.tsx
  modified:
    - src/components/homeowner/HomeownerPage.tsx
    - src/components/homeowner/CurrentValue.tsx
    - src/components/homeowner/EquityGained.tsx
    - src/components/homeowner/EarningsReframe.tsx
    - src/components/homeowner/NetProceeds.tsx
    - src/components/homeowner/PurchaseStory.tsx
    - src/components/homeowner/RentVsOwn.tsx
    - src/components/homeowner/AppreciationChart.tsx
    - package.json

key-decisions:
  - "motion library (v12.x) for all scroll animations via whileInView + viewport={{ once: true }}"
  - "Custom useCountUp hook with easeInQuad instead of react-countup dependency"
  - "Voice thresholds: <24mo recent, 24-83mo midterm, 84+mo longterm"
  - "Milestone impressiveness scoring: equity threshold/50K, anniversary years, appreciation*10"
  - "Stagger pattern for key data sections (intro->number->copy), unit animation for quieter sections"
  - "TypeScript ease typing fixed with as const for motion variant compatibility"

patterns-established:
  - "AnimatedSection stagger pattern: wrap in AnimatedSection stagger, children use motion.div variants={childVariants}"
  - "CountUpNumber pattern: value + format prop, uses useInView internally"
  - "Voice-keyed copy maps: Record<NarrativeVoice, { intro, supporting }> per section"
  - "Milestone section matching: selectMilestones returns section field, HomeownerPage finds by section name"

requirements-completed: [NARR-03, NARR-04, NARR-05]

# Metrics
duration: 9min
completed: 2026-03-12
---

# Phase 3 Plan 1: Animation Infrastructure & Adaptive Narrative Summary

**Motion-powered scroll animations with staggered reveals, ease-in countUp numbers, voice-adaptive SB7 copy for 3 ownership durations, and inline milestone badges wired into all homeowner page sections**

## Performance

- **Duration:** 9 min
- **Started:** 2026-03-12T20:46:43Z
- **Completed:** 2026-03-12T20:56:00Z
- **Tasks:** 2
- **Files modified:** 20

## Accomplishments
- Installed motion library and created full animation infrastructure (AnimatedSection, CountUpNumber, useCountUp hook) with scroll-triggered reveals that play exactly once
- Built adaptive narrative voice system with 3 ownership duration tiers and SB7 copy maps for all 6 data sections
- Implemented milestone selection logic scoring anniversary, equity thresholds, and appreciation percentage, capped at 3 per page
- Wired all existing section components with staggered animations for key data sections, unit animations for quieter sections, and chart draw-in on scroll
- 16 new tests covering voice boundaries, milestone selection, and countUp behavior -- all 133 tests pass, build succeeds

## Task Commits

Each task was committed atomically:

1. **Task 1: Create animation utilities, narrative voice, and milestone logic** - `6235601` (feat) -- TDD: tests written first, then implementations
2. **Task 2: Wire animations, adaptive narrative, and milestones into existing sections** - `a598d77` (feat)

## Files Created/Modified
- `src/lib/narrative.ts` - getNarrativeVoice() + SB7 copy maps for 6 sections x 3 voices
- `src/lib/narrative.test.ts` - Boundary tests for voice selection at 0, 23, 24, 83, 84 months
- `src/lib/milestones.ts` - selectMilestones() with impressiveness scoring and top-3 cap
- `src/lib/milestones.test.ts` - Empty results, single milestone, max 3, sorting, appreciation threshold tests
- `src/lib/hooks/useCountUp.ts` - rAF counter with easeInQuad easing and hasAnimated guard
- `src/lib/hooks/useCountUp.test.ts` - Returns 0 when not in view, returns target on completion, no replay
- `src/components/homeowner/AnimatedSection.tsx` - Motion wrapper with stagger/unit modes and exported childVariants
- `src/components/homeowner/CountUpNumber.tsx` - Animated number display with format options
- `src/components/homeowner/MilestoneCallout.tsx` - Gold pill badge for inline milestone display
- `src/components/homeowner/HomeownerPage.tsx` - Computes voice + milestones, passes to all sections
- `src/components/homeowner/CurrentValue.tsx` - Staggered animation + voice-adaptive copy + milestone
- `src/components/homeowner/EquityGained.tsx` - Staggered animation + voice-adaptive copy + milestone
- `src/components/homeowner/EarningsReframe.tsx` - Staggered animation + voice-adaptive copy
- `src/components/homeowner/NetProceeds.tsx` - Staggered animation + voice-adaptive copy
- `src/components/homeowner/PurchaseStory.tsx` - Voice-adaptive copy + milestone (unit animation via parent)
- `src/components/homeowner/RentVsOwn.tsx` - Voice-adaptive copy (unit animation via parent)
- `src/components/homeowner/AppreciationChart.tsx` - useInView controlling isAnimationActive for chart draw
- `package.json` - Added motion dependency

## Decisions Made
- Used motion (v12.x) for all animations -- de facto React animation library, whileInView maps directly to requirements
- Custom useCountUp hook (~40 lines) instead of react-countup package -- avoids dependency for simple feature
- Voice thresholds at 24 and 84 months match "under 2 years" and "7+ years" from context document
- Milestone impressiveness scoring uses equity/50K for equity thresholds, raw years for anniversary, appreciation*10 for percentage
- Key data sections (CurrentValue, EquityGained, EarningsReframe, NetProceeds) get staggered element entrances; quieter sections (PurchaseStory, RentVsOwn, PersonalNote) animate as a unit
- HeroSection stays unwrapped -- above the fold, should render immediately
- Fixed TypeScript ease typing with `as const` for motion variant compatibility

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Installed @testing-library/react for hook tests**
- **Found during:** Task 1 (RED phase -- useCountUp test)
- **Issue:** @testing-library/react not installed, needed for renderHook in useCountUp tests
- **Fix:** `npm install -D @testing-library/react @testing-library/dom`
- **Files modified:** package.json, package-lock.json
- **Verification:** Tests import and run successfully
- **Committed in:** 6235601 (Task 1 commit)

**2. [Rule 1 - Bug] Fixed TypeScript ease type incompatibility**
- **Found during:** Task 2 (build verification)
- **Issue:** `ease: "easeOut"` in childVariants inferred as `string` which is not assignable to motion's `Easing` type
- **Fix:** Changed to `ease: "easeOut" as const` for narrow type inference
- **Files modified:** src/components/homeowner/AnimatedSection.tsx
- **Verification:** Build passes with no TypeScript errors
- **Committed in:** a598d77 (Task 2 commit)

---

**Total deviations:** 2 auto-fixed (1 blocking dependency, 1 type bug)
**Impact on plan:** Both auto-fixes necessary for correctness. No scope creep.

## Issues Encountered
- Milestone sorting test initially failed due to tied impressiveness values (3-year anniversary = 3, $150K equity / $50K = 3). Adjusted test to use values where equity clearly outranks anniversary.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Animation infrastructure complete and tested -- AnimatedSection, CountUpNumber, MilestoneCallout all ready for reuse
- Narrative voice system provides copy maps for all sections -- Plan 03-02 can use same pattern for scenario card copy
- All sections animated and wired -- Plan 03-02 adds ScenarioCards and CTASection as new sections
- motion library installed and working -- Plan 03-02 can use AnimatePresence for expand/collapse on scenario cards

## Self-Check: PASSED

All 9 created files verified present. Both task commits (6235601, a598d77) verified in git log.

---
*Phase: 03-narrative-experience*
*Completed: 2026-03-12*
