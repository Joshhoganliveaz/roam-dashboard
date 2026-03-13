# Architecture Patterns: v1.1 Enhanced Intelligence

**Domain:** Interactive features on server-rendered homeowner pages
**Researched:** 2026-03-13

## Key Architectural Change: Server-Rendered Page with Client-Side Interactivity

v1.0 pages are fully server-rendered -- no client-side state. v1.1 introduces homeowner-driven interactions (condition picker, projection slider, market rate slider) that require client-side state and recalculation.

### Component Boundary: Server vs Client

```
Server Component (page.tsx)
  |-- Fetches homeowner data from Supabase
  |-- Fetches cromford_data for the homeowner's city
  |-- Computes initial metrics with default condition
  |-- Passes data as props to:
      |
      Client Component (InteractiveJourneyPage)
        |-- Receives: homeowner record, value range, cromford data, initial metrics
        |-- Manages: selectedCondition, projectionYears, marketRateOverride (local state)
        |-- Recalculates: metrics via useMemo when selections change
        |-- Renders: all sections with cascading animations
```

### Data Flow

```
Supabase (homeowners table + cromford_data table)
  |
  v
Server Component: fetch + initial compute
  |
  v
Client Component Props: { record, valueRange, cromfordData, initialMetrics }
  |
  v
Homeowner Interaction: selects condition / adjusts slider
  |
  v
Local State Update (useState, NOT Zustand)
  |
  v
useMemo: recompute metrics from new inputs
  |
  v
React Re-render: all sections update with new numbers
  |
  v
Motion: animates the number transitions (layout + AnimatePresence)
```

### Why useState Instead of Zustand for Homeowner Page

The admin form uses Zustand because it has complex state (many fields, CSV import, slug generation, persistence concerns). The homeowner page has exactly three interactive values:

1. `selectedCondition: 'original' | 'updated' | 'remodeled'`
2. `projectionYears: 5 | 10 | 15`
3. `marketRateOverride: number | null`

This is 3 state values. Zustand adds unnecessary indirection. Use `useState` in the `InteractiveJourneyPage` component and pass derived metrics down as props.

If later features add more interactive state (e.g., multiple independent sliders, saved preferences), then consider a Zustand store. But for v1.1, keep it simple.

## Patterns to Follow

### Pattern 1: Pure Calculation Engine with Parameterized Projections

**What:** All financial math lives in `calculations.ts` as pure, tested functions. v1.1 adds projection functions that accept time horizon and rate parameters.

**When:** Every new calculation for scenarios, projections, or comparisons.

**Example:**
```typescript
// Extend existing pattern: pure function, no side effects
export function calcProjectedScenarios(
  input: HomeownerInput,
  years: number,
  appreciationRate: number,
  metrics: HomeownerMetrics
): ProjectedScenarios {
  // Uses existing calc functions at different time horizons
  const projectedValue = input.currentValue * Math.pow(1 + appreciationRate, years);
  const projectedBalance = calcRemainingBalance(
    input.loanAmount, input.interestRate, input.loanTerm,
    metrics.ownershipMonths + (years * 12)
  );
  // ... build all scenario projections
}
```

### Pattern 2: Cromford Data as City-Level Shared Resource

**What:** Cromford data is stored per-city, not per-homeowner. The page fetches cromford_data WHERE city = homeowner.city.

**When:** Rendering the appreciation chart, computing city-average comparison.

**Example:**
```typescript
// Server component fetch
const cromfordData = await supabase
  .from('cromford_data')
  .select('year, quarter, avg_price_per_sqft')
  .eq('city', homeowner.city)
  .eq('state', homeowner.state)
  .order('year', { ascending: true });
```

### Pattern 3: Staggered Animation on State Change

**What:** When the condition picker changes, numbers throughout the page update with a cascading ripple effect -- top sections first, then flowing downward.

**When:** Any time `selectedCondition` changes.

**Example:**
```typescript
// Each section gets a stagger delay based on its position
const sectionVariants = {
  update: (i: number) => ({
    opacity: [1, 0.7, 1],
    transition: { delay: i * 0.08, duration: 0.4 }
  })
};

// In each section component:
<motion.div
  variants={sectionVariants}
  custom={sectionIndex}
  animate="update"
  key={selectedCondition}  // re-trigger on condition change
>
```

## Anti-Patterns to Avoid

### Anti-Pattern 1: Storing Computed Metrics in State

**What:** Putting the output of `computeHomeownerMetrics()` into Zustand or useState.
**Why bad:** Creates stale data risk. If any input changes but the metrics are not re-derived, the UI shows incorrect numbers. Also doubles memory usage.
**Instead:** Use `useMemo` to derive metrics from inputs. React guarantees re-computation when dependencies change.

### Anti-Pattern 2: Fetching Cromford Data Client-Side

**What:** Making the homeowner's browser call Supabase for cromford_data.
**Why bad:** Exposes Supabase anon key in client bundle, adds loading state to a read-only page, unnecessary network request.
**Instead:** Fetch in the server component and pass as props. The data is the same for every visitor viewing the same homeowner's page.

### Anti-Pattern 3: One Giant Calculation Function

**What:** Adding all v1.1 projections into `computeHomeownerMetrics()`.
**Why bad:** The function is already 90 lines. Adding 5/10/15yr projections, combo scenario, rate lock math would make it 200+ lines and hard to test.
**Instead:** Keep `computeHomeownerMetrics()` for current-state metrics. Create a separate `computeProjections()` function for forward-looking scenarios. Compose them in the component.

### Anti-Pattern 4: Re-running ALL Calculations When Only Projection Years Change

**What:** Recalculating equity, mortgage balance, S&P comparison, etc. when the homeowner toggles from 5yr to 10yr projections.
**Why bad:** The current-state metrics (equity, appreciation, earnings) do not change when projection years change. Only the future scenarios change.
**Instead:** Split the calculation into two stages:
1. `currentMetrics = computeHomeownerMetrics(input)` -- runs when condition changes
2. `projections = computeProjections(input, currentMetrics, years, rate)` -- runs when years/rate changes

This way, changing the projection slider only recalculates projections, not the entire page.

## Scalability Considerations

| Concern | Current (200 pages) | At 1,000 pages | At 5,000 pages |
|---------|---------------------|-----------------|-----------------|
| Cromford data volume | ~20 cities x 15 years = 300 rows | Same | Same (city-level, not homeowner-level) |
| Value range storage | 3 extra numeric columns per homeowner | Trivial | Trivial |
| Client-side recalculation | < 1ms per computation | Same (per-page, not aggregate) | Same |
| Supabase reads | 1 homeowner + 1 cromford query per page load | Same (ISR caches the result) | Same |

No scalability concerns for v1.1 features at any realistic volume.

## Sources

- Existing store.ts pattern: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/store.ts
- Existing calculations.ts pattern: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/calculations.ts
- STR Analyzer useMemo pattern (from CLAUDE.md): "Changes to any field trigger useMemo-based recalculation"
- Motion animation docs: https://motion.dev/docs (HIGH confidence)
