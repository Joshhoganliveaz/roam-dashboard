# Architecture Patterns: v1.1 Enhanced Intelligence

**Domain:** Homeowner Journey Map -- v1.1 feature integration with existing Next.js + Zustand + Supabase architecture
**Researched:** 2026-03-13
**Focus:** How new features integrate with existing codebase, not greenfield architecture

## Existing Architecture Summary

The v1.0 architecture is a clean server-first pipeline:

```
Admin Form (Zustand store) --> Supabase --> Server Component fetches row
  --> applyDefaults() --> computeHomeownerMetrics() --> HomeownerPage (client)
```

Key characteristics:
- **Server-side calculation:** `/h/[slug]/page.tsx` is a Server Component that fetches data, runs `computeHomeownerMetrics()`, and passes pre-computed `record` + `metrics` as props to the client `HomeownerPage`
- **No client-side recalculation:** HomeownerPage receives everything pre-computed. All interactivity is cosmetic (expand/collapse, animations)
- **Single `currentValue`:** The entire pipeline assumes one value. Changing this to a range is the deepest architectural change in v1.1
- **Pure calculation engine:** `calculations.ts` is stateless, testable, side-effect-free
- **Admin Zustand store:** Only used on `/admin` routes for form state. NOT used on public homeowner pages

## v1.1 Architectural Shift: Server-Computed to Client-Reactive

The core architectural change in v1.1 is that the homeowner page **gains interactive state**. The condition picker means the page must recalculate downstream numbers client-side when the homeowner selects a condition. This is the single biggest structural change.

### The Shift

**v1.0:** Server computes everything. Client renders static data with animations.
**v1.1:** Server computes initial values. Client holds condition state and recalculates on change.

This does NOT mean moving `computeHomeownerMetrics()` to the client. It means:
1. Server computes metrics for all three conditions (original/updated/remodeled) at render time
2. Client receives three pre-computed metric sets
3. Condition picker switches which metric set is displayed
4. Animation cascades through sections as values change

This is the right approach because the calculation engine is pure and fast, the input range is bounded (exactly 3 conditions), and pre-computing avoids shipping the entire calculation engine to the client.

---

## Feature-by-Feature Integration

### 1. Value Range Selector + Condition Picker

**Database changes:**
```
homeowners table additions:
  value_low      NUMERIC  -- bottom of comp range (Josh enters)
  value_high     NUMERIC  -- top of comp range (Josh enters)
```

The existing `current_value` column is retained and repurposed as the "updated" (40th percentile) value. On save, the admin form computes: `current_value = value_low + (value_high - value_low) * 0.4`.

**Admin form changes:**
- Replace single "Current Value" input with two inputs: "Low Comp Value" and "High Comp Value"
- Store `useAdminStore` gains `valueLow` and `valueHigh` fields
- The `currentValue` field becomes derived (computed on save, not entered)
- Add these two fields to `SaveHomeownerInput` in `actions.ts` and the Supabase insert/update

**Types changes:**
```typescript
// types.ts additions
export type PropertyCondition = "original" | "updated" | "remodeled";

export interface HomeownerRecord {
  // ... existing fields ...
  valueLow: number;       // NEW: bottom of comp range
  valueHigh: number;      // NEW: top of comp range
  // currentValue remains -- set to 40th percentile on save
}

// NEW: Pre-computed metrics for all three conditions
export interface ConditionMetrics {
  original: HomeownerMetrics;
  updated: HomeownerMetrics;
  remodeled: HomeownerMetrics;
  values: {
    original: number;     // valueLow
    updated: number;      // 40th percentile
    remodeled: number;    // valueHigh
  };
}
```

**Server component changes (`/h/[slug]/page.tsx`):**
```typescript
// Compute metrics for all three conditions
const conditionValues = {
  original: record.valueLow,
  updated: record.valueLow + (record.valueHigh - record.valueLow) * 0.4,
  remodeled: record.valueHigh,
};

const conditionMetrics: ConditionMetrics = {
  original: computeHomeownerMetrics({ ...input, currentValue: conditionValues.original }),
  updated: computeHomeownerMetrics({ ...input, currentValue: conditionValues.updated }),
  remodeled: computeHomeownerMetrics({ ...input, currentValue: conditionValues.remodeled }),
  values: conditionValues,
};
```

