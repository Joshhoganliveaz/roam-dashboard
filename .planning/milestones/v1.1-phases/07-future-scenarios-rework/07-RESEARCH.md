# Phase 7: Future Scenarios Rework - Research

**Researched:** 2026-03-14
**Domain:** Financial projections, interactive UI, real-time calculation
**Confidence:** HIGH

## Summary

Phase 7 transforms the existing three scenario cards (Hold & Rent, Move-Up, Equity Play) from single-point-in-time snapshots into multi-horizon projection engines, adds a market rate slider that drives all scenarios, introduces rate lock advantage narrative, and creates a 4th combo scenario that chains the three into a unified wealth-building path.

The existing codebase is well-structured for this work. The calculation engine in `calculations.ts` already has all the foundational math (mortgage payments, remaining balance, HELOC, net proceeds, hold-and-rent cash flow). The current `ScenarioCard` component is a simple expandable card with title/hero-metric/details. The architecture decision to pre-compute condition variants server-side while keeping the market rate slider as the only client-side recalculation is already documented in STATE.md. The FRED API integration for current mortgage rate is already working.

**Primary recommendation:** Extend the calculation engine with multi-horizon projection functions, add a shared `MarketRateSlider` component above the scenario cards, refactor `ScenarioCard` to display tabbed 5/10/15yr projections, and build the combo scenario as a composition of existing calculation outputs.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SCEN-04 | All three scenarios show 5yr, 10yr, 15yr projection timelines | New projection functions in calculations.ts using existing `calcRemainingBalance`, `calcMortgagePayment`, appreciation compounding |
| SCEN-05 | One market rate slider affects all scenarios (refi for Hold&Rent, purchase for Move-Up, HELOC for Equity Play) | Client-side `useState` slider + `useMemo` recalculation; `currentMarketRate` already passed from server |
| SCEN-06 | Rate lock advantage narrative -- below-market rate highlights competitive advantage | Existing `calcRateLockSavings` function provides the math; conditional narrative logic |
| SCEN-07 | 4th combo scenario card chaining Hold&Rent + Equity Play + Move-Up math | Composition of existing calc functions; builds on SCEN-04 projection outputs |
| SCEN-08 | Enhanced Hold & Rent: projected cash flow, appreciation at 5/10/15yr, rent-vs-mortgage gap widening | New `calcHoldAndRentProjection(years)` function extending existing `calcHoldAndRent` |
| SCEN-09 | Enhanced Move-Up: education on sell-to-buy paths | Content/UI-driven (bridge loan, rent-back, simultaneous close); net proceeds projection at horizons |
| SCEN-10 | Enhanced Equity Play: HELOC with rate-adjusted costs from slider | Extend `calcEquityPlay` with HELOC payment calc at slider rate; show monthly HELOC cost |
</phase_requirements>

## Standard Stack

### Core (Already Installed)
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router, server components | Already in use |
| React | 19.2.3 | UI components | Already in use |
| TypeScript | 5.x | Type safety | Already in use |
| Tailwind CSS | 4.x | Styling | Already in use |
| Motion (Framer Motion) | 12.36.0 | Animations, expandable cards | Already in use |
| Recharts | 3.8.0 | Charts (if needed for projection viz) | Already in use |
| Vitest | 4.0.18 | Testing | Already in use |

### No New Dependencies Needed

All Phase 7 requirements can be met with the existing stack. The market rate slider is a native HTML range input styled with Tailwind. Multi-horizon projections are pure math functions. No charting library additions needed -- Recharts is already available if visual projections are desired, but the requirements specify "specific dollar amounts" which suggests data tables, not charts.

## Architecture Patterns

### Current Data Flow (Preserved)
```
Server Component (page.tsx)
  -> applyDefaults(record)
  -> computeHomeownerMetrics() x3 (per condition)
  -> getCurrentMortgageRate() from FRED API
  -> Pass to HomeownerPage client component

HomeownerPage (client)
  -> useState(condition) swaps pre-computed metrics
  -> NEW: useState(marketRate) drives slider recalculation
  -> useMemo computes scenario projections from metrics + marketRate
```

### Key Architecture Decision (from STATE.md)
> "Market rate slider is the only client-side recalculation; everything else swaps pre-computed sets"

This means the slider-driven projection math runs in `useMemo` on the client. This is correct -- projection calculations are lightweight (a few loops over 15 years max) and should not cause mobile lag.

