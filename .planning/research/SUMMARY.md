# Project Research Summary

**Project:** Homeowner Journey Map
**Milestone:** v1.1 Enhanced Intelligence
**Domain:** Interactive financial projections and real market data on personalized homeowner pages
**Researched:** 2026-03-13
**Confidence:** MEDIUM-HIGH

## Executive Summary

The v1.1 milestone transforms the Homeowner Journey Map from a read-only narrative page into a guided interactive experience. The core shift is architectural: v1.0 pages are fully server-rendered with zero client-side state, while v1.1 introduces two focused interaction points -- a condition picker (original/updated/remodeled) and a market rate slider -- that cascade through all downstream financial calculations. This is a deliberate reversal of the v1.0 anti-feature "no interactive calculators," reframed as "guided interactivity" where Josh controls the bounds and the homeowner explores within them. The existing Next.js 16 / React 19 / Supabase / Tailwind / Motion / Recharts stack handles everything with zero new required dependencies.

The recommended approach is to pre-compute metrics for all three condition tiers server-side (avoiding shipping the calculation engine to the client), then let the client switch between pre-computed metric sets on condition change. The market rate slider is the exception -- it feeds a continuous value into scenario projection functions that must run client-side, extracted into a shared `scenario-calculations.ts` module. Cromford appreciation data should be manually entered by Josh into a city-level `cromford_data` Supabase table and rendered with the existing Recharts library. Automated Tableau scraping is explicitly rejected as fragile and unmaintainable -- manual entry at 5 minutes per city per quarter is the right tradeoff.

The primary risks are (1) breaking existing published pages when adding new schema columns (mitigated by nullable columns and backward-compatible `applyDefaults()`), (2) hydration mismatches from the server-to-client architecture shift (mitigated by passing server-computed `now` date as props and deferring client recalculation to first user interaction), and (3) the interactive controls undermining the narrative experience by turning the page into a calculator (mitigated by limiting to 2 visible controls max per viewport, pre-selecting defaults, and ensuring the page tells a complete story with zero interaction). The combo scenario ("keep, rent, and buy another") is the highest-complexity feature and should be built last.

## Key Findings

### Stack Additions (from STACK-v1.1.md)

No new required libraries. The existing validated stack handles all v1.1 features. Calculations extend `calculations.ts` with pure TypeScript functions. UI sliders use native HTML range inputs + Tailwind. Cromford charts render via Recharts from manually-entered Supabase data. Cascading animations use Motion (already installed).

**Optional additions (only if design confirms):**
- **@radix-ui/react-slider ^1.3.2:** Only if dual-thumb range slider UX is chosen over 3 preset buttons. 3.5KB, ARIA-compliant, Tailwind-compatible.
- **tesseract.js ^7.0.0:** Only if OCR fallback for Cromford screenshots is needed. Defer unless there is a strong reason.

**Key schema additions:**
- `value_low` and `value_high` numeric columns on homeowners table (nullable for backward compat)
- New `cromford_data` table: city-level price-per-sqft by year/quarter, shared across all homeowners in a city

**Explicitly rejected:**
- Automated Tableau scraping (fragile, unmaintained libraries, serverless constraints)
- Puppeteer on Vercel (50MB limit, 10-second timeout, memory issues)
- Any new charting, animation, or state management library

### Expected Features (from FEATURES.md / FEATURES-v1.1.md)

**Must have (table stakes for v1.1):**
- Value range selector with condition picker (original/updated/remodeled) -- the core v1.1 interaction
- Cascading recalculation on condition change -- without this, the picker is cosmetic
- Cromford appreciation chart with real market data -- even if manually entered
- Merged Investment Comparison section (replaces separate S&P + Rent vs Own)
- 5/10/15yr projection timelines for all scenarios
- Market rate slider for exploring appreciation scenarios
- Rate lock advantage callout (for sub-4% mortgage holders)
- 4th combo scenario: "Keep, rent, and buy another"

**Should have (differentiators):**
- Cascading ripple animation on condition change (numbers flow down the page)
- Cromford data comparison to homeowner's specific appreciation rate
- Enhanced scenario details (projected cash flow, purchasing power, HELOC cost at rates)

