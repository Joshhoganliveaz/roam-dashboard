# Phase 9: Scenario Cards Rework - Research

**Researched:** 2026-03-13
**Domain:** Financial projections, interactive mortgage calculators, effective interest rate calculation, component architecture
**Confidence:** HIGH

## Summary

Phase 9 replaces the existing four scenario cards (Hold & Rent, Move Up, Equity Play, Keep-Rent-Buy Another) with four new scenarios that match real homeowner decision paths: Stay & Build Wealth, Sell & Move Up, Stay & Invest, and Move & Keep as Rental. This phase also introduces dynamic date-based horizons (actual dates from page view date instead of generic "5yr/10yr/15yr"), a "Now" horizon, mini mortgage calculators inside two cards, and the effective interest rate concept as the hero metric for Move & Keep as Rental.

The existing codebase provides a strong foundation. The calculation engine (`calculations.ts`) already has all core financial math: `calcMortgagePayment`, `calcRemainingBalance`, `calcNetProceeds`, `calcEquityPlay`, and multi-horizon projection functions. The current `ScenarioCard` component supports horizon tabs with expandable details and children prop for custom content. The `MarketRateSlider` is already implemented with 3-10% range and 0.25% steps. The `HorizonTabs` component and all animation infrastructure (Motion, AnimatePresence) are in place. The key new challenges are: (1) the effective interest rate calculation requires a numerical solver (bisection method), (2) horizon tabs need to switch from relative years to dynamic dates, (3) mini mortgage calculators need their own local state for rate overrides and down payment selection, and (4) net proceeds moves from a standalone section into the Sell & Move Up card.

**Primary recommendation:** Refactor the four scenario cards with new calculation functions, convert HorizonTabs from relative years to date-based horizons with a "Now" tab, add a `MiniMortgageCalculator` component reused across Sell & Move Up and Move & Keep as Rental, and implement `calcEffectiveInterestRate` using bisection method on the existing `calcMortgagePayment` function.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Four new scenarios replace the old four: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental
- Stay & Build Wealth: do nothing, let appreciation and equity grow. Shows home value, mortgage balance, equity at each horizon. Narrative: "Your home is already working for you"
- Sell & Move Up: shows net proceeds with "cost to sell*" label (title fees & broker fees), mini mortgage calculator with 30yr fixed, adjustable rate, 5/10/15/20% down payment presets (buttons not free-form), new home price input. Commission 6%, seller closing costs 1%. CTA: lender connect
- Stay & Invest: shows HELOC available, monthly HELOC cost (interest-only at market rate), investment property framing, tax advantages callout (depreciation for W-2 earners). CTAs: STR Analyzer link + Josh & Jacqui investing callout
- Move & Keep as Rental: current home becomes rental, buy new primary. Shows rental income (from monthlyRentEstimate admin data), cash flow, effective interest rate as HERO METRIC. Mini mortgage calculator. HELOC for down payment is optional
- Effective interest rate: new mortgage payment minus rental cash flow = out-of-pocket. The rate that produces that out-of-pocket payment on the same loan is the effective rate. Example: $3,200 - $800 = $2,400 out-of-pocket, rate that makes $3,200 equal $2,400 is ~4.2% vs actual 6.5%
- Market rate slider: one slider above all four cards, 3% to 10% range, defaults to today's actual market rate. Individual mini mortgage calculators can override independently
- Dynamic date horizons: calculated from page view date. Four tabs: current date, +5yr, +10yr, +15yr. Label format: Mar '26 | Mar '31 | Mar '36 | Mar '41 (3-letter month + 2-digit year). Year-only fallback if too tight on mobile. Build month abbreviation first
- Rental income from existing admin data (monthlyRentEstimate field), NOT adjusted by homeowner
- "Purchasing power" concept rejected -- depends on DTI which only a lender can determine
- Down payment presets as buttons (5% | 10% | 15% | 20%), not slider or free input

### Claude's Discretion
- Exact projection calculation implementations (reuse/extend existing calc functions)
- Component architecture for the four new scenario cards
- Styling details within branded guidelines
- How the effective interest rate is calculated (reverse mortgage payment formula)
- How to handle edge cases (zero mortgage, negative equity, etc.)