### Pattern 1: Multi-Horizon Projection Functions
**What:** Pure functions that project financial scenarios at 5, 10, and 15-year horizons.
**When to use:** Every scenario card needs these.

```typescript
// New type for projection results
interface HorizonProjection {
  year: number; // 5, 10, or 15
  homeValue: number;
  mortgageBalance: number;
  equity: number;
  // Scenario-specific fields added per scenario type
}

// Hold & Rent specific
interface HoldAndRentProjection extends HorizonProjection {
  monthlyRent: number; // projected rent at that year
  monthlyMortgage: number; // fixed (or refi'd)
  monthlyCashFlow: number;
  annualCashFlow: number;
  cumulativeCashFlow: number;
  rentMortgageGap: number; // rent - mortgage (widening gap)
}

// Projection function pattern
function calcHoldAndRentProjections(
  currentValue: number,
  mortgageBalance: number,
  loanAmount: number,
  interestRate: number, // homeowner's locked rate
  marketRate: number, // from slider -- refi rate
  loanTerm: number,
  monthlyRent: number,
  appreciationRate: number,
  rentGrowthRate: number, // ~3% annual
  ownershipMonths: number,
  horizons: number[] // [5, 10, 15]
): HoldAndRentProjection[]
```

### Pattern 2: Market Rate Slider State
**What:** Single slider component that passes its value to all scenario calculations.
**When to use:** Placed above or at the top of the scenario section.

```typescript
// In HomeownerPage or ScenarioCards:
const [marketRate, setMarketRate] = useState(currentMarketRate);

// All projections recalculate via useMemo when marketRate changes
const holdAndRentProjections = useMemo(() =>
  calcHoldAndRentProjections(..., marketRate, ...),
  [metrics, marketRate]
);

const moveUpProjections = useMemo(() =>
  calcMoveUpProjections(..., marketRate, ...),
  [metrics, marketRate]
);

const equityPlayProjections = useMemo(() =>
  calcEquityPlayProjections(..., marketRate, ...),
  [metrics, marketRate]
);

const comboProjections = useMemo(() =>
  calcComboScenario(holdAndRentProjections, moveUpProjections, equityPlayProjections),
  [holdAndRentProjections, moveUpProjections, equityPlayProjections]
);
```

### Pattern 3: Combo Scenario Composition
**What:** 4th scenario that chains: (1) rent current home, (2) use HELOC for down payment on investment property, (3) benefit from both properties appreciating.
**When to use:** Built after the three base scenarios have projection functions.

```typescript
// Combo scenario: "Keep, rent, and buy another"
// Year 1: HELOC from current home -> down payment on property #2
// Ongoing: rental income from home #1 + appreciation on both properties
interface ComboScenarioProjection extends HorizonProjection {
  property1Value: number; // current home
  property2Value: number; // new investment
  totalPortfolioValue: number;
  helocDrawn: number;
  rentalIncome: number; // from home #1
  property2Mortgage: number;
  netMonthlyCashFlow: number;
  totalEquity: number; // across both properties
}
```

### Pattern 4: Rate Lock Advantage Narrative
**What:** Conditional display logic based on homeowner's rate vs market rate.
**When to use:** Within Hold & Rent scenario when homeowner's rate is below slider rate.

```typescript
const homeownerRate = record.interestRate ?? 0.065;
const isRateLocked = homeownerRate < marketRate;
const rateSavings = calcRateLockSavings(loanAmount, homeownerRate, marketRate, loanTerm);

// If locked below market: "Your 3.5% rate is saving you $X/month vs today's Y% rate"
// If at or above market: show refi option at slider rate
```

### Recommended File Changes
```
src/
├── lib/
│   ├── calculations.ts          # ADD: projection functions for all 4 scenarios
│   ├── calculations.test.ts     # ADD: tests for new projection functions
│   ├── types.ts                 # ADD: projection interfaces
│   └── narrative.ts             # ADD: scenario-specific SB7 copy maps
├── components/
│   └── homeowner/
│       ├── ScenarioCards.tsx     # REFACTOR: add slider, pass marketRate to cards
│       ├── ScenarioCard.tsx      # REFACTOR: support horizon tabs, richer detail rows
│       ├── MarketRateSlider.tsx  # NEW: range input with rate display
│       ├── HorizonTabs.tsx       # NEW: 5yr/10yr/15yr tab selector
│       ├── ComboScenario.tsx     # NEW: 4th card with chained projections
│       └── HomeownerPage.tsx     # MODIFY: pass marketRate, additional data to ScenarioCards
```

