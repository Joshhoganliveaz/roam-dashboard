---
phase: 09-scenario-cards-rework
verified: 2026-03-14T19:57:00Z
status: passed
score: 10/10 must-haves verified
re_verification:
  previous_status: passed
  previous_score: 10/10
  gaps_closed: []
  gaps_remaining: []
  regressions: []
---

# Phase 9: Scenario Cards Rework Verification Report

**Phase Goal:** Rework scenario cards with effective interest rate, date horizons, mini mortgage calculators, and four new homeowner decision paths
**Verified:** 2026-03-14T19:57:00Z
**Status:** passed
**Re-verification:** Yes -- confirming previous passed status holds

---

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | calcEffectiveInterestRate returns the annual rate that produces a given monthly payment on a given loan | VERIFIED | Bisection solver at calculations.ts:346-378, calls calcMortgagePayment(loanAmount, mid, termYears) in loop, returns null for invalid inputs, 0 for below-zero-rate threshold |
| 2 | getHorizonDates returns four date-based horizon objects (0, 5, 10, 15 years from now) with formatted labels | VERIFIED | horizon-dates.ts:28-38 maps HORIZON_YEARS [0,5,10,15] with format(date, "MMM ''yy") for label, format(date, "yyyy") for shortLabel |
| 3 | New projection types compile and export correctly for all four scenario cards | VERIFIED | types.ts:295-332 declares HorizonYears, StayAndBuildProjection, SellAndMoveUpProjection, StayAndInvestProjection, MoveAndKeepProjection -- all imported and used by card components |
| 4 | HorizonTabs renders four date-based tabs with formatted date labels and mobile year-only fallback | VERIFIED | HorizonTabs.tsx:36-37 uses hidden sm:inline for label, inline sm:hidden for shortLabel; renders horizons.map() with olive active styling |
| 5 | MiniMortgageCalculator renders home price input, 4 down payment preset buttons (5/10/15/20%), rate override, and monthly P&I output | VERIFIED | MiniMortgageCalculator.tsx: DOWN_PAYMENT_PRESETS=[0.05,0.10,0.15,0.20], text input with inputMode="numeric", rate override with resetRate callback, P&I via calcMortgagePayment |
| 6 | Stay & Build Wealth card shows home value, mortgage balance, and equity at each horizon | VERIFIED | StayAndBuildCard.tsx:83-96 builds details array with "Home value", "Mortgage balance", "Total equity" for all 4 horizons using calcRemainingBalance and appreciation compounding |
| 7 | Sell & Move Up card shows net proceeds with "cost to sell*" label, uses 6% commission + 1% closing, includes MiniMortgageCalculator | VERIFIED | SellAndMoveUpCard.tsx imports SELL_COMMISSION_RATE (0.06) + SELL_CLOSING_RATE (0.01) from horizon-dates; line 100 shows "Cost to sell*"; embeds MiniMortgageCalculator at line 130 |
| 8 | Stay & Invest card shows HELOC available, monthly HELOC cost, investment property framing, and tax advantages callout | VERIFIED | StayAndInvestCard.tsx:99-104 shows HELOC available and monthly cost details; lines 131-138 render olive-tinted OBBBA/depreciation callout; STR Analyzer link at line 144; Josh & Jacqui CTA at line 161 |
| 9 | Move & Keep as Rental card shows the effective interest rate as hero metric, rental cash flow, and mini mortgage calculator | VERIFIED | MoveAndKeepAsRentalCard.tsx:137-156 computes three effectiveRateResult states (computed/fully-covered/negative-cashflow); lines 200-208 show olive box with rate%; MiniMortgageCalculator with onPaymentChange at line 179 |
| 10 | All four new scenario cards render in ScenarioCards.tsx, replacing the old four cards, with market rate slider controlling baseline | VERIFIED | ScenarioCards.tsx:79-105 renders StayAndBuildCard, SellAndMoveUpCard, StayAndInvestCard, MoveAndKeepAsRentalCard; MarketRateSlider at line 72; horizons computed via getHorizonDates at line 51 |

**Score:** 10/10 truths verified