**Client-side state (NEW -- homeowner page gets state):**

Do NOT use Zustand for this. The existing Zustand store is admin-only. The homeowner page needs a simple `useState` for the condition:

```typescript
// HomeownerPage.tsx
const [condition, setCondition] = useState<PropertyCondition>("updated");
const metrics = conditionMetrics[condition];
const currentValue = conditionMetrics.values[condition];
```

This is a React state + derived value pattern, not a store pattern. Three possible states, three pre-computed results. No recalculation happens client-side.

**Component: ValueRangeSelector (NEW)**
- Placement: after PersonalNote, before PurchaseStory (replaces CurrentValue position in narrative flow)
- Shows the range: "$X - $Y based on recent sales nearby"
- Three tappable buttons: "Mostly original" / "Made some updates" / "Completely remodeled"
- Selected condition highlights, value displays with count-up animation
- On change: triggers cascading animation (see UX section below)

**Cascading recalc approach:**
- No actual recalculation happens -- the client switches between pre-computed metric objects
- The "cascade" is purely visual: a staggered animation that ripples down sections as the selected metrics change
- Each section that depends on `metrics` re-renders with new values and triggers its entrance animation

### 2. Cromford Appreciation Chart (Tableau Scraper)

**This is a server-side data pipeline, not a client-side feature.**

**New API route: `/api/cromford` (POST)**
```
Input:  { tableauUrl: string, sqft: number }
Output: { dataPoints: Array<{ date: string, pricePSF: number }>, error?: string }
```

**Architecture:**
- The Tableau scraper runs server-side only (API route, not Server Action)
- API route because: it involves external HTTP requests to Tableau Public, may take several seconds, and is called from the admin form during page creation -- not during public page rendering
- Uses `tableau-scraper` npm package (or Python microservice if npm package is insufficient)
- Admin workflow: Josh pastes Cromford share URL, enters sqft. System extracts price-per-sqft time series
- Extracted data points are stored in the homeowner record (JSON column or separate table)

**Database changes:**
```
homeowners table additions:
  cromford_data    JSONB  -- Array of { date: string, pricePSF: number }
  cromford_sqft    NUMERIC -- sqft used for price calculation
```

**Why JSONB in the homeowners table (not a separate table):**
- One chart per homeowner, always fetched together
- Data is static after extraction (not queried independently)
- Avoids JOIN overhead on the critical read path

**Chart component changes:**
- Existing `AppreciationChart.tsx` currently generates synthetic data from `annualizedAppreciation`
- v1.1: When `cromfordData` is present, use real data points instead of synthetic curve
- When absent, fall back to current synthetic approach (backward compatible)
- Chart multiplies each `pricePSF * sqft` to get estimated value at each point
- Condition selector shifts the curve: each data point is scaled proportionally to the selected condition value vs the "updated" baseline

**Data flow:**
```
Admin enters Tableau URL + sqft
  --> POST /api/cromford (server-side scrape)
  --> Returns time series data points
  --> Stored in cromford_data JSONB column
  --> At page render: server passes cromford_data to client
  --> AppreciationChart uses real data if available, synthetic if not
```

**Important architectural note:** The Tableau scraper is NOT called at page render time. It runs once during admin data entry. The extracted data is stored and served like any other column.

### 3. OCR Fallback for Cromford Screenshots

**New API route: `/api/cromford-ocr` (POST)**
```
Input:  FormData with screenshot file + sqft
Output: { dataPoints: Array<{ date: string, pricePSF: number }>, confidence: number }
```

**Architecture:**
- Server Action or API route that accepts a file upload
- Sends the image to an AI vision API (OpenAI GPT-4o or Claude) for data extraction
- Prompt engineering: "Extract the monthly price-per-sqft data points from this Cromford Report chart. Return as JSON array of {date, pricePSF}."
- Returns structured data in the same format as the Tableau scraper
- The admin form shows a "Upload Screenshot" option alongside the Tableau URL input
- Both paths produce identical output format -- the chart component does not know which source was used

**Why a separate route (not combined with `/api/cromford`):**
- Different input formats (URL vs file upload)
- Different processing (HTTP scrape vs vision API)
- Different error modes and latency characteristics
- Admin UI shows them as separate options with clear labels

