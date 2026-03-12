---
phase: 03-narrative-experience
plan: 02
subsystem: ui
tags: [scenario-cards, cta, expand-collapse, sb7, sms, motion, animate-presence]

# Dependency graph
requires:
  - phase: 03-narrative-experience/plan-01
    provides: AnimatedSection, childVariants, CountUpNumber, getNarrativeVoice, motion library
  - phase: 02-core-homeowner-page
    provides: HomeownerPage orchestrator, SectionWrapper, HomeownerMetrics type with scenario results
provides:
  - ScenarioCards section with three expandable future scenario cards (Hold & Rent, Move Up, Equity Play)
  - ScenarioCard reusable expandable card component with AnimatePresence
  - CTASection branded closing section with team photos and sms: text link
  - Complete 11-section narrative arc from hero to CTA
affects: [phase-04 if any, deployment]

# Tech tracking
tech-stack:
  added: []
  patterns: [AnimatePresence expand/collapse, tap-to-expand card interaction, sms: protocol link, placeholder team photos]

key-files:
  created:
    - src/components/homeowner/ScenarioCard.tsx
    - src/components/homeowner/ScenarioCards.tsx
    - src/components/homeowner/ScenarioCards.test.tsx
    - src/components/homeowner/CTASection.tsx
    - src/components/homeowner/CTASection.test.tsx
    - public/team/josh.jpg
    - public/team/jacqui.jpg
  modified:
    - src/components/homeowner/HomeownerPage.tsx

key-decisions:
  - "ScenarioCard uses useState toggle + AnimatePresence for smooth expand/collapse"
  - "Cards present pre-computed HomeownerMetrics data only -- no new calculations"
  - "Each expanded card ends at a deliberate information cliff (question requiring Josh)"
  - "CTASection uses sms: protocol link to (480) 369-8280 for single warm CTA action"
  - "Team placeholder photos as minimal JPGs -- Josh will replace with real photos"
  - "Voice-adaptive section headers for scenario cards match 3 ownership duration tiers"

patterns-established:
  - "Expandable card pattern: button wrapper, useState toggle, AnimatePresence height animation"
  - "Information cliff pattern: italic question at end of expanded details"
  - "CTA pattern: branded background, team photos, warm narrative sign-off, single action"

requirements-completed: [SCEN-01, SCEN-02, SCEN-03, NARR-08]

# Metrics
duration: 8min
completed: 2026-03-12
---

# Phase 3 Plan 2: Scenario Cards & CTA Section Summary

**Three tap-to-expand future scenario cards (Hold & Rent, Move Up, Equity Play) with information cliffs, plus branded CTA closing section with team photos and sms: text link completing the 11-section narrative arc**

## Performance

- **Duration:** 8 min
- **Started:** 2026-03-12T21:30:00Z
- **Completed:** 2026-03-12T21:45:00Z
- **Tasks:** 3
- **Files modified:** 8

## Accomplishments
- Built three expandable scenario cards mapping pre-computed HomeownerMetrics data with voice-adaptive headers and stagger animation
- Each card collapses to hero metric + implication, expands to show 2-3 detail rows and an italic cliff question that requires Josh to answer
- Created branded CTA closing section with circular team photos, warm SB7 narrative sign-off, and sms: text link to (480) 369-8280
- Wired ScenarioCards and CTASection into HomeownerPage as sections 10 and 11, completing the full narrative arc
- All tests pass (scenario cards render three cards with correct titles/metrics, CTA renders sms link), build succeeds
- User visually verified complete narrative experience on desktop and mobile -- approved

## Task Commits

Each task was committed atomically:

1. **Task 1: Build ScenarioCards, ScenarioCard, and CTASection components** - `23e41af` (feat) -- TDD: tests written first, then implementations
2. **Task 2: Wire ScenarioCards and CTASection into HomeownerPage** - `8c09bfd` (feat)
3. **Task 3: Visual verification of complete narrative experience** - No commit (human-verify checkpoint, approved)

## Files Created/Modified
- `src/components/homeowner/ScenarioCard.tsx` - Individual expandable card with AnimatePresence, hero metric, details, cliff question
- `src/components/homeowner/ScenarioCards.tsx` - Card cluster section mapping HomeownerMetrics to 3 cards with voice-adaptive header
- `src/components/homeowner/ScenarioCards.test.tsx` - Renders three cards, correct titles, hero metrics from mock data
- `src/components/homeowner/CTASection.tsx` - Branded closing with team photos, warm copy, sms: button
- `src/components/homeowner/CTASection.test.tsx` - Verifies sms: link and team name text
- `src/components/homeowner/HomeownerPage.tsx` - Added ScenarioCards after NetProceeds, CTASection as final element
- `public/team/josh.jpg` - Placeholder team photo (to be replaced)
- `public/team/jacqui.jpg` - Placeholder team photo (to be replaced)

## Decisions Made
- ScenarioCard uses useState toggle with AnimatePresence for expand/collapse -- simple, no external state needed
- Cards present pre-computed data only (no new calculations) per plan requirement
- Each card ends at a deliberate information cliff -- a question only Josh can answer, driving engagement
- CTASection uses sms: protocol for single-action simplicity -- "Text us" is warmer than "Contact us"
- Voice-adaptive headers match getNarrativeVoice() tiers from Plan 01 for narrative consistency
- Placeholder team photos are minimal JPGs -- Josh will replace with real photos before launch

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required. Josh should replace placeholder team photos in `public/team/` with real photos before production use.

## Next Phase Readiness
- Complete 11-section narrative arc is built and verified: Hero -> PersonalNote -> PurchaseStory -> CurrentValue -> EquityGained -> EarningsReframe -> AppreciationChart -> RentVsOwn -> NetProceeds -> ScenarioCards -> CTASection
- Phase 3 (Narrative Experience) is fully complete -- all scroll animations, countUp numbers, adaptive voice, milestones, scenario cards, and CTA are wired and working
- Ready for Phase 4 or deployment

## Self-Check: PASSED

All 8 created/modified files verified present. Both task commits (23e41af, 8c09bfd) verified in git log.

---
*Phase: 03-narrative-experience*
*Completed: 2026-03-12*