---

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `src/lib/calculations.ts` | calcEffectiveInterestRate function | VERIFIED | Substantive bisection implementation at line 346; exports calcEffectiveInterestRate and NEW_PROJECTION_HORIZONS |
| `src/lib/horizon-dates.ts` | getHorizonDates utility and HorizonDate type | VERIFIED | 53-line file with getHorizonDates, HorizonDate interface, SELL_COMMISSION_RATE/SELL_CLOSING_RATE/SELL_TOTAL_COST_RATE constants |
| `src/lib/types.ts` | Four new projection type interfaces | VERIFIED | StayAndBuildProjection, SellAndMoveUpProjection, StayAndInvestProjection, MoveAndKeepProjection at lines 297-332; HorizonYears type at line 295 |
| `src/lib/calculations.test.ts` | Test coverage for calcEffectiveInterestRate | VERIFIED | 101 total calc tests passing (includes effective rate tests) |
| `src/lib/horizon-dates.test.ts` | Test coverage for getHorizonDates | VERIFIED | 14 tests all passing |
| `src/components/homeowner/HorizonTabs.tsx` | Date-based horizon tab selector with mobile fallback | VERIFIED | 42-line component with sm:inline/inline sm:hidden responsive classes, e.stopPropagation() on click |
| `src/components/homeowner/MiniMortgageCalculator.tsx` | Reusable mini mortgage calculator with local state | VERIFIED | 194-line component with useState for homePrice, downPct, rate; DOWN_PAYMENT_PRESETS buttons; rate reset link |
| `src/components/homeowner/StayAndBuildCard.tsx` | Stay & Build Wealth scenario card | VERIFIED | 119-line component with useMemo projections for all 4 horizons, initialHorizon=5, SB7 narrative |
| `src/components/homeowner/SellAndMoveUpCard.tsx` | Sell & Move Up scenario card with net proceeds and mini calculator | VERIFIED | 151-line component with SELL rates imported, "cost to sell*" footnote, MiniMortgageCalculator, lender tel CTA |
| `src/components/homeowner/StayAndInvestCard.tsx` | Stay & Invest scenario card with HELOC and tax callout | VERIFIED | 172-line component with HELOC projections, olive tax callout, STR_ANALYZER_URL, Josh & Jacqui warm CTA |
| `src/components/homeowner/MoveAndKeepAsRentalCard.tsx` | Move & Keep as Rental card with effective interest rate hero metric | VERIFIED | 246-line component with calcEffectiveInterestRate wired via onPaymentChange, 3-state effective rate display, initialHorizon=0 |
| `src/components/homeowner/ScenarioCards.tsx` | Refactored parent component rendering all 4 new scenario cards | VERIFIED | 111-line refactored file; imports all 4 new cards; MarketRateSlider; getHorizonDates; old projection code removed |
| `src/components/homeowner/ScenarioCards.test.tsx` | Updated tests for new scenario card structure | VERIFIED | 9 tests passing in ScenarioCards.test.tsx |

---

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `calculations.ts` | `calcMortgagePayment` | bisection calling calcMortgagePayment(loanAmount, mid, termYears) | WIRED | Line 365: `const payment = calcMortgagePayment(loanAmount, mid, termYears)` |
| `horizon-dates.ts` | `date-fns` | import addYears and format | WIRED | Line 1: `import { addYears, format } from "date-fns"` |
| `HorizonTabs.tsx` | `horizon-dates.ts` | HorizonDate type import | WIRED | Line 11: `import type { HorizonDate } from "@/lib/horizon-dates"` |
| `MiniMortgageCalculator.tsx` | `calculations.ts` | calcMortgagePayment for P&I output | WIRED | Line 4 import; Line 43: `calcMortgagePayment(loanAmount, rate, 30)` |
| `SellAndMoveUpCard.tsx` | `horizon-dates.ts` | SELL_COMMISSION_RATE and SELL_CLOSING_RATE | WIRED | Line 6 import; used at lines 51 and 75 |
| `MoveAndKeepAsRentalCard.tsx` | `calculations.ts` | calcEffectiveInterestRate for hero metric | WIRED | Line 6 import; Line 154: `calcEffectiveInterestRate(newLoanAmount, outOfPocket, 30)` |
| `ScenarioCards.tsx` | `StayAndBuildCard.tsx` | import and render | WIRED | Line 11 import; Line 79 render |
| `ScenarioCards.tsx` | `horizon-dates.ts` | getHorizonDates for date-based horizons | WIRED | Line 7 import; Line 51: `useMemo(() => getHorizonDates(new Date()), [])` |