### Deferred Ideas (OUT OF SCOPE)
- Zillow Zestimate API for rent estimate auto-fill in admin -- future admin enhancement
- DTI-based affordability calculation -- that's the lender's job, the CTA drives them there
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| SCNR-01 | Four scenario cards: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental | Replace existing 4 cards (Hold & Rent, Move Up, Equity Play, Combo). New calc functions for each, reusing existing `calcRemainingBalance`, `calcMortgagePayment`, `calcNetProceeds`, `calcEquityPlay` |
| SCNR-02 | Dynamic date horizons from page view date (e.g., Mar '26 to Mar '41), year-only fallback on mobile | Refactor `HorizonTabs` to accept Date-based horizons instead of numeric years. Use `date-fns` format() for "MMM 'yy" labels. CSS media query or ResizeObserver for mobile fallback |
| SCNR-03 | One market rate slider above all cards; individual card calculators can override | Existing `MarketRateSlider` already works. Mini calculators get their own local `useState` for rate, initialized from market rate but independently adjustable |
| SCNR-04 | Stay & Build Wealth shows home value, mortgage balance, equity at each horizon | Simplest card -- reuse `calcRemainingBalance` for mortgage paydown, compound appreciation at DEFAULTS.appreciationRate. No new calc function needed, inline useMemo |
| SCNR-05 | Sell & Move Up shows net proceeds, mini mortgage calculator (30yr, adjustable rate, 5/10/15/20% down presets) | Reuse `calcNetProceeds` with updated rates (6% commission + 1% closing = 7% total). Mini calculator: `MiniMortgageCalculator` component with local state for newHomePrice, downPaymentPercent, rate |
| SCNR-06 | Sell & Move Up uses 6% commission + 1% seller closing costs | Update `DEFAULTS.agentCommissionRate` from 0.05 to 0.06, `DEFAULTS.closingCostRate` from 0.02 to 0.01, OR pass these rates directly to the card. Net effect: 7% total (same as current 5%+2% total) |
| SCNR-07 | Stay & Invest shows HELOC available, monthly cost, investment framing, tax advantages, CTAs | Reuse `calcEquityPlayProjections` for HELOC math. Tax callout is static SB7 narrative content. CTAs are links/buttons |
| SCNR-08 | Move & Keep as Rental shows rental income, cash flow, effective interest rate, mini mortgage calculator | New `calcEffectiveInterestRate` function using bisection method. Uses `monthlyRentEstimate` from record. Mini calculator reuses `MiniMortgageCalculator` |
| SCNR-09 | Effective interest rate calculation: rate producing out-of-pocket payment on same loan | Bisection search on `calcMortgagePayment`: find rate r where calcMortgagePayment(loan, r, 30) = targetPayment. Target = newMortgagePayment - rentalCashFlow |
| SCNR-10 | Each scenario has contextual CTAs (lender connect, STR Analyzer, Josh & Jacqui) | SB7-aligned inline CTAs using tel/sms links (matching existing CTA pattern). STR Analyzer external link is an exception to the "no external links" rule because it is Josh's own tool |
</phase_requirements>

## Standard Stack

### Core (Already Installed -- No New Dependencies)
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router, server components | Already in use |
| React | 19.2.3 | UI components, useState for local calc state | Already in use |
| TypeScript | 5.x | Type safety for projection types | Already in use |
| Tailwind CSS | 4.x | Styling, responsive breakpoints for mobile fallback | Already in use |
| Motion | 12.36.0 | AnimatePresence for expandable cards | Already in use |
| date-fns | 4.1.0 | Date arithmetic for horizon labels (addYears, format) | Already in use |
| Vitest | 4.0.18 | Testing calculation functions | Already in use |

### No New Dependencies Needed

All Phase 9 requirements are met with the existing stack. The mini mortgage calculator is a pure React component with local state. The effective interest rate calculation is a pure math function. Date horizon labels use `date-fns` format. No new charting or UI libraries required.

## Architecture Patterns

### Current Data Flow (Preserved)
```
Server Component (page.tsx)
  -> fetches record + currentMarketRate (FRED API)
  -> pre-computes conditionVariants (3 condition sets)
  -> passes to HomeownerPage (client component)

HomeownerPage
  -> useState for condition selection
  -> passes record, metrics, currentMarketRate to ScenarioCards

ScenarioCards
  -> useState for marketRate (slider)
  -> useMemo for each scenario's projections
  -> renders MarketRateSlider + 4 scenario cards
```

