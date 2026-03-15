# Phase 6: Investment Comparison + Narrative Cleanup - Research

**Researched:** 2026-03-14
**Domain:** Component consolidation, leverage calculations, mortgage rate data, SB7 narrative, information cliff removal
**Confidence:** HIGH

## Summary

Phase 6 merges two existing standalone sections (SP500Callout and RentVsOwn) into a single InvestmentComparison section, adds a leverage hero calculation, conditionally shows a rate lock callout for down-market homeowners, and removes information cliffs from scenario cards. The existing codebase already contains all the calculation primitives needed -- `calcSP500Comparison()`, `calcRentVsOwn()`, and mortgage math functions in `calculations.ts`. The primary new calculation is leverage ROI (equity gained / down payment) and rate lock savings (payment difference at locked rate vs current market rate).

For the current market rate needed by the rate lock callout, the FRED API (Federal Reserve Economic Data) provides free, reliable access to Freddie Mac's PMMS 30-year fixed rate data via a simple JSON endpoint. This is the recommended approach over scraping Mortgage News Daily or building a custom rate feed.

**Primary recommendation:** Build the InvestmentComparison component as a single new file that consumes existing metric shapes from HomeownerMetrics, add two new calculation functions (`calcLeverageROI` and `calcRateLockSavings`), store a `currentMarketRate` value in admin/settings, and modify ScenarioCard to drop the `cliffQuestion` prop.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Single "Investment Comparison" section replaces both SP500Callout and RentVsOwn as standalone sections -- net reduction in page sections
- Section placement: after Equity Gained, before Appreciation Chart (new flow: Equity -> Investment Comparison -> Earnings Reframe -> Chart -> Net Proceeds -> Scenarios)
- Leverage hero number is the centerpiece -- bold standalone callout card, designed to be screenshot-worthy
- Personalized leverage callout beneath hero: "$25K down controlling a $500K asset = $125K in equity built"
- S&P 500 demoted from standalone section to conditional one-liner beneath the leverage callout
- S&P only shown when home outperformed S&P -- hidden entirely when S&P outperformed
- Rent comparison only shown for homeowners who purchased within the last 2 years
- For under-2-year owners: keeps the villain/triumph SB7 contrast
- For owners 2+ years: rent contrast removed entirely
- Rate lock callout only appears when home value has actually gone down (negative appreciation) AND below-market rate
- Rate lock NOT shown when home has appreciated -- Josh does NOT want "golden handcuffs" discouraging homeowners from moving
- Rate lock quantifies monthly and annual savings vs current market rates
- Current market rate sourced automatically (FRED API recommended)
- Scenario cards: keep tap-to-expand interaction pattern
- Remove cliff questions entirely from expanded state
- When expanded, show ALL details
- All three cards expandable independently (not accordion)
- Net proceeds breakdown: stays expandable

### Claude's Discretion
- Exact SB7 copy for leverage section intro, supporting narrative, and tenure-adapted variants
- Leverage callout card visual design (colors, borders, typography within brand guidelines)
- S&P one-liner phrasing
- Rate lock callout visual treatment
- How rent contrast is visually integrated within the merged section for under-2-year owners
- Cascade animation delay for the new section's position in the page flow
- Threshold for "meaningfully below market" rate difference (e.g., 0.5%+ gap)

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| INVS-01 | Single "Investment Comparison" section replaces separate S&P 500 and Rent vs Own sections | New `InvestmentComparison.tsx` component replaces `SP500Callout.tsx` and `RentVsOwn.tsx` imports in `HomeownerPage.tsx` |
| INVS-02 | Section shows three key numbers: home ROI with leverage, S&P alternative return on down payment, and rent drag subtracted from S&P path | New `calcLeverageROI()` function; existing `calcSP500Comparison()` and `calcRentVsOwn()` provide the other two values |
| INVS-03 | Personalized leverage callout using their specific numbers | Leverage ROI = (equityGained / downPayment) * 100; format as "$X down controlling a $Y asset = Z% return" |
| INVS-04 | Rate lock advantage callout quantifying annual savings vs current market rates | New `calcRateLockSavings()` function; FRED API for current market rate; only shown when appreciationPercent < 0 AND rate gap >= threshold |
| SCEN-11 | Information cliffs removed -- full information so homeowners feel empowered | Remove `cliffQuestion` prop from `ScenarioCard.tsx`; remove cliff question rendering from expanded state |
</phase_requirements>