---

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| SCNR-01 | 09-02, 09-03 | Four scenario cards: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental | SATISFIED | All 4 card components exist and render in ScenarioCards.tsx lines 79-105 |
| SCNR-02 | 09-01, 09-02 | Dynamic date horizons from page view date, year-only mobile fallback | SATISFIED | getHorizonDates(new Date()) in ScenarioCards; HorizonTabs uses hidden sm:inline / inline sm:hidden |
| SCNR-03 | 09-02 | One market rate slider above all cards; individual card calculators can override independently | SATISFIED | MarketRateSlider in ScenarioCards.tsx; MiniMortgageCalculator owns local rate state with reset-to-market link |
| SCNR-04 | 09-02 | Stay & Build Wealth shows home value, mortgage balance, equity at each horizon | SATISFIED | StayAndBuildCard horizonContent details: "Home value", "Mortgage balance", "Total equity" |
| SCNR-05 | 09-02 | Sell & Move Up shows net proceeds ("cost to sell*"), mini mortgage calculator with 5/10/15/20% down payment presets | SATISFIED | "Cost to sell*" at line 100; MiniMortgageCalculator with DOWN_PAYMENT_PRESETS=[0.05,0.10,0.15,0.20] |
| SCNR-06 | 09-01, 09-02 | Sell & Move Up uses 6% commission + 1% seller closing costs | SATISFIED | SELL_COMMISSION_RATE=0.06, SELL_CLOSING_RATE=0.01 in horizon-dates.ts; imported and used in SellAndMoveUpCard |
| SCNR-07 | 09-03 | Stay & Invest shows HELOC available, monthly cost, rental investment framing, tax advantages callout, CTAs to STR Analyzer and Josh & Jacqui | SATISFIED | HELOC details at lines 99-104; olive OBBBA callout at 131-138; STR_ANALYZER_URL link; Josh & Jacqui tel CTA |
| SCNR-08 | 09-03 | Move & Keep as Rental shows rental income, cash flow, effective interest rate as hero metric, mini mortgage calculator | SATISFIED | monthlyRent/monthlyCashFlow in horizonContent; effective rate in olive box; MiniMortgageCalculator with onPaymentChange |
| SCNR-09 | 09-01, 09-03 | Effective interest rate: rate that produces out-of-pocket payment on same loan amount | SATISFIED | calcEffectiveInterestRate bisection solver; MoveAndKeepAsRentalCard: outOfPocket = newMortgagePayment - rentalCashFlow |
| SCNR-10 | 09-03 | Each scenario has contextual CTAs | SATISFIED | StayAndBuildCard: no CTA (do-nothing); SellAndMoveUpCard: lender tel; StayAndInvestCard: STR Analyzer + Josh & Jacqui; MoveAndKeepAsRentalCard: Josh tel |

**All 10 requirements: SATISFIED**

No orphaned requirements found -- all SCNR-01 through SCNR-10 are claimed by plans and verified.

---

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `MiniMortgageCalculator.tsx` | 112 | HTML `placeholder="Enter price"` | Info | Standard HTML input attribute, not a code stub |

No TODO/FIXME markers, empty implementations, or code stubs found in any phase 9 files.

---

### Human Verification Required

#### 1. Date-based horizon tab rendering

**Test:** Open a homeowner page, scroll to scenario cards, observe tab labels on each card.
**Expected:** Four pill tabs showing "Mar '26", "Mar '31", "Mar '36", "Mar '41" on desktop; "2026", "2031", "2036", "2041" on mobile (< 640px).
**Why human:** CSS responsive classes (hidden sm:inline / inline sm:hidden) require a real browser rendering engine.

#### 2. MiniMortgageCalculator independent state

**Test:** Expand "Sell & Move Up", enter a home price. Then expand "Move & Keep as Rental", enter a different price. Return to first card.
**Expected:** Each calculator retains its own values independently.
**Why human:** useState isolation across expanded/collapsed card lifecycles requires browser interaction.

#### 3. Effective interest rate live calculation

**Test:** Expand "Move & Keep as Rental", enter a new home price (e.g., $600,000) with 20% down. Observe effective rate display.
**Expected:** Olive box appears showing a rate lower than market rate (rental cash flow offsets new mortgage).
**Why human:** Requires real interaction with MiniMortgageCalculator + onPaymentChange callback triggering effective rate computation.

#### 4. MiniMortgageCalculator rate override and reset

**Test:** In any card with a MiniMortgageCalculator, change the interest rate field. Then click "Reset to market".
**Expected:** Rate changes reflect immediately in P&I. Reset restores slider market rate. "Reset to market" link disappears when rate matches market.
**Why human:** Real-time state update behavior requires browser interaction.

---

### Gaps Summary

No gaps. All 10 observable truths verified against actual codebase. All 13 artifacts are substantive and wired. All 8 key links confirmed. All 10 requirements satisfied. All 245 tests pass. No anti-patterns found.

---

## Commits Verified

All 6 phase-9 commits confirmed in git log:

| Commit | Plan | Description |
|--------|------|-------------|
| `2c21269` | 09-01 Task 1 | test: add horizon dates, projection types, and sell cost constants |
| `d5c36f0` | 09-01 Task 2 | feat: add calcEffectiveInterestRate and NEW_PROJECTION_HORIZONS |
| `4ebc353` | 09-02 Task 1 | feat: refactor HorizonTabs to date-based labels and create MiniMortgageCalculator |
| `af45b13` | 09-02 Task 2 | feat: add StayAndBuildCard and SellAndMoveUpCard scenario components |
| `a4b032c` | 09-03 Task 1 | feat: add StayAndInvestCard and MoveAndKeepAsRentalCard components |
| `3289722` | 09-03 Task 2 | feat: wire four new scenario cards into ScenarioCards and update tests |

---

_Verified: 2026-03-14T19:57:00Z_
_Verifier: Claude (gsd-verifier)_
