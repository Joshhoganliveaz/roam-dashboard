---
phase: 09-scenario-cards-rework
verified: 2026-03-13T23:30:00Z
status: passed
score: 10/10 must-haves verified
re_verification: false
---

# Phase 9: Scenario Cards Rework Verification Report

**Phase Goal:** Rework scenario cards with effective interest rate, date horizons, mini mortgage calculators, and four new homeowner decision paths
**Verified:** 2026-03-13T23:30:00Z
**Status:** passed
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | calcEffectiveInterestRate returns the annual rate that produces a given monthly payment on a given loan | VERIFIED | bisection solver at calculations.ts:346 with 11 passing tests incl. round-trip within $0.01 |
| 2 | getHorizonDates returns four date-based horizon objects (0, 5, 10, 15 years from now) with formatted labels | VERIFIED | horizon-dates.ts:28 with 14 passing tests confirming "Mar '26/Mar '31/Mar '36/Mar '41" labels |
| 3 | New projection types compile and export correctly for all four scenario cards | VERIFIED | types.ts:288-327 declares HorizonYears, StayAndBuildProjection, SellAndMoveUpProjection, StayAndInvestProjection, MoveAndKeepProjection — TypeScript compiles cleanly in production code |
| 4 | HorizonTabs renders four date-based tabs with formatted date labels and mobile year-only fallback | VERIFIED | HorizonTabs.tsx:36-37 uses `hidden sm:inline` / `inline sm:hidden` CSS for label/shortLabel, renders horizons.map() |
| 5 | MiniMortgageCalculator renders home price input, 4 down payment preset buttons (5/10/15/20%), rate override, and monthly P&I output | VERIFIED | MiniMortgageCalculator.tsx with DOWN_PAYMENT_PRESETS=[0.05,0.10,0.15,0.20], home price text input, rate override with reset link, P&I output via calcMortgagePayment |
| 6 | Stay & Build Wealth card shows home value, mortgage balance, and equity at each horizon | VERIFIED | StayAndBuildCard.tsx:79-95 builds horizonContent with all 4 horizons using calcRemainingBalance and appreciation compounding |
| 7 | Sell & Move Up card shows net proceeds with "cost to sell*" label, uses 6% commission + 1% closing, includes MiniMortgageCalculator | VERIFIED | SellAndMoveUpCard.tsx imports SELL_COMMISSION_RATE (0.06) + SELL_CLOSING_RATE (0.01) from horizon-dates, shows "cost to sell*" detail row, embeds MiniMortgageCalculator |
| 8 | Stay & Invest card shows HELOC available, monthly HELOC cost, investment property framing, and tax advantages callout | VERIFIED | StayAndInvestCard.tsx:98-107 shows HELOC available/monthly cost details; lines 130-138 render olive-tinted OBBBA/depreciation callout; STR Analyzer link and Josh & Jacqui CTA present |
| 9 | Move & Keep as Rental card shows the effective interest rate as hero metric, rental cash flow, and mini mortgage calculator | VERIFIED | MoveAndKeepAsRentalCard.tsx:133-228 computes three effectiveRateResult states (computed/fully-covered/negative-cashflow), shows olive box with rate%, embeds MiniMortgageCalculator with onPaymentChange callback |
| 10 | All four new scenario cards render in ScenarioCards.tsx, replacing the old four cards, with market rate slider controlling baseline | VERIFIED | ScenarioCards.tsx:42-111 renders all 4 new cards; old projection code removed; MarketRateSlider present; horizons computed once via useMemo |

**Score:** 10/10 truths verified

