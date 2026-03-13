# Research Summary: Homeowner Journey Map v1.1 Enhanced Intelligence

**Domain:** Real estate homeowner engagement platform -- interactive features milestone
**Researched:** 2026-03-13
**Overall confidence:** MEDIUM-HIGH

## Executive Summary

The v1.1 milestone transforms the Homeowner Journey Map from a static, admin-generated narrative into an interactive experience where homeowners can select their home condition, explore future scenarios at different time horizons, and see real Cromford market appreciation data. The central research question was: what new technology does this require?

The answer is surprisingly little. The existing stack (Next.js 16, React 19, Zustand 5, Recharts 3, Motion 12) already provides the primitives for every v1.1 feature. The calculation engine is a pure function -- cascading recalculation is simply re-running it with different inputs when the homeowner changes a selection. Sliders and condition pickers are UI components, not library problems. Chart rendering of Cromford data uses the existing Recharts.

The one genuinely new domain is Cromford data acquisition. Research reveals that automated Tableau scraping is impractical in this stack: the only JS scraping library (`scrape-tableau`) is abandoned, Puppeteer on Vercel has severe constraints (50MB limit, 10-second timeout), and the Python scraper does not fit. The recommended approach is manual admin entry of city-level appreciation data into a new Supabase table, which aligns with the project's existing "manual data input" philosophy and serves all homeowners in a city from one data entry.

The optional library additions are @radix-ui/react-slider (only if the design requires a dual-thumb slider rather than preset buttons) and tesseract.js v7 (only if OCR screenshot fallback is deemed necessary). Neither is required for the core feature set.

## Key Findings

**Stack:** No new required dependencies. The existing stack handles all v1.1 features. Two optional additions: @radix-ui/react-slider and tesseract.js.
**Architecture:** New homeowner-facing state (separate from admin store) + pure-function calculation extensions + new `cromford_data` Supabase table.
**Critical pitfall:** Do NOT attempt automated Tableau scraping. It is fragile, poorly supported in JS, and the serverless environment makes it impractical. Manual data entry is the correct approach at this volume.

## Implications for Roadmap

Based on research, suggested phase structure for v1.1:

1. **Schema & Data Foundation** - Add value range columns to homeowners table, create cromford_data table, seed with initial Phoenix data
   - Addresses: Value range storage, Cromford data storage
   - Avoids: Building UI before data layer exists

2. **Calculation Engine Extensions** - Add projection functions, value-range-aware input, combo scenario, rate lock advantage
   - Addresses: 5/10/15yr projections, market rate slider math, 4th combo scenario
   - Avoids: UI work before the math is proven (test-driven)

3. **Interactive Value Selector** - Condition picker UI, homeowner-facing state, cascading recalc wiring
   - Addresses: Value range selector, cascading recalculation, condition picker
   - Avoids: Animation polish before functionality works

4. **Cromford Chart & Merged Investment Section** - Recharts visualization of Cromford data, merged S&P + Rent vs Own section
   - Addresses: Cromford appreciation chart, Investment Comparison merge
   - Avoids: Depends on cromford_data being populated (Phase 1)

5. **Future Scenarios Rework** - 5/10/15yr projection UI, market rate slider, rate lock callout, combo scenario UI
   - Addresses: All future scenario features
   - Avoids: Depends on calculation engine extensions (Phase 2)

6. **UX Polish** - Cascading ripple animations, information cliff removal, scroll flow tuning
   - Addresses: Animation UX, information density
   - Avoids: Polishing UI that may change during Phases 3-5

**Phase ordering rationale:**
- Schema first because all other phases depend on the data structure
- Calculations second because they can be test-driven without UI, and UI phases depend on them
- Interactive selector third because it is the core v1.1 interaction and informs how other sections render
- Cromford and scenarios can run in parallel once schema + calc engine are done
- UX polish last because animating things that might change is wasted work

**Research flags for phases:**
- Phase 1: Standard Supabase migration work, no research needed
- Phase 2: Standard TypeScript math, existing patterns in calculations.ts, no research needed
- Phase 3: May need research on dual-thumb slider UX if design calls for it (Radix slider vs native HTML)
- Phase 4: May need research on Cromford data structure if the actual report format differs from expectations
- Phase 5: No research needed, straightforward UI + existing calculation functions
- Phase 6: No research needed, Motion animation patterns are well-documented

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack additions | HIGH | Verified all libraries against npm, official docs, and existing project |
| Cromford data strategy | MEDIUM | Tableau Public URL confirmed but may change; manual entry approach is stable |
| Calculation extensions | HIGH | Follows established patterns in existing calculations.ts |
| Supabase schema | HIGH | Standard relational modeling, no novel patterns |
| Cascading recalc | HIGH | Same pattern as STR Analyzer (Zustand + useMemo) |
| OCR fallback | MEDIUM | tesseract.js works for text but unreliable for chart line interpolation |

## Gaps to Address

- Exact Cromford data points to seed: need Josh to specify which cities and date ranges matter
- Dual-thumb slider vs preset buttons: design decision affects whether @radix-ui/react-slider is needed
- Tableau embed feasibility: need to test whether Cromford's Tableau Public viz allows embedding (some vizzes block it)
- Market rate slider range: need Josh to specify reasonable min/max appreciation rates for Phoenix market
- Combo scenario logic: "Keep, rent, and buy another" needs requirements refinement -- does the new purchase use HELOC proceeds or net equity from the original home?
