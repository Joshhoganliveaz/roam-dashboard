# Stack Additions for v1.1 Enhanced Intelligence

**Project:** Homeowner Journey Map
**Milestone:** v1.1 Enhanced Intelligence
**Researched:** 2026-03-13
**Confidence:** MEDIUM-HIGH (Cromford/Tableau approach needs validation; all other areas HIGH)

## Scope

This document covers ONLY new libraries and schema changes needed for v1.1. The existing validated stack (Next.js 16, React 19, TypeScript, Tailwind CSS 4, Zustand 5, Supabase, Motion 12, Recharts 3, date-fns 4) is NOT re-evaluated here.

---

## 1. Cromford Appreciation Chart: Data Extraction Strategy

### The Problem

The Cromford Report publishes price-per-square-foot appreciation charts on Tableau Public (confirmed URL: `public.tableau.com/views/AnnualAverageSalesPriceperSquareFootbyCity/AnnualAverageSPSF`). We need to extract this data and display it on homeowner pages.

### Recommended Approach: Three-Tier Strategy

**Tier 1 (Primary): Manual Admin Data Entry + Cached Storage**

No new library needed. Josh enters Cromford data points manually in the admin form (city, year, price-per-sqft). Store in Supabase. This matches the v1 philosophy of "manual data input plus Cromford chart is sufficient" (per PROJECT.md out-of-scope notes). At 20-30 pages/month, entering a few data points per city is trivial.

Why this is the right primary approach:
- Cromford data changes infrequently (monthly or quarterly updates)
- A city's historical price-per-sqft data is the same across all homeowners in that city
- One data entry serves every homeowner in that city forever
- No scraper maintenance, no API keys, no breakage risk

**Tier 2 (Enhancement): Tableau Embedding API for Client-Side Chart Display**

| Technology | Version | Purpose | Confidence |
|------------|---------|---------|------------|
| Tableau Embedding API v3 | loaded from Tableau Public server | Embed actual Cromford chart in homeowner page | MEDIUM |

For Tableau Public, the official recommendation is to load the Embedding API from the server URL rather than npm, because the server always provides the latest compatible version:

```html
<script type="module"
  src="https://public.tableau.com/javascripts/api/tableau.embedding.3.latest.min.js">
</script>
```

This can embed the live Cromford chart directly in the homeowner page. The `getSummaryDataReaderAsync()` method can extract the underlying data for use in calculations. However, this is a CLIENT-SIDE API -- it requires a browser context and the Tableau viz to be loaded in the DOM.

Limitations:
- Requires the Cromford Tableau Public viz to remain at a stable URL
- Data extraction happens in the browser, not at build/generation time
- If Cromford changes their Tableau structure, it breaks
- Adds ~200KB of Tableau JS to the page

**Tier 3 (Fallback): Screenshot + OCR**

If the Tableau chart cannot be embedded (e.g., Cromford blocks embedding), fall back to a screenshot uploaded by Josh with optional OCR for data extraction.

| Technology | Version | Purpose | Confidence |
|------------|---------|---------|------------|
| tesseract.js | ^7.0.0 | OCR text extraction from chart screenshots | HIGH |

tesseract.js v7.0.0 (released December 2025) is the current major version. Key improvements over v5/v6:
- 15-35% faster runtimes via relaxedsimd WASM build
- Fixed memory leak that caused crashes in long-running processes
- Requires Node.js v16+ (we use Node 20+, so no issue)
- Pure JS, runs in browser and Node.js

Reality check on OCR for chart data extraction: OCR works well for axis labels and data point values when they are rendered as text on the chart. It does NOT work for reading values from plotted line positions. For a price-per-sqft chart, OCR can reliably extract:
- Y-axis dollar values ($150, $200, $250...)
- X-axis year labels (2015, 2016, 2017...)
- Data labels if the chart renders them as text

OCR CANNOT reliably extract:
- The exact value where a line intersects a grid line
- Interpolated data points between labeled values

### Recommendation

**Use Tier 1 (manual entry) as the primary approach.** Create a `cromford_data` Supabase table that stores city-level price-per-sqft by year/quarter. Josh enters this data once per city and it serves all homeowners in that city. Render the chart using Recharts (already in the stack) with this stored data.