## Standard Stack

### Core (already in project)
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router, server components | Project framework |
| React | 19.2.3 | UI rendering | Project framework |
| TypeScript | 5.x | Type safety | Project standard |
| motion | 12.36.0 | Animation (AnimatePresence, motion.div) | Already used for cascade animations |
| Recharts | 3.8.0 | Data visualization | Already used for AppreciationChart |
| date-fns | 4.1.0 | Date calculations | Already used for differenceInMonths |
| Tailwind CSS | 4.x | Styling | Project standard |

### Supporting (new for this phase)
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| FRED API | N/A | Current 30yr mortgage rate | Rate lock callout -- fetched server-side at build/request time |

### No New Dependencies
This phase requires zero new npm packages. All UI is built with existing motion + Tailwind. The FRED API is a simple `fetch()` call.

## Architecture Patterns

### Component Changes
```
src/components/homeowner/
  InvestmentComparison.tsx   # NEW -- replaces SP500Callout + RentVsOwn
  ScenarioCard.tsx           # MODIFY -- remove cliffQuestion prop
  ScenarioCards.tsx          # MODIFY -- remove cliffQuestion values
  HomeownerPage.tsx          # MODIFY -- swap imports, reorder sections
  SP500Callout.tsx           # DELETE (or keep as dead code for reference)
  RentVsOwn.tsx              # DELETE (or keep as dead code for reference)
```

### Calculation Changes
```
src/lib/
  calculations.ts            # ADD calcLeverageROI(), calcRateLockSavings()
  calculations.test.ts       # ADD tests for new functions
  narrative.ts               # ADD investmentComparisonCopy (3 voice variants)
  types.ts                   # EXTEND HomeownerMetrics with leverageROI, rateLockSavings
```

### Server-Side Changes
```
src/app/h/[slug]/page.tsx    # ADD current market rate fetch, pass to client
```

### Pattern 1: Leverage ROI Calculation
**What:** Simple ROI based on down payment vs total equity gained
**When to use:** Always shown as the hero number
**Example:**
```typescript
// Source: derived from existing codebase patterns
export function calcLeverageROI(
  downPayment: number,
  equityGained: number,
  currentValue: number,
  purchasePrice: number
): {
  leverageROI: number;        // decimal (1.65 = 165%)
  downPayment: number;
  equityGained: number;
  currentValue: number;
  purchasePrice: number;
} {
  if (downPayment <= 0) {
    return { leverageROI: 0, downPayment, equityGained, currentValue, purchasePrice };
  }
  return {
    leverageROI: equityGained / downPayment,
    downPayment,
    equityGained,
    currentValue,
    purchasePrice,
  };
}
```

### Pattern 2: Rate Lock Savings Calculation
**What:** Monthly/annual savings from having a locked rate below current market
**When to use:** Only when appreciation is negative AND rate gap is meaningful
**Example:**
```typescript
// Source: derived from existing calcMortgagePayment pattern
export function calcRateLockSavings(
  loanAmount: number,
  lockedRate: number,
  currentMarketRate: number,
  loanTerm: number
): {
  lockedPayment: number;
  marketPayment: number;
  monthlySavings: number;
  annualSavings: number;
  rateDifference: number;
} {
  const lockedPayment = calcMortgagePayment(loanAmount, lockedRate, loanTerm);
  const marketPayment = calcMortgagePayment(loanAmount, currentMarketRate, loanTerm);
  const monthlySavings = marketPayment - lockedPayment;
  return {
    lockedPayment,
    marketPayment,
    monthlySavings,
    annualSavings: monthlySavings * 12,
    rateDifference: currentMarketRate - lockedRate,
  };
}
```