**Admin form integration:**
```
CromfordInput component (NEW):
  Tab 1: "Paste Tableau URL" --> calls /api/cromford
  Tab 2: "Upload Screenshot" --> calls /api/cromford-ocr
  Both update the same cromford_data field in the form store
  Preview shows extracted data points for verification before save
```

### 4. Market Rate Slider

**This is pure client-side state. No database changes.**

The market rate slider is an interactive element on the homeowner page that lets the homeowner explore how different interest rate environments affect their scenarios.

**State:**
```typescript
// HomeownerPage.tsx
const [marketRate, setMarketRate] = useState(DEFAULTS.currentMarketRate); // e.g., 0.065
```

**Impact on calculations:**

The market rate slider CANNOT use the "pre-compute everything server-side" pattern because the rate is continuous (not 3 discrete values). This means scenario calculations that depend on market rate must run client-side.

**Hybrid approach:**
- Core metrics (equity, appreciation, earnings, SP500, rent vs own) are STILL server-computed (they do not depend on market rate)
- Only the future scenario calculations (Hold & Rent refi math, Move-Up purchasing power, Equity Play HELOC cost, Combo scenario) recalculate client-side when the slider moves
- Extract scenario calculation functions into a separate file (`src/lib/scenario-calculations.ts`) that is imported by both the server page and the client ScenarioCards component

```typescript
// scenario-calculations.ts (NEW -- shared between server and client)
export function calcHoldAndRentProjection(
  metrics: HomeownerMetrics,
  marketRate: number,
  lockedRate: number,
  years: number
): HoldAndRentProjection { ... }

export function calcMoveUpProjection(
  metrics: HomeownerMetrics,
  marketRate: number,
  years: number
): MoveUpProjection { ... }

export function calcEquityPlayProjection(
  metrics: HomeownerMetrics,
  marketRate: number
): EquityPlayProjection { ... }

export function calcComboScenario(
  holdAndRent: HoldAndRentProjection,
  equityPlay: EquityPlayProjection,
  moveUp: MoveUpProjection
): ComboScenarioProjection { ... }
```

**Key principle:** The scenario calculation functions must be lightweight and fast enough to run on mobile without jank. The current `calcHoldAndRent`, `calcMoveUp`, and `calcEquityPlay` functions are already fast (simple arithmetic, no loops except the break-even projection). Adding market rate as a parameter does not change their performance characteristics.

**Rate lock advantage callout:**
- Computed client-side: `savings = (marketRate - lockedRate) * mortgageBalance * yearsRemaining` (simplified)
- Displayed as a callout within the scenarios section: "Your 3.0% rate saves $X/year compared to today's rates"
- Only shown when `lockedRate < marketRate`

### 5. Merged Investment Comparison Section

**This replaces two existing components with one new component.**

**Components affected:**
- `SP500Callout.tsx` -- REMOVED (absorbed into InvestmentComparison)
- `RentVsOwn.tsx` -- REMOVED (absorbed into InvestmentComparison)
- `InvestmentComparison.tsx` -- NEW (replaces both)

**Migration path:**
1. Create `InvestmentComparison.tsx` with combined logic
2. Update `HomeownerPage.tsx` to render `InvestmentComparison` instead of separate `SP500Callout` + `RentVsOwn`
3. Remove old components only after new component is verified
4. The narrative position stays in the same place (after AppreciationChart, before NetProceeds)

**Data dependencies (no new calculations needed):**
- Uses existing `metrics.sp500Comparison` (home return vs S&P)
- Uses existing `metrics.rentVsOwn` (total rent paid, equity built)
- NEW calculation: leverage return = `(currentValue - purchasePrice) / downPayment * 100` (percent return on actual cash invested)
- This leverage calculation is simple enough to compute inline or add to `computeHomeownerMetrics`

**Component structure:**
```
InvestmentComparison
  |-- Leverage callout (your $25K controlled $500K)
  |-- Three-column comparison:
  |     Home ROI | S&P Alternative | Rent Drag
  |-- Net result narrative (homeowner wins because rent drag kills S&P path)
  |-- Data source footnote
```

**The section responds to condition changes** because `sp500Comparison` and `rentVsOwn` are derived from `currentValue`, which changes per condition.

### 6. Fourth Combo Scenario ("Keep, Rent, and Buy Another")