**Defer if timeline is tight:**
- Cascading ripple animation (ship with simple opacity transitions, polish later)
- 4th combo scenario (most complex; three existing scenarios are already strong)
- OCR fallback for Cromford screenshots (manual entry works)
- Tableau embedding (adds 200KB for marginal benefit over Recharts rendering)

**Anti-features (do NOT build):**
- Freeform home value slider (undermines Josh's comp-anchored authority)
- Automated Cromford data refresh (over-engineering for quarterly data)
- Editable mortgage assumptions (turns narrative into calculator)
- Tax calculation integration (STR Analyzer handles that)
- PDF export of interactive page (URL is the deliverable)

### Architecture Approach (from ARCHITECTURE.md / ARCHITECTURE-v1.1.md)

The fundamental shift is from server-computed-only to a hybrid model. The server pre-computes metrics for all 3 conditions and passes them as props. The client holds 3 pieces of state via `useState` (NOT Zustand): `selectedCondition`, `projectionYears`, and `marketRateOverride`. Condition changes swap between pre-computed metric sets (no client-side recalculation). Rate slider changes trigger lightweight scenario projection functions extracted into `scenario-calculations.ts` that run client-side.

**Key architectural decisions:**
1. **Pre-compute 3 condition variants server-side** -- avoids shipping the calculation engine to the client
2. **Use useState, not Zustand** -- homeowner page has exactly 3 state values; Zustand adds unnecessary indirection
3. **Split calculation into two stages** -- `computeHomeownerMetrics()` for current-state (runs on condition change), `computeProjections()` for forward-looking (runs on rate/year change)
4. **Cromford data is city-level, stored in a separate table** -- one data entry serves all homeowners in that city
5. **Cromford data fetched server-side at render time from Supabase, never scraped at render time**

**New files:**
- `src/components/homeowner/ValueRangeSelector.tsx`
- `src/components/homeowner/InvestmentComparison.tsx`
- `src/components/homeowner/MarketRateSlider.tsx`
- `src/lib/scenario-calculations.ts`

**Removed files:**
- `SP500Callout.tsx` and `RentVsOwn.tsx` (absorbed into InvestmentComparison)

### Critical Pitfalls (from PITFALLS.md / PITFALLS-v1.1.md)

1. **Breaking existing published pages with schema migration** -- All new columns must be nullable. `applyDefaults()` must handle absence. Integration test with v1.0-shaped data must produce identical output. This is the very first thing to build and verify.
2. **Hydration mismatch from server/client metric divergence** -- Pass server-computed `now` date as serialized prop. Client uses passive hydration (server-computed metrics) until first user interaction triggers client-side recalculation.
3. **Cascading recalculation turns page into laggy calculator on mobile** -- Debounce the market rate slider (150ms). Memoize child components with `React.memo()`. Split metrics props so shallow comparison works. Test on real phone with 4x CPU throttle. Performance budget: condition change under 100ms.
4. **Condition picker creates decision paralysis** -- Place after the value reveal (not at page top). Default to admin-selected condition. Use homeowner-friendly labels. The page must work with zero interaction.
5. **Combo scenario scope creep** -- Keep it simpler than individual scenarios, not more complex. Compose existing calculation functions. Display as timeline narrative, not data table. Build last.

## Implications for Roadmap

Based on combined research, the dependency chain and risk profile suggest 5 phases.

### Phase 1: Value Range Foundation + Backward Compatibility
**Rationale:** Every other v1.1 feature depends on the condition-aware data flow. The schema migration and backward compatibility layer must be proven before any feature work. This is also the highest-risk area (breaking existing pages) and must be isolated and tested.
**Delivers:** Schema migration (value_low, value_high columns), updated types, updated applyDefaults with null-safe fallbacks, server component computing conditionMetrics for all 3 tiers, ValueRangeSelector component, HomeownerPage gains useState for condition, cascading section updates, admin form with low/high value inputs.
**Addresses:** Value range selector, condition picker, cascading recalculation, cascading animation (basic version).
**Avoids:** Pitfall 1 (schema breaking existing pages), Pitfall 2 (hydration mismatch), Pitfall 4 (recalc performance -- establish the pattern here).
**Research flag:** Standard patterns. No phase research needed -- this is React state management and Supabase column additions.

### Phase 2: Cromford Data Integration
**Rationale:** Independent of interactivity work. High perceived value (real market data = authority). Isolated risk -- if it takes longer, it does not block other phases. The existing synthetic appreciation chart serves as automatic fallback.
**Delivers:** `cromford_data` Supabase table, admin UI for manual data entry (city, year, price-per-sqft), AppreciationChart updated to use real data when available, Cromford vs homeowner appreciation comparison, graceful fallback when no city data exists.
**Addresses:** Cromford appreciation chart, market context in narrative.
**Avoids:** Pitfall 1 from PITFALLS-v1.1.md (automated scraping -- we skip it entirely in favor of manual entry).
**Research flag:** No phase research needed. Manual data entry + Recharts rendering is well-understood.

### Phase 3: Merged Investment Comparison + Rate Lock
**Rationale:** Cleans up page structure before adding projection complexity. Removes two components, adds one. The leverage calculation is simple. Best done before future scenarios rework since it changes section ordering.
**Delivers:** InvestmentComparison component (leverage callout + three-column comparison + net result narrative), rate lock advantage callout, removal of SP500Callout and RentVsOwn components, updated narrative copy for all 3 voice variants.
**Addresses:** Merged Investment Comparison, rate lock advantage callout, information cliff removal for these sections.
**Avoids:** Pitfall 6 (broken narrative arc from merge -- keep calc engine unchanged, merge only at presentation layer).
**Research flag:** No phase research needed. Component refactoring with existing calculation outputs.

### Phase 4: Future Scenarios Rework (Projections + Rate Slider + Combo)
**Rationale:** Heaviest calculation and UX work. Depends on condition picker (Phase 1) being stable since projections build on the selected value. The market rate slider introduces the only client-side recalculation in the app. The combo scenario is the most complex feature and highest conversation-generation potential.
**Delivers:** `scenario-calculations.ts` (extracted, shared server/client), 5/10/15yr projection logic for all scenarios, MarketRateSlider component, updated ScenarioCards with projection tabs, 4th combo scenario card, rate slider bounds (conservative to aggressive), assumption disclosures on all projections.
**Addresses:** 5/10/15yr projections, market rate slider, 4th combo scenario, enhanced Hold & Rent / Move-Up / Equity Play.
**Avoids:** Pitfall 3 (mobile performance -- debounce slider, memoize components), Pitfall 5 (projection inconsistency -- single-pass projection function), Pitfall 7 (combo scope creep -- compose existing functions, keep simpler than individual scenarios).
**Research flag:** Needs phase research. The combo scenario requirements need clarification with Josh (HELOC vs sale proceeds for down payment). Slider debounce and mobile performance need real-device profiling.

### Phase 5: UX Polish + Deployment
**Rationale:** All features are functional. This phase refines animations, handles edge cases, and ensures mobile performance across the board.
**Delivers:** Cascading ripple animation refinement (value-to-value transitions if needed), information cliff removal across all sections, mobile responsive fixes for new components, loading/error states, animation priority system (suppress competing animations during recalc), bulk ISR revalidation script for rollout.
**Addresses:** Cascading ripple animation (polished), slider fatigue mitigation (max 2 controls per viewport), information cliff removal.
**Avoids:** Pitfall 8 (slider fatigue -- holistic UX review), Pitfall 9 (animation conflicts -- priority system).
**Research flag:** No phase research needed. UX polish with established patterns.

### Phase Ordering Rationale

- **Schema and backward compatibility first:** The highest risk in v1.1 is breaking 200+ existing published pages. Isolating the migration and proving backward compatibility before any feature work eliminates this risk early.
- **Cromford independent and early:** It has high perceived value and zero dependency on other v1.1 features. If it runs long, it does not block the critical path.
- **Investment Comparison before scenarios:** The merge cleans up the page structure. Adding projections and a 4th scenario card to a messy section layout would compound complexity.
- **Scenarios are the heaviest phase:** They depend on both the condition picker (Phase 1) and the clean page structure (Phase 3). The market rate slider introduces the only client-side recalculation, which needs careful performance work.
- **Polish last:** The page must work correctly before it works beautifully. Animation refinement and edge case handling are separable from functionality.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 4 (Future Scenarios):** Combo scenario requirements need Josh's input. Client-side projection performance needs real-device profiling. Rate slider UX (bounds, defaults, labels) needs design iteration.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Value Range Foundation):** React useState, Supabase column additions, pre-computed condition switching. Well-understood.
- **Phase 2 (Cromford Data):** Manual data entry + Recharts rendering. No novel patterns.
- **Phase 3 (Investment Comparison):** Component refactoring. Existing calculations, new presentation.
- **Phase 5 (UX Polish):** Motion animations and responsive CSS. Established patterns.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | No new required libraries. All v1.1 features build on the validated v1.0 stack. Optional additions (Radix slider, tesseract) are well-documented if needed. |
| Features | HIGH | Both researchers converge on the same table stakes and anti-features. Clear dependency graph. Phased prioritization with explicit deferral criteria. |
| Architecture | HIGH | Two independent architecture researchers arrived at the same core pattern: pre-compute 3 conditions server-side, useState (not Zustand) for client state, extract scenario-calculations.ts for shared server/client use. Minor disagreement on Cromford storage (JSONB column vs separate table) -- separate table is correct for city-level shared data. |
| Pitfalls | HIGH | 13 pitfalls identified across two researchers with significant overlap on the critical ones (schema migration, hydration, recalc performance). Concrete file-level mappings and phase-specific warnings. |

**Overall confidence:** MEDIUM-HIGH

The MEDIUM-HIGH (rather than HIGH) reflects one genuine uncertainty: the Cromford data acquisition strategy. Research strongly recommends manual admin entry, which is the safest approach, but the Tableau embedding option and OCR fallback both have MEDIUM confidence. If Josh later wants automated data, the path is unclear. For v1.1, manual entry is the right call.

### Gaps to Address

- **Combo scenario requirements:** The "keep, rent, and buy another" scenario needs Josh's input on key assumptions: does the new purchase use HELOC funds or sale proceeds? What vacancy rate and management fee to assume? What qualification model (75% rental income offset)? Clarify before Phase 4 planning.
- **Cromford data coverage by city:** Research assumes Cromford publishes city-level price-per-sqft data for Phoenix metro cities. Need to verify which cities have data available (Scottsdale, Tempe, Mesa, Gilbert, Chandler, etc.) and what the data format looks like. Affects the `cromford_data` schema.
- **Condition value mapping:** Research recommends original = valueLow, updated = 40th percentile, remodeled = valueHigh. The 40th percentile formula needs validation with Josh -- does this match how he anchors comp values?
- **Market rate slider default:** Research suggests initializing from Cromford CAGR when available, falling back to DEFAULTS.appreciationRate (4%). Need to verify what appreciation rate Cromford data actually yields for Phoenix metro to ensure the default feels reasonable.
- **Animation approach for number transitions:** Two approaches documented (key-based re-mount vs value-to-value animation). Phase 1 should start with key-based re-mount and Phase 5 should upgrade if it feels jarring. This is a design decision, not a technical gap.

## Sources

### Primary (HIGH confidence)
- Existing codebase: `/Users/joshuahogan/Projects/homeowner-journey-map/src/` -- direct code review of calculations.ts, types.ts, defaults.ts, transforms.ts, store.ts, HomeownerPage.tsx, page.tsx
- Next.js 16 Server/Client Components: https://nextjs.org/docs/app/getting-started/server-and-client-components
- Motion animation docs: https://motion.dev/docs
- Radix UI Slider: https://www.radix-ui.com/primitives/docs/components/slider
- Supabase migration patterns: https://supabase.com/docs/guides/deployment/database-migrations
- React useMemo: https://react.dev/reference/react/useMemo
- tesseract.js v7.0.0: https://github.com/naptha/tesseract.js/releases

### Secondary (MEDIUM confidence)
- Tableau Embedding API v3: https://help.tableau.com/current/api/embedding_api/en-us/index.html
- Homebot "Tune Your Value": https://help.homebotapp.com/en/articles/3631274-module-overview-tune-your-value
- Cromford Report: https://cromfordreport.com/
- Zillow Zestimate Owner Updates: https://www.zillow.com/premier-agent/make-instant-updates-to-zestimate/
- Cognitive Overload in UX (Smashing Magazine): https://www.smashingmagazine.com/2016/09/reducing-cognitive-overload-for-a-better-user-experience/

### Tertiary (LOW confidence)
- Cromford Tableau Public URL stability (may change without notice)
- tableau-scraping library reliability (documented failures in GitHub issues)

---
*Research completed: 2026-03-13*
*Ready for roadmap: yes*