### Anti-Patterns to Avoid
- **Server-side slider computation:** Do NOT pre-compute projections for every possible slider value. The slider is continuous -- compute client-side in useMemo.
- **Heavy render on slider drag:** Use `useDeferredValue` or throttle if slider drag causes jank on mobile. Profile first -- the math is likely fast enough without optimization.
- **Separate state stores:** Do NOT add Zustand for slider state. `useState` in the ScenarioCards parent is sufficient. The admin store is for the admin form only.
- **Breaking condition variant architecture:** The market rate slider is INDEPENDENT of the condition picker. Condition swaps pre-computed metrics; slider recalculates projections from those metrics. Do not conflate the two.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Mortgage amortization | Custom payment formula | Existing `calcMortgagePayment` + `calcRemainingBalance` | Already tested, handles edge cases (zero rate, negative amounts) |
| Rate lock savings | New comparison logic | Existing `calcRateLockSavings` | Reuses `calcMortgagePayment` internally for consistency |
| Slider debounce | Custom setTimeout wrapper | React 19 `useDeferredValue` or `useTransition` | Built-in, optimized for concurrent rendering |
| Currency formatting | `toLocaleString` calls | Existing `formatCurrency` | Already handles zero decimals, negatives |

## Common Pitfalls

### Pitfall 1: Rent Growth Assumption
**What goes wrong:** Hold & Rent projections show rent staying flat, making the gap between rent and mortgage unrealistically narrow at 15 years.
**Why it happens:** Current `calcHoldAndRent` uses static rent. Phoenix rents grow ~3-4%/year.
**How to avoid:** Add a `rentGrowthRate` constant to DEFAULTS (suggest 0.03) and compound rent in projection loops. This is what makes the "rent-vs-mortgage gap widening" requirement (SCEN-08) work.
**Warning signs:** If 15-year rent projection is close to current rent, the growth rate is missing.

### Pitfall 2: Slider Lag on Mobile
**What goes wrong:** Dragging the rate slider causes visible jank as all four scenarios recalculate and re-render.
**Why it happens:** Four projection calculations + four card re-renders on every input event.
**How to avoid:** (1) Profile first -- the math is likely < 1ms total. (2) If needed, use `useDeferredValue(marketRate)` so the slider thumb tracks immediately while projections update on next idle frame. (3) Avoid unnecessary React re-renders by memoizing scenario card components.
**Warning signs:** Stuttering slider on older iPhones.

### Pitfall 3: Combo Scenario Double-Counting
**What goes wrong:** The combo scenario adds up equity from both properties but forgets to subtract the HELOC balance, overstating total wealth.
**Why it happens:** HELOC drawn from property 1 is a liability. If it funds property 2's down payment, the HELOC balance must be tracked alongside property 2's mortgage.
**How to avoid:** Track HELOC as a separate liability line item. Total equity = (home1 value - home1 mortgage - HELOC balance) + (home2 value - home2 mortgage).
**Warning signs:** Combo scenario total wealth exceeds sum of individual scenario totals.

### Pitfall 4: Rate Lock Narrative Conflict with Slider
**What goes wrong:** Homeowner has a 3.5% rate. Slider shows 7%. Rate lock callout says "save $X/month." But when user slides to 3.5%, the narrative breaks -- savings become $0.
**Why it happens:** Rate lock narrative should react to slider value, not just initial market rate.
**How to avoid:** Rate lock advantage = homeowner's locked rate vs slider rate. When slider equals or goes below homeowner's rate, hide the advantage callout. Use `marketRate` from slider, not `currentMarketRate` from server.
**Warning signs:** Rate lock callout showing when slider is below homeowner's rate.

### Pitfall 5: Move-Up Scenario Rate Confusion
**What goes wrong:** Move-Up shows purchasing power at the homeowner's existing rate instead of the slider rate.
**Why it happens:** New purchase mortgage should use the market rate from the slider, not the homeowner's locked rate.
**How to avoid:** Move-Up mortgage payment calculation uses `marketRate` (slider), while Hold & Rent refi uses `marketRate`. Different applications of the same slider value.
**Warning signs:** Move-Up purchasing power not changing when slider moves.