**This chains math from the three existing scenarios.**

**Data flow:**
```
Hold & Rent output (cash flow, rental income)
  + Equity Play output (HELOC available, HELOC cost at market rate)
  + Move-Up output (purchasing power from HELOC funds as down payment)
  = Combo Scenario
```

**New calculation function:**
```typescript
// scenario-calculations.ts
export function calcComboScenario(
  holdAndRent: HoldAndRentProjection,
  equityPlay: EquityPlayProjection,
  marketRate: number
): ComboScenarioProjection {
  // HELOC cost folds into rental holding costs
  const helocMonthlyPayment = equityPlay.helocAt80Pct * (marketRate / 12);
  const adjustedCashFlow = holdAndRent.cashFlow - helocMonthlyPayment;

  // HELOC funds become next down payment
  const nextDownPayment = equityPlay.helocAt80Pct;
  const nextHomePurchasingPower = nextDownPayment / 0.2;

  return {
    monthlyRentalIncome: holdAndRent.monthlyRent,
    monthlyMortgageOnCurrent: holdAndRent.monthlyMortgage,
    monthlyHelocPayment: helocMonthlyPayment,
    netMonthlyCashFlow: adjustedCashFlow,
    helocForDownPayment: nextDownPayment,
    nextHomePurchasingPower: nextHomePurchasingPower,
    // ... 5/10/15yr projections
  };
}
```

**Component: fourth ScenarioCard in the grid.**
- Uses the same `ScenarioCard` component pattern (title, heroMetric, implication, expandable details)
- The combo scenario card is visually distinct (maybe gold border instead of standard) to signal it is the "power move"
- It depends on the market rate slider value (recalculates when slider changes)

### 7. 5/10/15 Year Projections for All Scenarios

**This extends the scenario calculation output, not the component architecture.**

Each scenario calculation function gains a `projections` array:

```typescript
export interface ScenarioProjection {
  year: 5 | 10 | 15;
  projectedValue: number;
  projectedEquity: number;
  cumulativeCashFlow?: number;     // Hold & Rent
  projectedRent?: number;          // Hold & Rent (rent growth)
  purchasingPower?: number;        // Move-Up
  helocAvailable?: number;         // Equity Play
}
```

**Component change:** ScenarioCard gains a projection toggle or tab strip (5yr / 10yr / 15yr) inside the expanded details view. This is local `useState` within ScenarioCard -- not a global state concern.

**Appreciation rate for projections:** Uses `DEFAULTS.appreciationRate` (4%) for conservative, with the market rate slider allowing the user to see how different appreciation assumptions change outcomes.

---

## New Component Map

```
HomeownerPage.tsx (MODIFIED -- gains condition + marketRate state)
  |
  |-- HeroSection (unchanged)
  |-- PersonalNote (unchanged)
  |-- ValueRangeSelector (NEW -- condition picker)
  |-- PurchaseStory (unchanged -- uses record.purchasePrice, not currentValue)
  |-- CurrentValue (MODIFIED -- displays conditionMetrics.values[condition])
  |-- EquityGained (MODIFIED -- reads from selected metrics)
  |-- EarningsReframe (MODIFIED -- reads from selected metrics)
  |-- AppreciationChart (MODIFIED -- uses cromfordData when available, shifts by condition)
  |-- InvestmentComparison (NEW -- replaces SP500Callout + RentVsOwn)
  |-- NetProceeds (MODIFIED -- reads from selected metrics)
  |-- MarketRateSlider (NEW -- controls marketRate state)
  |-- ScenarioCards (MODIFIED -- gains 4th card, projection tabs, market rate input)
  |     |-- ScenarioCard: Hold & Rent (MODIFIED -- projections, rate lock callout)
  |     |-- ScenarioCard: Move-Up (MODIFIED -- projections, golden handcuffs)
  |     |-- ScenarioCard: Equity Play (MODIFIED -- projections, rate-adjusted HELOC)
  |     |-- ScenarioCard: Keep, Rent & Buy (NEW -- combo scenario)
  |-- CTASection (unchanged)
```

**Removed components:**
- `SP500Callout.tsx` -- absorbed into InvestmentComparison
- `RentVsOwn.tsx` -- absorbed into InvestmentComparison