### Pattern 3: Section Order Resequencing
**What:** New section flow with updated cascade delays
**Current flow:** Hero -> Note -> Purchase -> CurrentValue(0) -> Equity(500) -> Earnings(1000) -> Chart -> SP500(1500) -> RentVsOwn(2000) -> NetProceeds(2500) -> Scenarios -> CTA
**New flow:** Hero -> Note -> Purchase -> CurrentValue(0) -> Equity(500) -> **InvestmentComparison(1000)** -> Earnings(1500) -> Chart -> NetProceeds(2000) -> Scenarios -> CTA

The Investment Comparison section takes the cascade slot previously held by Earnings Reframe, and everything after shifts by 500ms.

### Pattern 4: Conditional Rendering Logic
**What:** Three layers of conditional content within InvestmentComparison
```typescript
// Always shown:
// - Leverage hero number + personalized callout

// Conditional S&P one-liner (only when home outperformed):
const showSP500 = sp500Comparison.difference >= 0 && sp500Comparison.sp500Return > 0;

// Conditional rent contrast (only under 2 years ownership):
const showRentContrast = ownershipMonths < 24;

// Conditional rate lock (only negative appreciation + meaningful rate gap):
const showRateLock = appreciationPercent < 0
  && rateLockSavings.rateDifference >= 0.005; // 0.5% threshold
```

### Anti-Patterns to Avoid
- **Rebuilding calculation orchestrator:** Do not duplicate `computeHomeownerMetrics`. Add new calculations as additional functions and extend the return type.
- **Client-side rate fetching:** The FRED API call must happen server-side. Never expose API keys to the client.
- **Accordion behavior on scenario cards:** Josh explicitly wants multiple cards open simultaneously. Do not use a shared state that closes one when another opens.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Current mortgage rate | Custom scraper for Mortgage News Daily | FRED API (`MORTGAGE30US` series) | Free, reliable, official government data, weekly updates, simple JSON endpoint |
| Animation sequencing | Custom setTimeout chains | motion's existing `AnimatedSection` + `cascadeDelay` pattern | Already proven in the codebase |
| Number formatting | Manual string manipulation | Existing `formatCurrency()` and `formatPercent()` | Already handles edge cases (negative values, zero decimals) |
| Narrative voice selection | Hardcoded month checks | Existing `getNarrativeVoice()` returns "recent"/"midterm"/"longterm" | Already defines the 24-month and 84-month thresholds |

## Common Pitfalls

### Pitfall 1: Down Payment = 0 (Cash Purchase or Refi)
**What goes wrong:** Division by zero in leverage ROI when downPayment is 0
**Why it happens:** Cash purchases have loanAmount = 0, so downPayment = purchasePrice - 0 = purchasePrice (fine). But refinanced homeowners might have originalLoanAmount = null and loanAmount > purchasePrice, making downPayment negative.
**How to avoid:** Guard `downPayment <= 0` returns a zeroed-out leverage result. The existing `computeHomeownerMetrics` already computes `downPayment = purchasePrice - (originalLoanAmount ?? loanAmount)` with this edge case.
**Warning signs:** NaN or Infinity in leverage percentage display.

### Pitfall 2: Rate Lock Shown When Appreciation Is Positive
**What goes wrong:** Josh explicitly does NOT want rate lock callout when home has appreciated -- it creates "golden handcuffs" suggesting the homeowner should stay put.
**Why it happens:** Developer might check rate difference only, not appreciation direction.
**How to avoid:** Two-part guard: `appreciationPercent < 0 AND rateDifference >= threshold`. Both conditions required.
**Warning signs:** Rate lock callout appearing on pages where home value increased.

### Pitfall 3: Cascade Delay Misalignment After Section Reorder
**What goes wrong:** Counter-roll animations fire out of sequence after adding/removing/reordering sections.
**Why it happens:** Cascade delays are currently hardcoded at 0/500/1000/1500/2000/2500ms. Changing section order without updating delays breaks the visual cascade.
**How to avoid:** Update all cascadeDelay values in HomeownerPage.tsx simultaneously when reordering sections.
**Warning signs:** Sections animating in wrong order or simultaneously.