**Use Tier 2 (Tableau embed) as an optional enhancement** only if you want to show the live Cromford chart alongside the Recharts version. Do NOT depend on it for data extraction.

**Use Tier 3 (OCR) only if** Josh wants to upload a screenshot and have the system attempt to extract numbers from it. This is a "nice to have" that adds complexity. Defer to a later phase unless there is a strong reason to include it.

### What NOT to Use for Cromford Data

| Avoid | Why |
|-------|-----|
| `scrape-tableau` (npm) | Last published 4 years ago (v1.0.0). Zero active maintenance. Zero other npm dependents. Dead package. |
| `TableauScraper` (Python/pip) | Python library -- would require a separate Python service or subprocess. Does not fit the Next.js/TypeScript stack. |
| Puppeteer on Vercel for scraping | Puppeteer + @sparticuz/chromium-min works on Vercel but has severe constraints: 50MB function size limit, 10-second timeout on free tier, 250-400MB memory usage, cold starts of 3-5 seconds. Using a headless browser to scrape a Tableau chart is fragile, slow, and expensive in serverless. |
| `@tableau/embedding-api` (npm) | Tableau's own docs say: for Tableau Public, load from the server URL, not npm. The npm package is for Tableau Server/Cloud where you control versions. |

---

## 2. Value Range Selector & Condition Picker

### No New Library Needed

The value range selector (low/high comp values with condition picker: original/updated/remodeled) is a UI component, not a library problem. Build it with:

- **HTML `<input type="range">`** styled with Tailwind CSS for the range slider
- **Radio buttons or segmented control** for condition picker (original/updated/remodeled)
- **Zustand** (already in stack) for state management

Why NOT add a slider library:

| Library | Why Skip |
|---------|----------|
| `rc-slider` | 45KB gzipped. Adds React class component patterns. Styling fights Tailwind. Overkill for a single dual-thumb slider. |
| `@radix-ui/react-slider` | Good library, but adds Radix as a dependency for one component. The native HTML range input + Tailwind is sufficient. If you later need multiple Radix primitives, reconsider. |
| `react-range` | Headless, which is good. But 8KB for what a styled `<input type="range">` already does. |

**If a dual-thumb range slider is needed** (homeowner drags between low and high value), then `@radix-ui/react-slider` is the best option because:
- Headless/unstyled -- works with Tailwind
- ARIA-compliant out of the box
- Supports range (dual thumb) natively
- 3.5KB gzipped
- Active maintenance (Radix is the foundation of shadcn/ui)

| Technology | Version | Purpose | When to Add |
|------------|---------|---------|-------------|
| @radix-ui/react-slider | ^1.3.2 | Dual-thumb range slider for value selection | Only if dual-thumb UX is confirmed in design. Skip if condition picker uses 3 preset buttons instead. |

### Condition Picker Design

The condition picker (original/updated/remodeled) maps to value adjustments. This is a calculation concern, not a UI library concern. The admin enters three values (low/mid/high or original/updated/remodeled) and the homeowner picks one. When they pick, Zustand updates `selectedCondition`, which cascades through the calculation engine.

---

## 3. Cascading Recalculation Architecture

### No New Library Needed

The existing calculation engine (`src/lib/calculations.ts`) is a pure function: `computeHomeownerMetrics(input) => HomeownerMetrics`. Cascading recalculation means:

1. Homeowner selects a condition (original/updated/remodeled)
2. This changes `currentValue` in the input
3. `computeHomeownerMetrics()` re-runs with the new value
4. All downstream metrics update (equity, scenarios, comparisons, net proceeds)

This is already how the engine works -- it is a pure function of inputs. The "cascade" is just React re-rendering when Zustand state changes.

### Implementation Pattern

```typescript
// New: homeowner-facing store (separate from admin store)
interface HomeownerPageState {
  // Data from Supabase (set once on page load)
  baseData: HomeownerRecord;
  valueRange: {
    original: number;  // low estimate
    updated: number;   // mid estimate
    remodeled: number; // high estimate
  };

  // Homeowner selections (interactive)
  selectedCondition: 'original' | 'updated' | 'remodeled';
  projectionYears: 5 | 10 | 15;
  marketRateOverride: number | null; // from slider, null = default

  // Derived (computed via useMemo, not stored)
  // metrics: HomeownerMetrics -- computed from selectedCondition + baseData
}
```