### New Component Architecture
```
ScenarioCards.tsx (refactored)
  ├── MarketRateSlider (existing, unchanged)
  ├── HorizonTabs (refactored: date-based instead of year-based)
  ├── StayAndBuildCard.tsx (new)
  │     └── Uses HorizonTabs internally
  ├── SellAndMoveUpCard.tsx (new)
  │     ├── Uses HorizonTabs internally
  │     ├── Net proceeds breakdown
  │     └── MiniMortgageCalculator (new, shared component)
  ├── StayAndInvestCard.tsx (new)
  │     ├── Uses HorizonTabs internally
  │     └── Tax advantages callout (static narrative)
  └── MoveAndKeepAsRentalCard.tsx (new)
        ├── Uses HorizonTabs internally
        ├── Effective interest rate hero metric
        └── MiniMortgageCalculator (reused)
```

### Pattern 1: Shared MiniMortgageCalculator Component
**What:** A reusable interactive calculator embedded within scenario cards
**When to use:** Inside Sell & Move Up and Move & Keep as Rental cards
**Key design decisions:**
- Component owns its own local state (rate override, down payment %, new home price)
- Rate defaults to parent's marketRate but can be independently adjusted
- Down payment uses button presets (5/10/15/20%) not free input
- Outputs P&I monthly payment via existing `calcMortgagePayment`
- Does NOT lift state to parent -- each instance is independent

```typescript
interface MiniMortgageCalculatorProps {
  marketRate: number;           // Default rate from slider
  defaultHomePrice?: number;    // Optional pre-fill
  onPaymentChange?: (payment: number, loanAmount: number, rate: number) => void;
}

function MiniMortgageCalculator({ marketRate, defaultHomePrice, onPaymentChange }: MiniMortgageCalculatorProps) {
  const [homePrice, setHomePrice] = useState(defaultHomePrice ?? 0);
  const [downPct, setDownPct] = useState(0.20);
  const [rate, setRate] = useState(marketRate);

  const loanAmount = homePrice * (1 - downPct);
  const monthlyPayment = calcMortgagePayment(loanAmount, rate, 30);

  // Notify parent when payment changes (for effective rate calc)
  useEffect(() => {
    onPaymentChange?.(monthlyPayment, loanAmount, rate);
  }, [monthlyPayment, loanAmount, rate]);

  return (/* UI with home price input, down payment buttons, rate slider, payment output */);
}
```

### Pattern 2: Dynamic Date Horizons
**What:** HorizonTabs shows actual dates instead of relative years
**When to use:** All four scenario cards
**Implementation:**

```typescript
import { addYears, format } from "date-fns";

// Generate horizon dates from current date
function getHorizonDates(now: Date): { label: string; shortLabel: string; years: number; date: Date }[] {
  return [
    {
      label: format(now, "MMM ''yy"),        // "Mar '26"
      shortLabel: format(now, "yyyy"),        // "2026"
      years: 0,
      date: now
    },
    ...([5, 10, 15] as const).map(y => {
      const d = addYears(now, y);
      return {
        label: format(d, "MMM ''yy"),         // "Mar '31"
        shortLabel: format(d, "yyyy"),        // "2031"
        years: y,
        date: d,
      };
    }),
  ];
}
```

**Mobile fallback:** Use a CSS media query or `useMediaQuery` hook to switch between `label` and `shortLabel`. Start with month abbreviation format, test on mobile, fall back to year-only if space is tight.

### Pattern 3: Effective Interest Rate Calculation (Bisection Method)
**What:** Given a target monthly payment and loan amount, find the interest rate that produces that payment
**When to use:** Move & Keep as Rental card -- the hero metric
**Why bisection over Newton-Raphson:** Bisection is simpler, guaranteed to converge within a bounded interval [0%, 30%], and the mortgage payment function is monotonically increasing with rate. No derivative needed. Converges in ~20 iterations to 6+ decimal places.