### Pitfall 4: Rent Contrast Edge Case at Exactly 24 Months
**What goes wrong:** Ambiguity at the 24-month boundary -- should a homeowner at exactly 24 months see the rent contrast?
**Why it happens:** `ownershipMonths < 24` vs `ownershipMonths <= 24`.
**How to avoid:** Use `< 24` (strict less than) to match the existing `getNarrativeVoice()` threshold which uses `< 24` for "recent" voice.
**Warning signs:** Inconsistent behavior between voice selection and rent contrast visibility.

### Pitfall 5: FRED API Key Exposure
**What goes wrong:** API key leaked to client bundle.
**Why it happens:** Calling FRED API from a client component or not using server-only patterns.
**How to avoid:** Fetch market rate in the server component (`/h/[slug]/page.tsx`) and pass the rate value (not the API key) as a prop. Store the API key in `.env.local` as `FRED_API_KEY`.
**Warning signs:** API key visible in browser network tab or Next.js client bundle.

### Pitfall 6: FRED API Unavailability
**What goes wrong:** FRED API is down or returns stale data, breaking the rate lock callout.
**Why it happens:** External API dependency.
**How to avoid:** Use a fallback rate from `DEFAULTS.interestRate` (currently 0.065) if FRED API fails. Cache the rate server-side (ISR or a simple env variable fallback). The rate only changes weekly, so aggressive caching is safe.
**Warning signs:** Rate lock callout showing $0 savings or not appearing when expected.

## Code Examples

### Leverage Hero Number Rendering
```typescript
// Source: pattern from EquityGained.tsx adapted for leverage
<CountUpNumber
  value={leverageROI.leverageROI * 100}  // e.g., 165
  format="percent"                         // or custom format
  suffix="%"
  className="text-charcoal font-heading text-5xl md:text-6xl lg:text-7xl font-bold"
  cascadeDelay={cascadeDelay}
/>
<p className="text-charcoal/70 font-body text-lg max-w-md mx-auto">
  {formatCurrency(leverageROI.downPayment)} down controlling a{" "}
  {formatCurrency(leverageROI.currentValue)} asset ={" "}
  {formatCurrency(leverageROI.equityGained)} in equity built
</p>
```

### Conditional S&P One-Liner
```typescript
// Source: logic from existing SP500Callout.tsx simplified
{showSP500 && (
  <p className="text-charcoal/50 font-body text-sm mt-4">
    Even the S&P 500 would have only returned{" "}
    <span className="font-semibold">{formatCurrency(sp500Comparison.sp500Return)}</span>{" "}
    on that same {formatCurrency(downPayment)}.
  </p>
)}
```