**New files:**
- `src/components/homeowner/ValueRangeSelector.tsx`
- `src/components/homeowner/InvestmentComparison.tsx`
- `src/components/homeowner/MarketRateSlider.tsx`
- `src/lib/scenario-calculations.ts` (extracted from calculations.ts, shared server/client)
- `src/app/api/cromford/route.ts`
- `src/app/api/cromford-ocr/route.ts`

**Modified files:**
- `src/lib/types.ts` -- new types
- `src/lib/calculations.ts` -- leverage calculation, condition metrics
- `src/lib/defaults.ts` -- new defaults (currentMarketRate, projection years)
- `src/app/h/[slug]/page.tsx` -- computes conditionMetrics
- `src/components/homeowner/HomeownerPage.tsx` -- gains state, new section order
- `src/components/homeowner/CurrentValue.tsx` -- reads from condition-selected value
- `src/components/homeowner/EquityGained.tsx` -- reads from condition-selected metrics
- `src/components/homeowner/EarningsReframe.tsx` -- reads from condition-selected metrics
- `src/components/homeowner/AppreciationChart.tsx` -- cromford data support, condition shifting
- `src/components/homeowner/NetProceeds.tsx` -- reads from condition-selected metrics
- `src/components/homeowner/ScenarioCards.tsx` -- 4 cards, market rate, projections
- `src/components/homeowner/ScenarioCard.tsx` -- projection tabs
- `src/components/admin/HomeownerForm.tsx` -- value range inputs, Cromford input

---

## Cascading Animation on Condition Change

**Pattern:** When the condition picker changes, sections that display condition-dependent data should animate their value transitions.

**Implementation approach:**
- Use a React `key` strategy: when `condition` changes, sections re-mount with new data
- Each section already uses `AnimatedSection` with `whileInView` + `once: true`
- For the cascade effect: remove `once: true` on condition-dependent sections and add a `key={condition}` so they re-animate when condition changes
- Add a staggered delay based on section position: section N gets `delay: N * 150ms`
- CountUpNumber already animates from 0 -- when re-mounted via key change, it auto-animates to the new value

**Alternative (smoother):** Instead of re-mounting, use `motion.animate` to transition values. This requires CountUpNumber to animate from old value to new value (not from 0). More polished, but more complex.

**Recommendation:** Start with the `key={condition}` re-mount approach. It reuses existing animation infrastructure. Upgrade to value-to-value transitions in a polish pass if the re-mount feels jarring.

---

## Supabase Schema Changes

```sql
-- Add to existing homeowners table
ALTER TABLE homeowners ADD COLUMN value_low NUMERIC;
ALTER TABLE homeowners ADD COLUMN value_high NUMERIC;
ALTER TABLE homeowners ADD COLUMN cromford_data JSONB;
ALTER TABLE homeowners ADD COLUMN cromford_sqft NUMERIC;

-- current_value remains and is computed: value_low + (value_high - value_low) * 0.4
-- For backward compatibility, existing rows with value_low = NULL
-- continue to use current_value as-is (single value, no range)
```

**Backward compatibility:** All new columns are nullable. Existing pages without a value range display exactly as they do today. The condition picker only appears when `valueLow` and `valueHigh` are both set.

---

## Build Order (Dependency-Driven)

### Phase 1: Value Range Foundation
**Why first:** Every other feature depends on the condition-aware data flow. The value range changes the server component, types, calculations, and client state. Build this first so all subsequent features integrate naturally.

1. Schema migration (add `value_low`, `value_high` columns)
2. Update types (`PropertyCondition`, `ConditionMetrics`, updated `HomeownerRecord`)
3. Update `applyDefaults` and `computeHomeownerMetrics` to support condition variants
4. Update server component to compute `conditionMetrics`
5. Update `HomeownerPage` to hold `condition` state and thread selected metrics through
6. Build `ValueRangeSelector` component
7. Update downstream components to read from selected metrics
8. Update admin form with low/high value inputs
9. Cascading animation on condition change
10. Backward compatibility: pages without value range still work

### Phase 2: Merged Investment Comparison
**Why second:** Removes two components and adds one. Best done before adding projection complexity to the scenario section, since it clarifies the page structure.

1. Build `InvestmentComparison.tsx` with leverage callout, three-column layout
2. Add leverage calculation to `computeHomeownerMetrics`
3. Update `HomeownerPage` to swap SP500Callout + RentVsOwn for InvestmentComparison
4. Verify condition picker cascades into new section correctly
5. Remove old components

