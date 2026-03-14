---
phase: 08-condition-picker-comp-context
plan: 01
subsystem: calculations
tags: [condition-values, clamping, comp-data, soldPrice, backward-compat]

# Dependency graph
requires:
  - phase: 05-data-foundation
    provides: comp-analyzer with RepresentativeComp and ValueRangeResult types
provides:
  - Clamped getConditionValues function (+/-3-5% from base instead of raw comp range)
  - ConditionCompLink type enriched with optional soldPrice
  - conditionCompLinks data plumbed from server component through HomeownerPage to CurrentValue
affects: [08-condition-picker-comp-context plan 02, scenario-cards]

# Tech tracking
tech-stack:
  added: []
  patterns: [asymmetric clamping (-3%/+5%) for condition value mapping, optional field enrichment for backward compat]

key-files:
  created: []
  modified:
    - src/lib/types.ts
    - src/lib/calculations.ts
    - src/lib/calculations.test.ts
    - src/lib/comp-analyzer.test.ts
    - src/components/admin/CSVUpload.tsx
    - src/app/h/[slug]/page.tsx
    - src/components/homeowner/HomeownerPage.tsx
    - src/components/homeowner/CurrentValue.tsx
    - src/app/admin/[id]/preview/page.tsx

key-decisions:
  - "Asymmetric clamping: -3% (original) / base (updated) / +5% (remodeled) reflects SB7 insight that homeowners think homes worth more"
  - "soldPrice optional on ConditionCompLink for backward compat with 200+ existing records"
  - "baseValue param defaults to midpoint of low/high for backward compat with two-arg callers"

patterns-established:
  - "Clamped condition values: always use currentValue as base, never map directly to raw comp range"
  - "Enrichment pattern: add optional fields to existing interfaces rather than breaking changes"

requirements-completed: [COMP-02, COMP-03]

# Metrics
duration: 5min
completed: 2026-03-14
---

# Phase 8 Plan 01: Condition Picker Data Layer Summary

**Clamped getConditionValues to +/-3-5% from base with asymmetric SB7-informed percentages, enriched ConditionCompLink with soldPrice, and plumbed comp data through server component to CurrentValue**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-14T05:56:16Z
- **Completed:** 2026-03-14T06:01:27Z
- **Tasks:** 2
- **Files modified:** 9

## Accomplishments
- getConditionValues now clamps to -3%/+5% from base value instead of mapping directly to raw comp range (which could be 15-20% wide)
- ConditionCompLink type enriched with optional soldPrice for inline comp display in Plan 02
- CSVUpload stores soldPrice from RepresentativeComp alongside address and zillowUrl
- conditionCompLinks data flows end-to-end from server component through HomeownerPage to CurrentValue
- Full backward compatibility maintained: two-arg form uses midpoint, optional soldPrice preserves 200+ existing records

## Task Commits

Each task was committed atomically:

1. **Task 1 (TDD RED): Failing tests for clamped values** - `cdd4562` (test)
2. **Task 1 (TDD GREEN): Implement clamped getConditionValues and soldPrice type** - `85f21e3` (feat)
3. **Task 2: Enrich comp data storage and plumb to CurrentValue** - `6fb0ae2` (feat)

## Files Created/Modified
- `src/lib/types.ts` - Added optional soldPrice to ConditionCompLink interface
- `src/lib/calculations.ts` - Rewrote getConditionValues with clamped +/-3-5% formula and optional baseValue param
- `src/lib/calculations.test.ts` - Updated getConditionValues tests for clamped behavior, added backward compat and asymmetry tests
- `src/lib/comp-analyzer.test.ts` - Added soldPrice > 0 assertion on representative comps (COMP-02 traceability)
- `src/components/admin/CSVUpload.tsx` - handleApproveComps now stores soldPrice in conditionCompLinks
- `src/app/h/[slug]/page.tsx` - Passes currentValue as third arg to getConditionValues, forwards conditionCompLinks to HomeownerPage
- `src/components/homeowner/HomeownerPage.tsx` - Accepts and forwards conditionCompLinks prop to CurrentValue
- `src/components/homeowner/CurrentValue.tsx` - Accepts conditionCompLinks prop (UI wiring deferred to Plan 02)
- `src/app/admin/[id]/preview/page.tsx` - Updated with same getConditionValues and conditionCompLinks changes for consistency

## Decisions Made
- Asymmetric clamping (-3% down, +5% up) reflects Josh's SB7 insight that homeowners always think their home is worth more -- the upside feels slightly more generous
- soldPrice is optional on ConditionCompLink to maintain backward compatibility with 200+ existing records that only have address + zillowUrl
- baseValue parameter defaults to midpoint of low/high when not provided, preserving backward compat for any two-arg callers

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Fixed admin preview page missing conditionCompLinks prop**
- **Found during:** Task 2 (TypeScript compilation verification)
- **Issue:** `src/app/admin/[id]/preview/page.tsx` also renders HomeownerPage but was not in the plan's file list. Adding conditionCompLinks as required prop broke this file.
- **Fix:** Updated admin preview to pass `currentValue` as base to getConditionValues and forward `conditionCompLinks` to HomeownerPage
- **Files modified:** src/app/admin/[id]/preview/page.tsx
- **Verification:** TypeScript compiles with zero non-test errors
- **Committed in:** 6fb0ae2 (part of Task 2 commit)

---

**Total deviations:** 1 auto-fixed (1 blocking)
**Impact on plan:** Necessary fix for TypeScript compilation. Admin preview must match public page prop signature. No scope creep.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- conditionCompLinks data is now available in CurrentValue component, ready for Plan 02 to wire the inline comp display UI
- All calculation tests pass with the new clamped formula
- Backward compatibility verified: existing pages with null conditionCompLinks or old format continue to work

## Self-Check: PASSED

All 9 modified files exist. All 3 commits (cdd4562, 85f21e3, 6fb0ae2) verified in git log.

---
*Phase: 08-condition-picker-comp-context*
*Completed: 2026-03-14*