```typescript
/**
 * Find the annual interest rate that produces the target monthly payment
 * for a given loan amount and term.
 * Uses bisection method on calcMortgagePayment.
 *
 * @param loanAmount - Principal loan amount
 * @param targetPayment - Desired monthly payment
 * @param termYears - Loan term in years (default 30)
 * @param tolerance - Convergence tolerance (default $0.01)
 * @param maxIterations - Safety limit (default 100)
 * @returns Annual interest rate as decimal, or null if no solution
 */
export function calcEffectiveInterestRate(
  loanAmount: number,
  targetPayment: number,
  termYears: number = 30,
  tolerance: number = 0.01,
  maxIterations: number = 100
): number | null {
  if (loanAmount <= 0 || targetPayment <= 0) return null;

  // Edge case: if targetPayment >= zero-interest payment, rate is 0 or negative
  const zeroRatePayment = loanAmount / (termYears * 12);
  if (targetPayment <= zeroRatePayment) return 0;

  let low = 0.0001;  // 0.01%
  let high = 0.30;   // 30%

  for (let i = 0; i < maxIterations; i++) {
    const mid = (low + high) / 2;
    const payment = calcMortgagePayment(loanAmount, mid, termYears);

    if (Math.abs(payment - targetPayment) < tolerance) {
      return mid;
    }

    if (payment < targetPayment) {
      low = mid;
    } else {
      high = mid;
    }
  }

  return (low + high) / 2; // Best approximation
}
```

**Effective rate flow for Move & Keep as Rental:**
1. User sets new home price and down payment in mini mortgage calculator
2. `newMortgagePayment = calcMortgagePayment(newLoanAmount, rate, 30)`
3. `rentalCashFlow = monthlyRentEstimate - oldMortgagePayment` (or just monthlyRentEstimate if considering gross rental income)
4. `outOfPocket = newMortgagePayment - rentalCashFlow` (clamped to >= 0)
5. `effectiveRate = calcEffectiveInterestRate(newLoanAmount, outOfPocket, 30)`
6. Display: "Your effective rate: X.XX%" as the hero metric