---

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `src/lib/calculations.ts` | calcEffectiveInterestRate function | VERIFIED | Substantive bisection implementation at line 346; exports calcEffectiveInterestRate and NEW_PROJECTION_HORIZONS |
| `src/lib/horizon-dates.ts` | getHorizonDates utility and HorizonDate type | VERIFIED | 53-line file with getHorizonDates, HorizonDate interface, SELL_COMMISSION_RATE/SELL_CLOSING_RATE/SELL_TOTAL_COST_RATE constants |
| `src/lib/types.ts` | Four new projection type interfaces | VERIFIED | StayAndBuildProjection, SellAndMoveUpProjection, StayAndInvestProjection, MoveAndKeepProjection at lines 292-327; HorizonYears type at line 290 |
| `src/lib/calculations.test.ts` | Test coverage for calcEffectiveInterestRate | VERIFIED | 11 new tests: round-trip, null cases, zero-rate threshold, high boundary convergence — all passing |
| `src/lib/horizon-dates.test.ts` | Test coverage for getHorizonDates | VERIFIED | 14 tests covering all 4 horizons, label formatting, shortLabel, date correctness, sell cost constants — all passing |
| `src/components/homeowner/HorizonTabs.tsx` | Date-based horizon tab selector with mobile fallback | VERIFIED | Accepts HorizonDate[], renders with sm:inline/inline sm:hidden responsive classes, e.stopPropagation() on click |
| `src/components/homeowner/MiniMortgageCalculator.tsx` | Reusable mini mortgage calculator with local state | VERIFIED | useState for homePrice, downPct, rate; calcMortgagePayment for P&I; DOWN_PAYMENT_PRESETS buttons; rate reset link |
| `src/components/homeowner/StayAndBuildCard.tsx` | Stay & Build Wealth scenario card | VERIFIED | useMemo projections for all 4 horizons, initialHorizon=5, SB7 narrative, no CTA (do-nothing scenario) |
| `src/components/homeowner/SellAndMoveUpCard.tsx` | Sell & Move Up scenario card with net proceeds and mini calculator | VERIFIED | SELL_COMMISSION_RATE/SELL_CLOSING_RATE imported and used, "cost to sell*" footnote, MiniMortgageCalculator, lender tel CTA |
| `src/components/homeowner/StayAndInvestCard.tsx` | Stay & Invest scenario card with HELOC and tax callout | VERIFIED | HELOC projections, olive-tinted tax callout with OBBBA depreciation text, STR_ANALYZER_URL constant, Josh & Jacqui warm CTA |
| `src/components/homeowner/MoveAndKeepAsRentalCard.tsx` | Move & Keep as Rental card with effective interest rate hero metric | VERIFIED | calcEffectiveInterestRate wired via onPaymentChange callback, 3-state effective rate display, initialHorizon=0 |
| `src/components/homeowner/ScenarioCards.tsx` | Refactored parent component rendering all 4 new scenario cards | VERIFIED | 111-line refactored file (from ~341 lines); imports all 4 new cards; MarketRateSlider; getHorizonDates; old projection code removed |
| `src/components/homeowner/ScenarioCards.test.tsx` | Updated tests for new scenario card structure | VERIFIED | 9 tests: 4 card titles, section header, horizon tabs expansion, voice intros, market rate slider, effective rate section, tax callout, STR Analyzer link |

---

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `src/lib/calculations.ts` | `calcMortgagePayment` | bisection calling calcMortgagePayment(loanAmount, mid, termYears) | WIRED | Line 365: `const payment = calcMortgagePayment(loanAmount, mid, termYears)` inside bisection loop |
| `src/lib/horizon-dates.ts` | `date-fns` | import addYears and format | WIRED | Line 1: `import { addYears, format } from "date-fns"` — both used in getHorizonDates |
| `src/components/homeowner/HorizonTabs.tsx` | `src/lib/horizon-dates.ts` | HorizonDate type import | WIRED | Line 11: `import type { HorizonDate } from "@/lib/horizon-dates"` — used in props interface |
| `src/components/homeowner/MiniMortgageCalculator.tsx` | `src/lib/calculations.ts` | calcMortgagePayment for P&I output | WIRED | Line 4 import, Line 43 usage: `calcMortgagePayment(loanAmount, rate, 30)` in computed value |
| `src/components/homeowner/SellAndMoveUpCard.tsx` | `src/lib/horizon-dates.ts` | SELL_COMMISSION_RATE and SELL_CLOSING_RATE | WIRED | Line 6 import; used at lines 51 and 75 in calcNetProceeds calls |
| `src/components/homeowner/MoveAndKeepAsRentalCard.tsx` | `src/lib/calculations.ts` | calcEffectiveInterestRate for hero metric | WIRED | Line 6 import; Line 154: `calcEffectiveInterestRate(newLoanAmount, outOfPocket, 30)` inside useMemo |
| `src/components/homeowner/ScenarioCards.tsx` | `src/components/homeowner/StayAndBuildCard.tsx` | import and render | WIRED | Line 11 import; Line 79-84 render with record/metrics/marketRate/horizons props |
| `src/components/homeowner/ScenarioCards.tsx` | `src/lib/horizon-dates.ts` | getHorizonDates for date-based horizons | WIRED | Line 7 import; Line 51: `useMemo(() => getHorizonDates(new Date()), [])` |