The key architectural decision: do NOT store computed metrics in Zustand. Use `useMemo` in the component tree to derive metrics from the selected condition. This is the same pattern the STR Analyzer uses ("Changes to any field trigger useMemo-based recalculation").

### Cascading Animation (UX)

The "ripple animation on condition change" is a Motion concern, not a library concern. Use Motion's `layout` prop and `AnimatePresence` (already in the stack) to animate number changes when the condition picker updates.

```typescript
// Example: animate number transitions
<motion.span
  key={formattedValue}  // re-mount on value change
  initial={{ opacity: 0, y: 10 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -10 }}
>
  {formattedValue}
</motion.span>
```

No new library needed.

---

## 4. Future Scenarios: 5/10/15yr Projections & Market Rate Slider

### Calculation Engine Extensions

The existing `calculations.ts` needs NEW functions but no new libraries. Extensions needed:

| Function | Purpose | Inputs |
|----------|---------|--------|
| `calcProjectedValue()` | Project home value at 5/10/15 years | currentValue, appreciationRate, years |
| `calcProjectedEquity()` | Project equity at 5/10/15 years | projectedValue, projectedMortgageBalance |
| `calcProjectedCashFlow()` | Project rental cash flow over time | monthlyRent (growing), monthlyMortgage (fixed), years |
| `calcComboScenario()` | "Keep, rent, and buy another" | holdAndRent + moveUp combined |
| `calcRateLockAdvantage()` | Show savings from locked rate vs current market | lockedRate, currentMarketRate, loanBalance |

All of these are pure math functions using the same patterns already in `calculations.ts`. No external library needed.

### Market Rate Slider

Same UI decision as the value range selector. A single-thumb slider for market appreciation rate (e.g., 2%-8%) built with native HTML range input + Tailwind. The slider value feeds into the projection functions.

---

## 5. Supabase Schema Additions

### New Columns on `homeowners` Table

```sql
-- Value range: admin enters three estimates
value_original    numeric,  -- original condition estimate
value_updated     numeric,  -- updated condition estimate
value_remodeled   numeric,  -- remodeled condition estimate

-- The existing current_value column becomes the "selected" value
-- (default to value_updated if range is provided)
```

### New Table: `cromford_data`

```sql
CREATE TABLE cromford_data (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at      timestamptz DEFAULT now(),

  -- Geographic scope
  city            text NOT NULL,
  state           text DEFAULT 'AZ',

  -- Time period
  year            int NOT NULL,
  quarter         int,  -- NULL = annual, 1-4 = quarterly

  -- Data points
  avg_price_per_sqft  numeric NOT NULL,
  median_sale_price   numeric,

  -- Source tracking
  source_note     text,  -- e.g., "Cromford Report March 2026"

  UNIQUE(city, state, year, quarter)
);

-- RLS: public read (published pages need this data)
ALTER TABLE cromford_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can view cromford data"
  ON cromford_data FOR SELECT
  USING (true);

-- Index for city+year lookups
CREATE INDEX idx_cromford_city_year ON cromford_data (city, year);
```

### Why a Separate Table

Cromford data is city-level, not homeowner-level. One Phoenix data set serves all Phoenix homeowners. Storing it per-homeowner would be massive duplication. A separate table with a city+year lookup is the correct normalization.

---

## Summary: What to Install

### Required (HIGH confidence)

Nothing new is strictly required. The existing stack handles all v1.1 features:
- **Calculations:** Pure TypeScript functions (extend `calculations.ts`)
- **UI sliders:** Native HTML range inputs + Tailwind styling
- **Cascading recalc:** Zustand state change triggers `useMemo` recomputation
- **Cromford chart:** Recharts rendering of manually-entered data from new Supabase table
- **Animations:** Motion (already installed) for cascading number transitions

### Optional (MEDIUM confidence)