### FRED API Server-Side Fetch
```typescript
// Source: FRED API documentation (https://fred.stlouisfed.org/docs/api/fred/)
async function getCurrentMortgageRate(): Promise<number> {
  const apiKey = process.env.FRED_API_KEY;
  if (!apiKey) return DEFAULTS.interestRate;

  try {
    const res = await fetch(
      `https://api.stlouisfed.org/fred/series/observations?series_id=MORTGAGE30US&api_key=${apiKey}&file_type=json&sort_order=desc&limit=1`,
      { next: { revalidate: 86400 } } // Cache for 24 hours
    );
    const data = await res.json();
    const latestRate = parseFloat(data.observations?.[0]?.value);
    return Number.isFinite(latestRate) ? latestRate / 100 : DEFAULTS.interestRate;
  } catch {
    return DEFAULTS.interestRate;
  }
}
```

### ScenarioCard Without Cliff Question
```typescript
// Source: existing ScenarioCard.tsx with cliffQuestion removed
interface ScenarioCardProps {
  title: string;
  heroMetric: string;
  implication: string;
  details: { label: string; value: string }[];
  // cliffQuestion removed
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Separate S&P + RentVsOwn sections | Merged InvestmentComparison | Phase 6 | Net reduction in page sections, leverage-first narrative |
| Cliff questions in scenario cards | Full details shown when expanded | Phase 6 | Homeowners get complete information without friction |
| Static market rate in DEFAULTS | FRED API with fallback | Phase 6 | Rate lock callout uses real current rates |

## Open Questions

1. **FRED API Key Registration**
   - What we know: Free registration at https://fred.stlouisfed.org/docs/api/api_key.html, requires email
   - What's unclear: Whether Josh already has a FRED API key
   - Recommendation: Register for key during Wave 0, store in `.env.local` as `FRED_API_KEY`. Fallback to `DEFAULTS.interestRate` if not set.

2. **Admin UI for Current Market Rate Override**
   - What we know: FRED API provides automatic rates, but Josh may want manual override
   - What's unclear: Whether a manual override field is needed in admin form
   - Recommendation: Use FRED API as primary source with `DEFAULTS.interestRate` as fallback. If Josh wants manual control, add a `currentMarketRate` column to the homeowners table in a future phase. For now, the FRED API approach is sufficient.

3. **CountUpNumber Format for Percentage**
   - What we know: CountUpNumber currently supports "currency" format via `formatCurrency`
   - What's unclear: Whether it supports a raw percentage format like "165%"
   - Recommendation: Check CountUpNumber implementation. May need to add a "percent" format option or pass a custom formatter. The leverage ROI could also be displayed as a dollar amount (equity gained) rather than percentage if formatting is simpler.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.0.18 |
| Config file | `vitest.config.ts` (exists, jsdom environment) |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| INVS-01 | InvestmentComparison renders, SP500Callout/RentVsOwn removed from page | smoke (manual) | Visual check in browser | N/A |
| INVS-02 | calcLeverageROI returns correct ROI, handles edge cases | unit | `npx vitest run src/lib/calculations.test.ts -t "calcLeverageROI"` | Wave 0 |
| INVS-03 | Leverage callout formats personalized copy with correct numbers | unit | `npx vitest run src/lib/calculations.test.ts -t "calcLeverageROI"` | Wave 0 |
| INVS-04 | calcRateLockSavings computes monthly/annual savings correctly | unit | `npx vitest run src/lib/calculations.test.ts -t "calcRateLockSavings"` | Wave 0 |
| INVS-04 | Rate lock only shown when appreciation < 0 AND rate gap >= 0.5% | unit | `npx vitest run src/lib/calculations.test.ts -t "rateLock"` | Wave 0 |
| SCEN-11 | ScenarioCard renders without cliffQuestion, shows full details | unit | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx` | Exists (needs update) |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts`
- **Per wave merge:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `src/lib/calculations.test.ts` -- add tests for `calcLeverageROI` and `calcRateLockSavings` (file exists, needs new test blocks)
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` -- update to verify cliffQuestion removal (file exists, needs update)
- [ ] FRED API key -- register at https://fred.stlouisfed.org/docs/api/api_key.html and add to `.env.local`

## Sources

### Primary (HIGH confidence)
- Codebase inspection: `calculations.ts`, `HomeownerPage.tsx`, `SP500Callout.tsx`, `RentVsOwn.tsx`, `ScenarioCard.tsx`, `ScenarioCards.tsx`, `narrative.ts`, `types.ts`, `defaults.ts`, `format.ts`
- Codebase inspection: `src/app/h/[slug]/page.tsx` (server component rendering pattern)
- Codebase inspection: `vitest.config.ts`, `calculations.test.ts` (test infrastructure)

### Secondary (MEDIUM confidence)
- [FRED API documentation](https://fred.stlouisfed.org/docs/api/fred/) -- JSON endpoint for MORTGAGE30US series, free API key
- [FRED MORTGAGE30US series](https://fred.stlouisfed.org/series/MORTGAGE30US) -- 30-year fixed rate mortgage weekly data

### Tertiary (LOW confidence)
- None -- all findings verified against codebase or official documentation

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all libraries already in use, zero new dependencies
- Architecture: HIGH -- patterns directly observed in existing codebase
- Calculations: HIGH -- new functions follow exact patterns of existing `calcSP500Comparison`, `calcRentVsOwn`
- FRED API: MEDIUM -- documentation confirmed, but untested in this specific project
- Pitfalls: HIGH -- derived from actual code inspection of edge cases already handled

**Research date:** 2026-03-14
**Valid until:** 2026-04-14 (stable codebase, no fast-moving dependencies)
