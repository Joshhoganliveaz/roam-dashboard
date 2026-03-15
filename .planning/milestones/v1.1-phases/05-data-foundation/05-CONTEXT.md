# Phase 5: Data Foundation - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning

<domain>
## Phase Boundary

Homeowners see a condition-aware home value (original/updated/remodeled) that cascades through all financial numbers, and the appreciation chart shows real Cromford market data instead of a straight-line estimate. Admin form captures value range (low/high comp values) and Cromford Report screenshot. No backward compatibility concern — no homeowner pages exist yet.

</domain>

<decisions>
## Implementation Decisions

### Condition Picker UX
- Claude's discretion on component style (segmented control, cards, etc.)
- Placed inline with the value section ("Today, your home is worth...") — contextual, part of the story
- Each option shows label + one-liner description to help homeowners self-identify (e.g., "Mostly original — Original fixtures, no major renovations")
- Values are NOT shown upfront — selecting a condition reveals the value and cascades the update (matches the page's "reveal" storytelling pattern)
- Default condition: "Made some updates" (middle) — most homes fall here, homeowner adjusts from there

### Value Mapping Within Range
- Original = low comp value
- Made some updates = midpoint of range
- Completely remodeled = high comp value + 3%
- Josh enters two values in admin: low comp value and high comp value
- No backward compatibility handling needed — no existing pages to worry about

### Cromford OCR & Chart Data
- Try full time-series extraction first (monthly/quarterly price-per-sqft data points) for a realistic market curve with dips, surges, and plateaus
- Fall back to shaped curve with single appreciation rate when OCR can't extract clean data points — apply realistic market curve shape (not a straight line)
- Cromford screenshot uploaded during page creation as part of the admin form (single workflow)
- After OCR runs, show Josh an extraction preview with extracted data points — he approves before data goes into the chart
- Chart renders real market data as branded Recharts chart (olive-to-gold gradient, established in Phase 2)
- Condition selector shifts the entire appreciation curve proportionally (price-per-sqft x home sqft x condition factor)

### Cascading Ripple Animation
- Counter roll animation — numbers roll/count from old value to new value (odometer style)
- Theatrical cascade timing: 3-4 seconds total, each section's counter completes before the next starts
- Sequential flow down the page section by section
- All sections animate regardless of viewport position — if homeowner scrolls down, they see the tail end still rolling
- Appreciation chart curve shifts smoothly to new position when condition changes (morph, not redraw)

### Claude's Discretion
- Condition picker component style (segmented control vs cards vs other)
- Exact one-liner descriptions for each condition option
- Counter roll animation easing and per-section timing within the 3-4 second budget
- OCR extraction approach (Claude Vision prompt engineering for Cromford screenshots)
- Shaped curve algorithm for appreciation rate fallback
- Admin form layout for the new value range fields and Cromford upload
- How the OCR preview is displayed to Josh (inline, modal, etc.)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `src/app/page.tsx`: Admin form with existing Cromford file upload pattern (`cromfordFiles` array, `FileDropZone` component) — extend for homeowner page creation
- `src/lib/csv-engine.ts`: `deriveValueFromComps()` already returns `{ derivedValue, derivedRange: { low, high } }` — pattern for value range
- `src/lib/comp-adjustments.ts`: Comp adjustment logic with weighted averaging — informs value mapping approach
- `src/components/homeowner/AppreciationChart.tsx`: Recharts AreaChart component (Phase 2) — extend with Cromford data and condition-aware shifting
- `src/components/homeowner/HomeownerPage.tsx`: Client-side orchestrator with `SectionWrapper` for alternating backgrounds — add animation triggers here
- `src/lib/calculations.ts`: Pure calculation functions with `computeHomeownerMetrics()` orchestrator — extend for condition-variant computation

### Established Patterns
- Pre-compute 3 condition variants server-side, switch client-side via useState (not Zustand) — decided in STATE.md
- Pure calculation functions with deterministic outputs, no side effects
- Server component fetches data → passes to client HomeownerPage orchestrator
- SB7 color coding: canyon for narrative, charcoal for hero numbers, charcoal/70 for supporting copy
- Hero number + narrative intro + supporting copy pattern in every data section
- Animate once only on scroll (Phase 3) — cascading animation is a separate trigger (condition change)
- FileDropZone drag-drop + click pattern for file uploads
- Claude Vision already used for Cromford image analysis in `/api/dashboard/generate/route.ts`

### Integration Points
- Admin form (`src/app/page.tsx`): Add value range fields (low/high), condition default selector, Cromford screenshot upload
- Homeowner page (`src/app/h/[slug]/page.tsx`): Server component pipeline with condition-variant pre-computation
- Supabase schema: Add `value_low`, `value_high` columns, `cromford_data` JSON column for extracted time-series
- API route: New OCR endpoint for Cromford screenshot extraction with preview response
- Recharts: `animationBegin` and `animationDuration` props for chart curve morphing

</code_context>

<specifics>
## Specific Ideas

- The appreciation chart must NOT look like a straight line — real appreciation has dips, surges, and plateaus. Even the fallback curve should have realistic shape
- Counter roll animation should feel like an odometer or slot machine — dramatic, satisfying, builds anticipation as each section cascades
- The 3-4 second theatrical cascade matches the cinematic feel established in Phase 3 — each section gets its moment
- OCR preview is about Josh's confidence — he needs to verify the extracted data points look right before they appear on a homeowner's page
- Condition picker is inline with the value story, not a separate UI element — it's part of the narrative flow

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 05-data-foundation*
*Context gathered: 2026-03-13*