| Library | Version | Purpose | Install If... |
|---------|---------|---------|---------------|
| @radix-ui/react-slider | ^1.3.2 | Accessible dual-thumb range slider | Design requires dual-thumb slider for value range (not just 3 preset buttons) |
| tesseract.js | ^7.0.0 | OCR fallback for Cromford screenshots | Josh wants to upload chart screenshots and auto-extract data |

### Do NOT Install

| Library | Reason |
|---------|--------|
| scrape-tableau | Dead package, unmaintained 4+ years |
| TableauScraper (Python) | Wrong language for this stack |
| puppeteer / puppeteer-core | Serverless constraints make Tableau scraping impractical |
| @sparticuz/chromium-min | Only needed with puppeteer; not recommended |
| @tableau/embedding-api (npm) | Use server URL instead for Tableau Public |
| rc-slider | Unnecessary weight; Radix or native HTML suffices |
| Any charting library | Recharts already handles all chart needs |
| Any animation library | Motion already handles all animation needs |
| Any state management library | Zustand already handles all state needs |

### Conditional Install Commands

```bash
# Only if dual-thumb slider UX is confirmed:
npm install @radix-ui/react-slider

# Only if OCR fallback is needed:
npm install tesseract.js
```

---

## Integration Points with Existing Stack

### Zustand Store Changes

The existing `useAdminStore` (admin form) needs new fields for value range entry. A NEW `usePageStore` (or inline `useMemo` in the homeowner page component) handles homeowner-facing interactive state (condition selection, projection years, market rate slider).

Do NOT merge admin state and homeowner-page state into one store. They serve different users (Josh vs homeowner) at different times (page creation vs page viewing).

### Calculation Engine Changes

`computeHomeownerMetrics()` currently takes a single `currentValue`. For v1.1, it needs to accept value-range-aware input:

```typescript
interface HomeownerInputV1_1 extends HomeownerInput {
  valueRange?: {
    original: number;
    updated: number;
    remodeled: number;
  };
  selectedCondition?: 'original' | 'updated' | 'remodeled';
  projectionYears?: 5 | 10 | 15;
  marketRateOverride?: number;
}
```

The engine uses `selectedCondition` to pick the `currentValue` from the range, then runs all existing calculations. New projection functions are additive -- they do not change existing calculation signatures.

### Recharts Integration

The Cromford price-per-sqft chart uses the same Recharts `<AreaChart>` or `<LineChart>` patterns already in the project. Data comes from the `cromford_data` Supabase table, fetched server-side and passed as props.

---

## Sources

- Tableau Embedding API v3 docs: https://help.tableau.com/current/api/embedding_api/en-us/index.html (HIGH confidence)
- Tableau Embedding API React components: https://help.tableau.com/current/api/embedding_api/en-us/docs/embedding_api_react.html (HIGH confidence)
- Tableau Public Cromford chart URL: https://public.tableau.com/views/AnnualAverageSalesPriceperSquareFootbyCity/AnnualAverageSPSF (MEDIUM confidence -- URL may change)
- Tableau data extraction methods: https://help.tableau.com/current/api/embedding_api/en-us/docs/embedding_api_data.html (HIGH confidence)
- tesseract.js v7.0.0 release: https://github.com/naptha/tesseract.js/releases (HIGH confidence)
- scrape-tableau npm: https://www.npmjs.com/package/scrape-tableau -- last published 4 years ago (HIGH confidence on "avoid" recommendation)
- Radix UI Slider: https://www.radix-ui.com/primitives/docs/components/slider (HIGH confidence)
- Puppeteer on Vercel guide: https://www.danielolawoyin.com/blog/puppeteer-on-vercel-the-ultimate-guide-to-serverless-browser-automation-2026 (HIGH confidence)
- Vercel serverless function limits: https://github.com/vercel/community/discussions/103 (HIGH confidence)
- Existing project package.json: /Users/joshuahogan/Projects/homeowner-journey-map/package.json (HIGH confidence)
- Existing calculations.ts: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/calculations.ts (HIGH confidence)
- Existing schema.sql: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/supabase/schema.sql (HIGH confidence)

---
*Stack additions research for: Homeowner Journey Map v1.1 Enhanced Intelligence*
*Researched: 2026-03-13*
