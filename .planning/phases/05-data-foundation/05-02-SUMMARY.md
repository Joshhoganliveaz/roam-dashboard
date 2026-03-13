---
phase: 05-data-foundation
plan: 02
subsystem: api, ui
tags: [claude-vision, ocr, cromford, admin-form, data-processing]

# Dependency graph
requires:
  - phase: 05-data-foundation/01
    provides: "Condition types, value range schema, AdminFormState with valueLow/valueHigh/homeSqft"
provides:
  - "extractCromfordData function for Claude Vision OCR of Cromford screenshots"
  - "validateCromfordData, generateShapedCurve, applyCurveConditionFactor, getConditionFactor utilities"
  - "POST /api/cromford/extract endpoint for screenshot processing"
  - "CromfordPreview component for admin approval workflow"
  - "Cromford upload section in HomeownerForm with extract/preview/approve flow"
  - "cromfordData field in AdminFormState and save payload"
affects: [05-data-foundation/03, appreciation-chart, homeowner-page]

# Tech tracking
tech-stack:
  added: []
  patterns: [raw-fetch-claude-vision, cromford-ocr-pipeline, admin-preview-approve]

key-files:
  created:
    - src/lib/cromford.ts
    - src/lib/cromford.test.ts
    - src/lib/claude-api.ts
    - src/app/api/cromford/extract/route.ts
    - src/components/admin/CromfordPreview.tsx
  modified:
    - src/lib/types.ts
    - src/lib/store.ts
    - src/components/admin/HomeownerForm.tsx

key-decisions:
  - "Used raw fetch to Claude Vision API (no SDK), matching dashboard-generator pattern"
  - "Cromford extraction is fully optional -- OCR failure does not block page creation"
  - "Deterministic shaped curve seeded from purchasePrice for consistent fallback charts"

patterns-established:
  - "Cromford OCR pipeline: upload -> Claude Vision -> preview -> approve -> store"
  - "generateShapedCurve with seasonal + market cycle noise for realistic fallback curves"
  - "applyCurveConditionFactor for proportional curve shifting by condition"

requirements-completed: [CROM-01, CROM-02]

# Metrics
duration: 5min
completed: 2026-03-13
---

# Phase 5 Plan 02: Cromford OCR Pipeline Summary

**Claude Vision OCR extraction pipeline with admin upload, preview/approve workflow, and Cromford data processing utilities (validation, shaped curve fallback, condition-factor scaling)**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-13T15:02:20Z
- **Completed:** 2026-03-13T15:07:25Z
- **Tasks:** 2
- **Files modified:** 8

## Accomplishments
- Cromford data processing utilities with 16 unit tests covering validation, curve generation, condition factor derivation, and scaling
- Claude Vision API wrapper using raw fetch for screenshot-to-time-series extraction
- API route with proper error handling, file type validation, and extraction quality checks
- Admin form Cromford upload section with full extract/preview/approve/reject workflow
- CromfordPreview component showing data point sample table and trend summary for Josh's review

## Task Commits

Each task was committed atomically:

1. **Task 1: Cromford data processing utilities with tests and Claude Vision API wrapper**
   - `37f588d` (test) - RED: failing tests for Cromford utilities
   - `5d59ea9` (feat) - GREEN: implement cromford.ts and claude-api.ts
2. **Task 2: Cromford extraction API route, admin upload with preview/approve workflow** - `c098144` (feat)

## Files Created/Modified
- `src/lib/cromford.ts` - Cromford data processing: validateCromfordData, generateShapedCurve, applyCurveConditionFactor, getConditionFactor
- `src/lib/cromford.test.ts` - 16 unit tests for all Cromford processing utilities
- `src/lib/claude-api.ts` - Claude Vision API wrapper with raw fetch, extraction prompt, JSON parse with fallback
- `src/app/api/cromford/extract/route.ts` - POST endpoint: receives screenshot FormData, returns extracted time-series
- `src/components/admin/CromfordPreview.tsx` - Preview component with data table, trend summary, approve/reject buttons
- `src/lib/types.ts` - Added cromfordData to AdminFormState
- `src/lib/store.ts` - Added cromfordData: null to initial state
- `src/components/admin/HomeownerForm.tsx` - Added Cromford Report section with upload, extraction, preview/approve flow

## Decisions Made
- Used raw fetch to Claude Vision API (no SDK) -- matches existing dashboard-generator pattern, avoids extra dependency
- Shaped curve uses deterministic seed from purchasePrice so the same home always generates the same fallback curve shape
- Cromford extraction is fully optional -- Josh can skip it and create pages without market data
- CromfordPreview shows first 5 and last 5 data points with trend summary for quick eyeball verification

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Removed duplicate cromford_data in save record**
- **Found during:** Task 2 (HomeownerForm integration)
- **Issue:** Linter auto-inserted cromford_data in the record object, causing a duplicate with the line added by plan execution
- **Fix:** Removed the duplicate property
- **Files modified:** src/components/admin/HomeownerForm.tsx
- **Verification:** Tests pass, no duplicate property warnings
- **Committed in:** c098144 (Task 2 commit)

---

**Total deviations:** 1 auto-fixed (1 bug)
**Impact on plan:** Trivial fix for duplicate property. No scope creep.

## Issues Encountered
- Pre-existing TypeScript errors in calculations.test.ts and ScenarioCards.test.tsx (not related to this plan's changes, existed before plan execution)

## User Setup Required

**ANTHROPIC_API_KEY must be set** for Cromford OCR to work:
- Add `ANTHROPIC_API_KEY=sk-ant-...` to `.env.local`
- Get key from https://console.anthropic.com/settings/keys
- OCR works without key -- extraction fails gracefully and Josh can still create pages

## Next Phase Readiness
- Cromford processing utilities ready for Plan 03 chart rendering (generateShapedCurve, applyCurveConditionFactor)
- extractCromfordData ready for production use with valid ANTHROPIC_API_KEY
- Admin form has full Cromford upload workflow integrated

---
*Phase: 05-data-foundation*
*Completed: 2026-03-13*
