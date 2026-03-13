# Domain Pitfalls: v1.1 Enhanced Intelligence

**Domain:** Adding interactivity to server-rendered homeowner pages
**Researched:** 2026-03-13

## Critical Pitfalls

Mistakes that cause rewrites or major issues.

### Pitfall 1: Attempting Automated Tableau Scraping

**What goes wrong:** Building a Puppeteer-based or library-based scraper for Cromford Tableau charts, then discovering it breaks constantly in production.
**Why it happens:** The idea of "automated data" is appealing. Developers assume Tableau Public charts are easily scrapable.
**Consequences:** The scraper breaks when Cromford updates their Tableau viz, changes the URL, or Tableau updates their rendering. Puppeteer on Vercel hits the 50MB function size limit or 10-second timeout. The `scrape-tableau` npm package is abandoned (last update 4 years ago). Time is spent debugging scraper reliability instead of building features.
**Prevention:** Use manual admin data entry into a `cromford_data` Supabase table. At 20-30 pages/month and quarterly Cromford updates, this is a 5-minute task per city per quarter. The data is stable, cacheable, and never breaks.
**Detection:** If you find yourself writing retry logic for a Tableau scraper, stop immediately.

### Pitfall 2: Hydration Mismatch from Server/Client Metric Divergence

**What goes wrong:** Server-rendered page shows one set of numbers, then client-side React hydration shows different numbers, causing a visible flash/jump.
**Why it happens:** The server computes metrics with a default condition (e.g., "updated"), then the client re-initializes state and recomputes. If the default condition or any input differs between server and client, numbers flash.
**Consequences:** Users see numbers change on page load. Looks buggy. Breaks trust in a financial context.
**Prevention:** Ensure the server passes the EXACT same initial state that the client will use. The server component should compute metrics with `selectedCondition = 'updated'` (the default) and pass both the metrics AND the selected condition as props. The client component initializes `useState('updated')` from props.
**Detection:** Test by watching the page load in slow 3G mode. If numbers visually change after initial render, you have a hydration mismatch.

### Pitfall 3: Storing Derived State in the Database

**What goes wrong:** Adding columns like `equity_gained`, `annualized_appreciation`, `purchasing_power` to the homeowners Supabase table.
**Why it happens:** Developer thinks "I should persist the calculated values so I do not recalculate every page load."
**Consequences:** When the calculation engine is updated (e.g., new formula for net proceeds), stored values become stale. You need a migration to recompute all stored metrics. If a user changes their condition, the stored values are wrong. Two sources of truth for the same data.
**Prevention:** Store ONLY inputs (purchase price, current value, value range, loan details). Derive ALL metrics at render time. The calculation takes < 1ms -- there is no performance benefit to caching it.
**Detection:** If a column in the homeowners table can be computed from other columns, it should not exist.

## Moderate Pitfalls

### Pitfall 4: Making the Entire Page a Client Component

**What goes wrong:** Wrapping the whole homeowner page in `"use client"` to support the interactive condition picker.
**Why it happens:** The condition picker needs client state, and it is easier to make everything a client component than to design the server/client boundary.
**Prevention:** Only the interactive wrapper component needs `"use client"`. The page-level server component fetches data and passes it down. The client component handles state and re-rendering. Static content (narrative text, SB7 framing, photos) can remain in server components or be passed as children.

### Pitfall 5: Animating Raw Numbers Instead of Formatted Strings

**What goes wrong:** Using Motion to animate a number from 450000 to 520000, which causes the number to visually count through every intermediate value.
**Why it happens:** Developer uses `motion.span` with `animate={{ value }}` and Motion interpolates the number.
**Prevention:** Do NOT animate the number value. Instead, animate the CONTAINER (opacity, position) and swap the formatted string. Use `key={formattedValue}` to trigger a mount/unmount animation, not a value interpolation.

### Pitfall 6: Value Range Without Guard Rails

**What goes wrong:** Admin enters value_original > value_remodeled, or enters 0 for one condition, or enters wildly different values that make scenarios nonsensical.
**Why it happens:** No validation on the value range inputs in the admin form.
**Prevention:** Validate that `value_original <= value_updated <= value_remodeled`. Warn (but allow) if the spread exceeds 30% of the median. Require all three values or none (do not allow partial entry). If no range is provided, fall back to the single `current_value` field.

### Pitfall 7: Projection Functions Using Different Appreciation Rates Than Cromford Data

**What goes wrong:** The 5/10/15yr projections use the default 4% appreciation rate from DEFAULTS.ts, while the Cromford chart shows actual historical rates that may be 6-8% for Phoenix. The homeowner sees conflicting stories on the same page.
**Why it happens:** Two data sources (DEFAULTS vs cromford_data) are not reconciled.
**Prevention:** When Cromford data exists for the homeowner's city, compute the actual historical CAGR from cromford_data and use it as the default for the market rate slider. The slider should initialize at the Cromford-derived rate, not the hardcoded 4%.

## Minor Pitfalls

### Pitfall 8: Condition Picker Labels Confusing Homeowners

**What goes wrong:** Homeowner does not understand what "original condition" vs "updated" vs "remodeled" means for their specific home.
**Prevention:** Add brief descriptions: "Original: As-purchased, minimal updates" / "Updated: New paint, fixtures, landscaping" / "Remodeled: Kitchen/bath renovation, major upgrades". Consider using dollar ranges instead of or alongside condition labels.

### Pitfall 9: Market Rate Slider Allowing Negative Appreciation

**What goes wrong:** Slider allows 0% or negative rates, which breaks projection calculations (division by zero, negative future values).
**Prevention:** Set slider minimum to 1% or 0.5%. If the homeowner wants to model depreciation, that is a separate (and demoralizing) feature to consider carefully.

### Pitfall 10: Cromford Data Gaps for Non-Phoenix Cities

**What goes wrong:** Josh creates a homeowner page for Scottsdale or Mesa, but the cromford_data table only has Phoenix data.
**Prevention:** Handle gracefully: if no cromford_data exists for the homeowner's city, hide the Cromford chart section rather than showing an empty chart or erroring. Show a fallback message or omit the section entirely.

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Schema migration | Adding columns without backfilling existing rows | Migration should set value_original = value_updated = value_remodeled = current_value for existing rows |
| Calculation engine | Breaking existing tests when adding new functions | New functions are additive; do not modify existing function signatures |
| Condition picker | Hydration mismatch on initial render | Server computes with default condition; client initializes from same default |
| Cromford chart | No data for homeowner's city | Graceful fallback: hide section, do not show empty chart |
| Projection slider | Conflicting appreciation rates between Cromford and projections | Initialize slider from Cromford CAGR when available |
| Cascading animation | Performance on low-end mobile devices | Profile on throttled CPU; keep animations to opacity/transform only (GPU-accelerated) |
| Combo scenario | Requirements ambiguity on "keep, rent, and buy another" | Clarify with Josh before building: does new purchase use HELOC or sale proceeds? |

## Sources

- Vercel function limits: https://github.com/vercel/community/discussions/103 (HIGH confidence)
- Next.js hydration docs: https://nextjs.org/docs/messages/react-hydration-error (HIGH confidence)
- Motion performance: https://motion.dev/docs/performance (HIGH confidence)
- scrape-tableau npm status: https://www.npmjs.com/package/scrape-tableau (HIGH confidence)
- Existing calculations.ts: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/calculations.ts (HIGH confidence)
- Existing defaults.ts: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/defaults.ts (HIGH confidence)