### Phase 3: Future Scenarios Rework
**Why third:** Builds on condition-aware metrics (Phase 1) and clean page structure (Phase 2).

1. Extract scenario calculations into `scenario-calculations.ts`
2. Add 5/10/15yr projection logic to each scenario function
3. Add market rate parameter to scenario functions
4. Build `MarketRateSlider` component
5. Update `ScenarioCards` to pass market rate, render projection tabs
6. Update `ScenarioCard` with projection toggle UI
7. Build combo scenario calculation function
8. Add 4th "Keep, Rent & Buy" ScenarioCard
9. Rate lock advantage callout (conditional on locked rate < market rate)
10. Information cliff removal (expand details by default or remove cliff questions)

### Phase 4: Cromford Data Integration
**Why fourth:** Independent of the other features. Can be done last because the existing synthetic chart works as a fallback. The chart component already exists -- this phase adds real data support.

1. Schema migration (add `cromford_data`, `cromford_sqft` columns)
2. Build `/api/cromford` route (Tableau scraper)
3. Build `/api/cromford-ocr` route (vision API fallback)
4. Build `CromfordInput` admin component (URL tab + screenshot tab)
5. Update `AppreciationChart` to use cromford data when available
6. Condition-based curve shifting with real data
7. Update admin form to include CromfordInput

### Phase 5: UX Polish
**Why last:** All features are functional. This phase refines the experience.

1. Cascading ripple animation refinement (value-to-value transitions if key-based feels jarring)
2. Information cliff removal across all sections
3. Mobile testing and responsive fixes for new components
4. Loading states for Cromford data extraction
5. Error handling for failed scrapes/OCR

---

## Anti-Patterns to Avoid

### Anti-Pattern: Client-Side Zustand Store for Homeowner Page
**What:** Adding condition/marketRate to a new Zustand store shared across homeowner components.
**Why bad:** The homeowner page has exactly two pieces of interactive state (condition + marketRate). Zustand adds complexity for what `useState` + prop drilling handles perfectly. The existing admin store is correctly scoped to admin routes -- do not conflate the two.
**Instead:** `useState` in HomeownerPage, pass down as props. If prop drilling becomes painful (more than 3 levels), use React context as an intermediate step before reaching for Zustand.

### Anti-Pattern: Computing All Rate Permutations Server-Side
**What:** Pre-computing scenario projections for every possible market rate (5.0%, 5.1%, 5.2%...).
**Why bad:** The rate slider is continuous. You cannot pre-compute infinite values. The scenario functions are simple arithmetic that runs in microseconds.
**Instead:** Pre-compute condition variants server-side (3 discrete values). Compute rate-dependent scenarios client-side on slider change.

### Anti-Pattern: Calling Tableau Scraper at Page Render Time
**What:** Having the server component fetch Cromford data live when a homeowner visits their page.
**Why bad:** External dependency at render time. If Tableau is slow or down, the page breaks. The data does not change between visits.
**Instead:** Scrape once during admin data entry, store the result, serve it like any other column.

### Anti-Pattern: Separate Database Table for Cromford Data Points
**What:** Creating a `cromford_data_points` table with foreign key to homeowners.
**Why bad:** The data is always fetched with the homeowner record, never queried independently. A JOIN adds latency for no benefit. The data is small (12-60 data points per homeowner).
**Instead:** JSONB column on the homeowners table. Queried in one SELECT, no JOIN.

## Sources

- Direct code review of `/Users/joshuahogan/Projects/homeowner-journey-map/src/` -- HIGH confidence
- v1.0 architecture research at `/Users/joshuahogan/.planning/research/ARCHITECTURE.md` -- HIGH confidence
- v1.1 brainstorm at `/Users/joshuahogan/.claude/projects/-Users-joshuahogan/memory/brainstorm_homeowner_v1.1.md` -- HIGH confidence
- PROJECT.md requirements at `/Users/joshuahogan/.planning/PROJECT.md` -- HIGH confidence
- Motion (Framer Motion) animation patterns from existing `AnimatedSection.tsx` -- HIGH confidence
- Zustand v5 store pattern from existing `store.ts` -- HIGH confidence
