---
phase: 07-future-scenarios-rework
plan: 02
subsystem: ui
tags: [react, recharts, cta, mobile-responsive, sb7]

requires:
  - phase: 01-foundation-admin
    provides: AnimatedSection wrapper, team photos in /public/team/
provides:
  - Warm contact CTA section with Josh & Jacqui photos and inline contact links
  - Mobile-friendly Cromford chart date labels with rotation and spacing
affects: [09-cta-per-card]

tech-stack:
  added: []
  patterns:
    - "Inline contact links instead of button CTAs for warm SB7 tone"
    - "Rotated XAxis labels (-45deg) with increased bottom margin for mobile chart readability"

key-files:
  created: []
  modified:
    - src/components/homeowner/CTASection.tsx
    - src/components/homeowner/CTASection.test.tsx
    - src/components/homeowner/AppreciationChart.tsx

key-decisions:
  - "Inline tel: and sms: links instead of a single large button -- warmer, less pushy"
  - "Rotated labels + reduced interval for mobile chart fix (simpler than custom tick renderer)"

patterns-established:
  - "SB7 closing pattern: guide photos + warm heading + inline contact links"

requirements-completed: [PAGE-03, PAGE-05]

duration: 1min
completed: 2026-03-14
---

# Phase 7 Plan 2: Simple CTA & Cromford Mobile Fix Summary

**Warm Josh & Jacqui contact CTA with inline phone/text links, and rotated Cromford chart date labels for mobile readability**

## Performance

- **Duration:** 1 min
- **Started:** 2026-03-14T04:59:08Z
- **Completed:** 2026-03-14T05:00:32Z
- **Tasks:** 2
- **Files modified:** 3

## Accomplishments
- Replaced sales-pitch CTA button with warm contact section showing both phone and text links
- Fixed Cromford chart date label overlapping on mobile with rotation, spacing, and margin adjustments
- Updated CTASection tests to match new heading and contact link structure (5 tests passing)

## Task Commits

Each task was committed atomically:

1. **Task 1: Replace CTA with simple Josh & Jacqui contact section** - `b2ba5c6` (feat)
2. **Task 2: Fix Cromford chart date label spacing on mobile** - `afaf677` (fix)

## Files Created/Modified
- `src/components/homeowner/CTASection.tsx` - Warm closing section with "Let's talk about your options" heading, inline phone/text links
- `src/components/homeowner/CTASection.test.tsx` - Updated tests for new heading text and both contact link types
- `src/components/homeowner/AppreciationChart.tsx` - Rotated XAxis labels, increased bottom margin, reduced tick interval

## Decisions Made
- Used inline `tel:` and `sms:` links instead of a single large button for warmer, less pushy SB7 guide energy
- Combined rotation (-45deg) + reduced interval (4 labels vs 5) + bottom margin (40px) for mobile chart fix rather than a custom tick renderer -- simpler and sufficient

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- CTA section ready for Phase 9 where individual scenario cards will get their own CTAs
- Cromford chart readable on all screen sizes

---
*Phase: 07-future-scenarios-rework*
*Completed: 2026-03-14*