---

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| SCNR-01 | 09-02, 09-03 | Four scenario cards: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental | SATISFIED | All 4 card titles confirmed in ScenarioCards.test.tsx; all 4 components exist and render in ScenarioCards.tsx |
| SCNR-02 | 09-01, 09-02 | Dynamic date horizons from page view date (Mar '26 / Mar '31 / Mar '36 / Mar '41), year-only mobile fallback | SATISFIED | getHorizonDates(new Date()) in ScenarioCards; HorizonTabs uses hidden sm:inline / inline sm:hidden; 14 tests confirm label formatting |
| SCNR-03 | 09-02 | One market rate slider above all cards; individual card calculators can override independently | SATISFIED | MarketRateSlider in ScenarioCards.tsx; MiniMortgageCalculator owns local rate state with reset-to-market link |
| SCNR-04 | 09-02 | Stay & Build Wealth shows home value, mortgage balance, equity at each horizon | SATISFIED | StayAndBuildCard horizonContent details array: "Home value", "Mortgage balance", "Total equity" at all 4 horizons |
| SCNR-05 | 09-02 | Sell & Move Up shows net proceeds ("cost to sell*"), mini mortgage calculator with 5/10/15/20% down payment presets | SATISFIED | SellAndMoveUpCard shows "cost to sell*" label; MiniMortgageCalculator with DOWN_PAYMENT_PRESETS=[0.05,0.10,0.15,0.20] |
| SCNR-06 | 09-01, 09-02 | Sell & Move Up uses 6% commission + 1% seller closing costs | SATISFIED | SELL_COMMISSION_RATE=0.06, SELL_CLOSING_RATE=0.01 in horizon-dates.ts; imported and used in SellAndMoveUpCard; test for constants passing |
| SCNR-07 | 09-03 | Stay & Invest shows HELOC available, monthly cost, rental investment framing, tax advantages callout (depreciation/OBBBA), CTAs to STR Analyzer and Josh & Jacqui | SATISFIED | StayAndInvestCard shows helocAvailable + monthlyHelocCost; investment framing text; olive OBBBA depreciation callout; STR_ANALYZER_URL link; Josh & Jacqui warm CTA |
| SCNR-08 | 09-03 | Move & Keep as Rental shows rental income, cash flow, effective interest rate as hero metric, mini mortgage calculator | SATISFIED | MoveAndKeepAsRentalCard shows monthlyRent/monthlyCashFlow in horizonContent; effective rate in olive box; MiniMortgageCalculator with onPaymentChange |
| SCNR-09 | 09-01, 09-03 | Effective interest rate: rate that produces out-of-pocket payment (new mortgage minus rental cash flow) on same loan amount | SATISFIED | calcEffectiveInterestRate bisection solver; MoveAndKeepAsRentalCard: outOfPocket = newMortgagePayment - rentalCashFlow; rate = calcEffectiveInterestRate(newLoanAmount, outOfPocket, 30) |
| SCNR-10 | 09-03 | Each scenario has contextual CTAs (lender connect, STR Analyzer, talk to Josh & Jacqui) | SATISFIED | StayAndBuildCard: no CTA (intentional — do-nothing path); SellAndMoveUpCard: lender tel CTA; StayAndInvestCard: STR Analyzer + Josh & Jacqui tel; MoveAndKeepAsRentalCard: Josh tel CTA |

**All 10 requirements: SATISFIED**

---

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `MiniMortgageCalculator.tsx` | 112 | HTML `placeholder="Enter price"` | Info | Standard HTML input placeholder — not a code stub |

No code stubs, empty implementations, or TODO/FIXME markers found in any phase 9 files. The single `placeholder` finding is an HTML input attribute for UX guidance, not a code placeholder.

**Pre-existing TypeScript errors (not introduced by phase 9):** `src/lib/calculations.test.ts`, `src/lib/comp-analyzer.test.ts`, and `src/lib/monsoon-parser.test.ts` have TypeScript index-signature and type-definition errors confirmed to exist before the first phase-9 commit (`cdd4562`, pre-phase-9). These do not affect runtime behavior and all 239 tests pass.

---

### Human Verification Required

The following items require manual browser testing to confirm goal achievement in a real environment:

#### 1. Date-based horizon tab rendering

**Test:** Open a homeowner page, scroll to the scenario cards section, observe the tab labels on each card.
**Expected:** Four pill tabs showing "Mar '26", "Mar '31", "Mar '36", "Mar '41" on desktop; "2026", "2031", "2036", "2041" on mobile (< 640px width).
**Why human:** CSS responsive classes (hidden sm:inline / inline sm:hidden) cannot be verified without a real browser rendering engine.

#### 2. MiniMortgageCalculator independent state

**Test:** Expand "Sell & Move Up", enter a home price and change the down payment. Then expand "Move & Keep as Rental", enter a different home price. Return to "Sell & Move Up".
**Expected:** Each calculator retains its own values independently. State in one card does not affect the other.
**Why human:** useState isolation across expanded/collapsed card lifecycles requires browser interaction to confirm.

#### 3. Effective interest rate live calculation

**Test:** Expand "Move & Keep as Rental". Enter a new home price (e.g., $600,000) with 20% down at the market rate. Observe the effective rate display.
**Expected:** An olive box appears showing a rate lower than the market rate (since rental cash flow offsets the new mortgage). The out-of-pocket amount displays below the rate.
**Why human:** Requires real interaction with the MiniMortgageCalculator + onPaymentChange callback firing to trigger the effective rate computation.

#### 4. MiniMortgageCalculator rate override and reset

**Test:** In any card with a MiniMortgageCalculator, change the interest rate field. Observe the P&I output update. Then click "Reset to market".
**Expected:** Rate changes reflect immediately in P&I. Reset restores the slider's market rate. The "Reset to market" link disappears when rate matches market.
**Why human:** Real-time state update behavior requires browser interaction.

---

### Gaps Summary

No gaps. All observable truths are verified, all artifacts are substantive and wired, all 10 requirements are satisfied, all 239 tests pass, and TypeScript compiles cleanly in production code.

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

_Verified: 2026-03-13T23:30:00Z_
_Verifier: Claude (gsd-verifier)_