### Pitfall 6: Appreciation Rate Mismatch
**What goes wrong:** Projection uses annualized historical appreciation for the subject property as the forward projection rate, which can be wildly off (e.g., 20% annualized for a 1-year owner).
**Why it happens:** Conflating backward-looking CAGR with forward assumptions.
**How to avoid:** Use `DEFAULTS.appreciationRate` (4%) for all forward projections. The historical rate is for the retrospective narrative, not predictions.
**Warning signs:** 15-year projected values that seem implausibly high or low.

## Code Examples

### Multi-Horizon Hold & Rent Projection
```typescript
// Source: Pattern derived from existing calcHoldAndRent + calcRemainingBalance
const HORIZONS = [5, 10, 15] as const;
const RENT_GROWTH_RATE = 0.03; // 3% annual rent growth

export function calcHoldAndRentProjections(
  currentValue: number,
  monthlyRent: number,
  monthlyMortgage: number,
  appreciationRate: number,
  refiRate: number, // from market rate slider
  loanAmount: number,
  interestRate: number, // homeowner's current rate
  loanTerm: number,
  ownershipMonths: number
): HoldAndRentProjection[] {
  // Determine if refi makes sense (slider rate < homeowner rate)
  const useRefi = refiRate < interestRate;
  const effectivePayment = useRefi
    ? calcMortgagePayment(/* remaining balance */, refiRate, 30)
    : monthlyMortgage;

  return HORIZONS.map((year) => {
    const projectedValue = currentValue * Math.pow(1 + appreciationRate, year);
    const projectedRent = monthlyRent * Math.pow(1 + RENT_GROWTH_RATE, year);
    const futureBalance = calcRemainingBalance(
      loanAmount, interestRate, loanTerm, ownershipMonths + year * 12
    );
    const cashFlow = projectedRent - effectivePayment;

    return {
      year,
      homeValue: projectedValue,
      mortgageBalance: futureBalance,
      equity: projectedValue - futureBalance,
      monthlyRent: projectedRent,
      monthlyMortgage: effectivePayment,
      monthlyCashFlow: cashFlow,
      annualCashFlow: cashFlow * 12,
      rentMortgageGap: projectedRent - effectivePayment,
    };
  });
}
```

### Market Rate Slider Component
```typescript
// Source: Native HTML range input, styled with Tailwind
interface MarketRateSliderProps {
  value: number; // decimal (0.065)
  onChange: (rate: number) => void;
  homeownerRate: number; // for rate lock comparison
}

export function MarketRateSlider({ value, onChange, homeownerRate }: MarketRateSliderProps) {
  const displayRate = (value * 100).toFixed(2);
  const isLocked = homeownerRate < value;

  return (
    <div className="mb-8">
      <label className="font-heading text-sm text-charcoal/60 block mb-2">
        Market Rate: {displayRate}%
      </label>
      <input
        type="range"
        min={0.03}
        max={0.10}
        step={0.0025}
        value={value}
        onChange={(e) => onChange(parseFloat(e.target.value))}
        className="w-full accent-olive"
      />
      {isLocked && (
        <p className="text-olive text-sm mt-2">
          Your {(homeownerRate * 100).toFixed(2)}% rate saves you money vs {displayRate}%
        </p>
      )}
    </div>
  );
}
```