### Anti-Patterns to Avoid
- **Lifting mini calculator state to ScenarioCards:** Each mini calculator instance should own its own state. Lifting creates unnecessary coupling and re-renders.
- **Using purchasing power concept:** Josh explicitly rejected this -- it depends on DTI which only a lender can determine. Show net proceeds and monthly payment, not "what you could buy."
- **Free-form down payment input:** Down payment MUST be button presets (5/10/15/20%), not a text input or slider. Clean, simple, no edge cases.
- **Linking to external sites for comps:** No Zillow/Redfin links. The STR Analyzer link is the only external link allowed (Josh's own tool).

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Mortgage payment calculation | Custom amortization formula | Existing `calcMortgagePayment` | Already tested, handles edge cases (zero rate, zero loan, negative loan) |
| Remaining balance at horizon | Manual amortization schedule | Existing `calcRemainingBalance` | Handles all edge cases, clamped to >= 0 |
| Net proceeds | Custom selling cost calc | Existing `calcNetProceeds` | Already has commission + closing cost breakdown |
| Date formatting | Manual month/year string manipulation | `date-fns` format() | Already a dependency, handles localization |
| Date arithmetic | Manual year addition | `date-fns` addYears() | Already a dependency, handles leap years |
| Interest rate solver | Custom root-finding from scratch | Bisection on existing `calcMortgagePayment` | Simple, guaranteed convergence, reuses existing tested function |
| HELOC calculation | New HELOC function | Existing `calcEquityPlayProjections` | Already computes HELOC available, monthly payment, annual cost |

**Key insight:** The existing calculation engine covers 80%+ of the math needed. The only truly new calculation is `calcEffectiveInterestRate`, and even that is a thin wrapper around `calcMortgagePayment`.

## Common Pitfalls

### Pitfall 1: Commission Rate Mismatch
**What goes wrong:** CONTEXT.md specifies 6% commission + 1% seller closing costs, but current DEFAULTS use 5% commission + 2% closing costs.
**Why it happens:** The total is 7% either way, but the labeling changes.
**How to avoid:** For the Sell & Move Up card, pass `commissionRate: 0.06` and `closingCostRate: 0.01` directly rather than changing DEFAULTS globally (which would affect other parts of the app). The net proceeds breakdown label says "cost to sell*" with "(title fees & broker fees)" -- not "agent commission" and "closing costs" separately.
**Warning signs:** Labels in the card don't match the CONTEXT.md specification.

### Pitfall 2: Effective Interest Rate Edge Cases
**What goes wrong:** `calcEffectiveInterestRate` returns null, NaN, or nonsensical values.
**Why it happens:** Several edge cases: (a) rental income exceeds new mortgage (out-of-pocket is 0 or negative -- effective rate is 0% or "free"), (b) no new home price entered yet (loan amount is 0), (c) rental income is null/zero in admin data.
**How to avoid:**
- If outOfPocket <= 0, display "0.00% -- rental income covers your payment" instead of computing
- If loanAmount <= 0, don't show effective rate
- If monthlyRentEstimate is null, use DEFAULTS.rentYieldMonthly fallback (already handled by ScenarioCards)
**Warning signs:** NaN displayed in the hero metric, or effective rate higher than actual rate.

### Pitfall 3: Horizon Tab "Now" Data Confusion
**What goes wrong:** The "Now" tab (year 0) shows projected values instead of current actual values.
**Why it happens:** Current projection functions start at year 5 (PROJECTION_HORIZONS = [5, 10, 15]). Adding year 0 means using current actual values, not projected.
**How to avoid:** For the "Now" tab, use current record data directly (currentValue, currentBalance, current equity) -- don't run it through the appreciation/projection logic. Only years 5/10/15 use projected values.
**Warning signs:** "Now" values don't match the numbers shown in the equity/value sections above.

### Pitfall 4: Rate Override Not Independent
**What goes wrong:** Changing the rate in one mini mortgage calculator affects the other card's calculator.
**Why it happens:** If the mini calculator rate is stored in a shared parent state instead of local component state.
**How to avoid:** Each `MiniMortgageCalculator` instance has its own `useState(marketRate)` for its rate. The `marketRate` prop is only used as the initial/default value.
**Warning signs:** Moving the rate slider in Sell & Move Up changes the rate in Move & Keep as Rental.

### Pitfall 5: Mobile Date Label Overflow
**What goes wrong:** Four date labels ("Mar '26 | Mar '31 | Mar '36 | Mar '41") overflow on small screens.
**Why it happens:** 4 tabs with 7-character labels in a horizontal row on 320px-375px screens.
**How to avoid:** Build with month abbreviation first. Test on iPhone SE (320px), iPhone 12 (390px). If overflow occurs, switch to year-only ("2026 | 2031 | 2036 | 2041") using CSS `@media (max-width: 380px)` or a JS check. The tabs already use `px-3 py-1 text-sm` sizing.
**Warning signs:** Tabs wrapping to two lines, horizontal scroll, or text truncation on mobile.

### Pitfall 6: StayAndBuild Duplicating Existing Data
**What goes wrong:** Stay & Build Wealth shows the same numbers as the equity section above.
**Why it happens:** At the "Now" horizon, Stay & Build shows current home value, current balance, current equity -- which is exactly what EquityGained and CurrentValue sections already display.
**How to avoid:** The "Now" tab for Stay & Build Wealth grounds them in reality, but the VALUE of this card is the future horizons (5/10/15). Consider defaulting this card to the 5yr tab, or using the "Now" tab as a brief anchor that makes the projections feel real by contrast. The narrative should emphasize the trajectory, not repeat current state.

## Code Examples

### New Type Definitions Needed

```typescript
// New horizon type (replaces old 5 | 10 | 15)
type HorizonYears = 0 | 5 | 10 | 15;

// Stay & Build Wealth projection
export interface StayAndBuildProjection {
  years: HorizonYears;
  date: Date;
  homeValue: number;
  mortgageBalance: number;
  equity: number;
}

// Sell & Move Up projection (extends existing pattern)
export interface SellAndMoveUpProjection {
  years: HorizonYears;
  date: Date;
  homeValue: number;
  mortgageBalance: number;
  netProceeds: number;      // after 6% commission + 1% closing
  costToSell: number;       // total fees deducted
}

// Stay & Invest projection (extends existing EquityPlayProjection)
export interface StayAndInvestProjection {
  years: HorizonYears;
  date: Date;
  homeValue: number;
  mortgageBalance: number;
  equity: number;
  helocAvailable: number;
  monthlyHelocCost: number;  // interest-only at market rate
}

// Move & Keep as Rental projection
export interface MoveAndKeepProjection {
  years: HorizonYears;
  date: Date;
  homeValue: number;
  mortgageBalance: number;
  monthlyRent: number;       // grows at rentGrowthRate
  oldMortgagePayment: number;
  monthlyCashFlow: number;   // rent - old mortgage
}
```

### Updated DEFAULTS for Sell & Move Up

```typescript
// In the Sell & Move Up card only (not changing global DEFAULTS):
const SELL_COMMISSION_RATE = 0.06;   // 6% broker fees
const SELL_CLOSING_RATE = 0.01;      // 1% title fees / seller closing costs
const SELL_TOTAL_COST_RATE = 0.07;   // labeled as "cost to sell*"
```

### Horizon Date Label Formatting

```typescript
import { addYears, format } from "date-fns";

function formatHorizonLabel(date: Date, compact: boolean): string {
  return compact
    ? format(date, "yyyy")           // "2026"
    : format(date, "MMM ''yy");      // "Mar '26"
}
```

### Effective Interest Rate Usage in Move & Keep as Rental

```typescript
// Inside MoveAndKeepAsRentalCard:
const rentalCashFlow = monthlyRentEstimate - oldMortgagePayment;
const outOfPocket = Math.max(0, newMortgagePayment - rentalCashFlow);

const effectiveRate = outOfPocket > 0
  ? calcEffectiveInterestRate(newLoanAmount, outOfPocket, 30)
  : 0;

// Display:
// Hero metric: "4.2%" (the effective rate)
// Subtext: "Your effective interest rate"
// Implication: "Rental income offsets your new mortgage, reducing your
//   actual cost to the equivalent of a 4.2% loan"
```

## State of the Art

| Old Approach (Current) | New Approach (Phase 9) | Impact |
|------------------------|------------------------|--------|
| Hold & Rent card | Stay & Build Wealth | Simpler, no rental assumption -- pure equity growth |
| Move Up card with "purchasing power" | Sell & Move Up with net proceeds + mini calculator | More actionable -- shows actual monthly payment instead of abstract purchasing power |
| Equity Play card | Stay & Invest with investment framing | Reframes HELOC as investment capital, adds tax callout |
| Combo scenario (Keep-Rent-Buy) | Move & Keep as Rental with effective rate | Effective rate is the conversation changer -- reframes "rates are too high" objection |
| Horizon tabs: "5yr / 10yr / 15yr" | Dynamic dates: "Mar '26 / Mar '31 / Mar '36 / Mar '41" | Grounded in reality -- actual dates make projections feel real |
| Single rate slider, no per-card override | Rate slider + independent mini calculator overrides | More flexibility -- homeowner can explore different rate assumptions per scenario |
| agentCommission 5% + closingCosts 2% | Commission 6% + closing 1% | Matches current Phoenix market standard broker fee structure |

**Superseded components to remove:**
- `ComboScenario.tsx` -- replaced by Move & Keep as Rental card
- The old `HorizonTabs` with numeric `5 | 10 | 15` type -- replaced by date-based tabs
- `holdHorizonContent`, `moveUpHorizonContent`, `equityPlayHorizonContent` maps in ScenarioCards.tsx -- replaced by new card components

**Components to keep/adapt:**
- `ScenarioCard.tsx` -- the expandable card shell can be reused but may need refactoring for the new card-specific content patterns
- `MarketRateSlider.tsx` -- unchanged
- `AnimatedSection` / `SectionWrapper` -- unchanged

## Open Questions

1. **STR Analyzer URL**
   - What we know: Stay & Invest has a CTA linking to Josh's STR Analyzer tool
   - What's unclear: The exact URL for the deployed STR Analyzer
   - Recommendation: Use the Vercel deployment URL from CLAUDE.md. The repo is at github.com/Joshhoganliveaz/stranalyzer. The deployed URL would be on Vercel. Use a config constant so it can be updated easily.

2. **Net Proceeds Label Change Impact**
   - What we know: Sell & Move Up labels net proceeds as "cost to sell*" with "(title fees & broker fees)" footnote. Commission is 6% + 1% closing.
   - What's unclear: Whether to change global DEFAULTS or use card-local constants
   - Recommendation: Use card-local constants (SELL_COMMISSION_RATE = 0.06, SELL_CLOSING_RATE = 0.01) to avoid side effects on other parts of the app that use DEFAULTS.agentCommissionRate and DEFAULTS.closingCostRate.

3. **New Home Price Input Format**
   - What we know: Mini mortgage calculator needs a new home price input
   - What's unclear: Whether to use a slider, text input, or presets for the home price
   - Recommendation: Use a formatted currency text input with comma separators. The down payment is preset buttons, but the home price must be flexible (homeowners want specific numbers).

4. **Effective Rate When No New Home Selected**
   - What we know: The effective rate requires a new mortgage payment to compute
   - What's unclear: What to show when the homeowner hasn't entered a new home price yet
   - Recommendation: Show a prompt ("Enter a new home price to see your effective rate") instead of computing with $0. The hero metric area displays "--" until the calculator has input.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.0.18 |
| Config file | `vitest.config.ts` (root) |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| SCNR-01 | Four scenario cards render | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | Exists (needs update) |
| SCNR-02 | Dynamic date horizons format correctly | unit | `npx vitest run src/lib/calculations.test.ts -x` | Needs new tests |
| SCNR-03 | Market rate slider + independent overrides | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | Needs update |
| SCNR-04 | Stay & Build Wealth projections correct | unit | `npx vitest run src/lib/calculations.test.ts -x` | Needs new tests |
| SCNR-05 | Sell & Move Up net proceeds + mini calc | unit + component | `npx vitest run src/lib/calculations.test.ts -x` | Partial (calcNetProceeds exists) |
| SCNR-06 | 6% commission + 1% closing costs | unit | `npx vitest run src/lib/calculations.test.ts -x` | Needs new test |
| SCNR-07 | Stay & Invest HELOC + tax callout renders | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | Needs new tests |
| SCNR-08 | Move & Keep as Rental projections correct | unit | `npx vitest run src/lib/calculations.test.ts -x` | Needs new tests |
| SCNR-09 | Effective interest rate calculation accurate | unit | `npx vitest run src/lib/calculations.test.ts -x` | Needs new tests |
| SCNR-10 | Contextual CTAs render per card | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | Needs new tests |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts`
- **Per wave merge:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] New unit tests for `calcEffectiveInterestRate` in `calculations.test.ts`
- [ ] New unit tests for Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep projection functions
- [ ] Updated `ScenarioCards.test.tsx` for new card titles and structure
- [ ] New tests for date horizon formatting/labeling
- [ ] Tests for edge cases: zero mortgage, negative equity, zero rent estimate, outOfPocket <= 0

## Sources

### Primary (HIGH confidence)
- Existing codebase: `src/lib/calculations.ts` -- all existing financial math functions (verified by reading source)
- Existing codebase: `src/lib/types.ts` -- all TypeScript interfaces (verified by reading source)
- Existing codebase: `src/components/homeowner/ScenarioCards.tsx`, `ScenarioCard.tsx`, `HorizonTabs.tsx`, `MarketRateSlider.tsx`, `ComboScenario.tsx` -- current implementation (verified by reading source)
- Existing codebase: `src/lib/defaults.ts` -- all financial defaults (verified by reading source)
- Existing codebase: `src/lib/calculations.test.ts` -- 1000+ lines of existing test coverage (verified by reading source)
- `date-fns` docs -- `format()` and `addYears()` for date horizon labels (standard documented API)

### Secondary (MEDIUM confidence)
- [Newton-Raphson and Bisection Method for Interest Rate Calculation](https://blog.bossylobster.com/2012/05/reverse-calculating-interest-rate) -- validates bisection approach for reverse interest rate solving
- [MFTransparency Interest Rate Calculation](https://www.mftransparency.org/calculating-interest-rates-using-newtons-method/) -- validates Newton-Raphson approach for interest rate solving

### Tertiary (LOW confidence)
- None -- all findings verified against codebase or official documentation

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no new dependencies needed, entire stack already in place
- Architecture: HIGH -- extending existing patterns (ScenarioCard, projection functions, useMemo)
- Calculations: HIGH -- all core math exists, only `calcEffectiveInterestRate` is new (bisection on existing function)
- Pitfalls: HIGH -- identified from direct codebase analysis (rate mismatch, edge cases, mobile overflow)
- Component architecture: MEDIUM -- exact decomposition is discretionary, but patterns are clear

**Research date:** 2026-03-13
**Valid until:** 2026-04-13 (stable -- no library upgrades or API changes expected)