### Horizon Tabs Pattern
```typescript
// Source: Simple tab UI for 5/10/15yr selection within each card
const [activeHorizon, setActiveHorizon] = useState<5 | 10 | 15>(5);

<div className="flex gap-2 mb-4">
  {[5, 10, 15].map((yr) => (
    <button
      key={yr}
      onClick={() => setActiveHorizon(yr as 5 | 10 | 15)}
      className={`px-3 py-1 rounded-full text-sm font-body ${
        activeHorizon === yr
          ? "bg-olive text-white"
          : "bg-charcoal/5 text-charcoal/60"
      }`}
    >
      {yr}yr
    </button>
  ))}
</div>
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single snapshot scenario | Multi-horizon projections | This phase | Shows compound growth, makes wealth-building tangible |
| Static rate assumptions | Interactive rate slider | This phase | Homeowner explores "what if" scenarios |
| Three independent scenarios | Chained combo scenario | This phase | Unified wealth-building narrative |
| Refi suggestion regardless | Rate lock advantage callout | This phase | Acknowledges low-rate holders, doesn't insult their intelligence |

## Open Questions

1. **Combo scenario HELOC vs sale proceeds**
   - What we know: STATE.md notes "Combo scenario requirements need Josh's input (HELOC vs sale proceeds for down payment)"
   - What's unclear: Does the combo scenario assume the homeowner takes a HELOC for the 2nd property's down payment (keeping home 1) or sells home 1? The name "Keep, rent, and buy another" strongly implies HELOC, not sale.
   - Recommendation: Build assuming HELOC funds the down payment on property 2, homeowner keeps and rents home 1. This matches the requirement name and chains Hold & Rent + Equity Play logically.

2. **Rent growth rate assumption**
   - What we know: Phoenix metro rent growth has averaged ~3-5% annually over the past decade.
   - What's unclear: Should this be a fixed constant or configurable?
   - Recommendation: Use `DEFAULTS.rentGrowthRate = 0.03` (conservative 3%). Not user-facing -- the slider is enough interactivity. Can be adjusted per-market later.

3. **Move-Up "education" scope (SCEN-09)**
   - What we know: Requirement says "education on sell-to-buy paths (sell first, bridge loan, rent-back, simultaneous close)"
   - What's unclear: This is content, not calculation. How much detail?
   - Recommendation: Four brief descriptions (2-3 sentences each) in expandable detail rows, not a separate page. Keep it light -- the point is awareness, not a tutorial.

4. **Slider range bounds**
   - What we know: FRED returns current 30yr rate (~6.5-7% as of early 2026).
   - What's unclear: Min/max slider range?
   - Recommendation: 3.0% to 10.0% with 0.25% steps. Covers historical lows to highs, gives enough range to explore.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.0.18 + Testing Library React 16.3.2 |
| Config file | `~/Projects/homeowner-journey-map/vitest.config.ts` |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| SCEN-04 | Projection functions return correct values at 5/10/15yr | unit | `npx vitest run src/lib/calculations.test.ts -t "projection"` | Partial (file exists, tests need adding) |
| SCEN-05 | Slider rate changes affect all scenario outputs differently | unit | `npx vitest run src/lib/calculations.test.ts -t "market rate"` | No -- Wave 0 |
| SCEN-06 | Rate lock narrative shows/hides based on rate comparison | unit+component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "rate lock"` | Partial (file exists, tests need adding) |
| SCEN-07 | Combo scenario correctly chains three scenario outputs | unit | `npx vitest run src/lib/calculations.test.ts -t "combo"` | No -- Wave 0 |
| SCEN-08 | Hold & Rent projections include rent growth and cash flow | unit | `npx vitest run src/lib/calculations.test.ts -t "holdAndRent projection"` | No -- Wave 0 |
| SCEN-09 | Move-Up shows sell-to-buy path descriptions | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "move-up"` | Partial |
| SCEN-10 | Equity Play HELOC cost responds to slider rate | unit | `npx vitest run src/lib/calculations.test.ts -t "equityPlay"` | Partial |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Per wave merge:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] Add projection function tests to `src/lib/calculations.test.ts` -- covers SCEN-04, SCEN-05, SCEN-08, SCEN-10
- [ ] Add combo scenario tests to `src/lib/calculations.test.ts` -- covers SCEN-07
- [ ] Add rate lock conditional tests to `src/components/homeowner/ScenarioCards.test.tsx` -- covers SCEN-06

## Sources

### Primary (HIGH confidence)
- **Project codebase** -- `calculations.ts`, `types.ts`, `ScenarioCards.tsx`, `ScenarioCard.tsx`, `HomeownerPage.tsx`, `market-rate.ts`, `defaults.ts` -- all read directly
- **STATE.md** -- documented architecture decisions including "market rate slider is the only client-side recalculation"
- **REQUIREMENTS.md** -- exact requirement text for SCEN-04 through SCEN-10

### Secondary (MEDIUM confidence)
- **React 19 useDeferredValue** -- standard pattern for slider performance; available in React 19.2.3 per package.json
- **Phoenix rent growth** -- ~3-4% annual average is widely reported; conservative 3% used

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no new dependencies, all existing tools sufficient
- Architecture: HIGH -- existing patterns (condition variants, server-side pre-compute, client-side useMemo) extend naturally to this work
- Pitfalls: HIGH -- identified from direct code reading and understanding of the rate/mortgage math
- Calculation correctness: HIGH -- existing test suite is thorough; new tests follow same patterns

**Research date:** 2026-03-14
**Valid until:** 2026-04-14 (stable domain, no fast-moving dependencies)
