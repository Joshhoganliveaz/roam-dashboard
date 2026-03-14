---
gsd_state_version: 1.0
milestone: v1.1
milestone_name: Enhanced Intelligence
status: executing
stopped_at: Completed 07-01-PLAN.md
last_updated: "2026-03-14T14:12:37.304Z"
last_activity: 2026-03-14 -- Completed 07-01 (Hero Paycheck Hook & Page Reorder)
progress:
  total_phases: 10
  completed_phases: 9
  total_plans: 25
  completed_plans: 24
  percent: 96
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-13)

**Core value:** Show homeowners the full journey of their home -- where they've been, where they are, and where they could go -- so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.
**Current focus:** Phase 7 - Future Scenarios Rework (07-01 complete)

## Current Position

Phase: 7 of 10 (Future Scenarios Rework)
Plan: 1 of 2
Status: executing
Last activity: 2026-03-14 -- Completed 07-01 (Hero Paycheck Hook & Page Reorder)

Progress: [██████████] 96%

## Performance Metrics

**Velocity:**
- Total plans completed: 13 (v1.0)
- Average duration: ~2.8 min
- Total execution time: ~0.60 hours

**By Phase (v1.0):**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 1. Foundation + Admin | 3 | ~8 min | ~2.7 min |
| 2. Core Homeowner Page | 3 | ~9 min | ~3.0 min |
| 2.1 Financial Data Audit | 4 | ~10 min | ~2.5 min |
| 3. Narrative Experience | 2 | ~6 min | ~3.0 min |
| 4. Documentation Cleanup | 1 | ~3 min | ~3.0 min |
| 5. Data Foundation | 3/5 | ~14 min | ~4.7 min |
| Phase 07 P02 | 1min | 2 tasks | 3 files |
| Phase 09 P02 | 6min | 2 tasks | 8 files |
| Phase 09 P03 | 4min | 2 tasks | 4 files |
| Phase 07 P01 | 1min | 2 tasks | 3 files |

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- v1.1: Pre-compute 3 condition variants server-side, switch client-side via useState (not Zustand)
- v1.1: Cromford data via manual screenshot upload + OCR, not automated scraping
- v1.1: Schema adds nullable value_low/value_high columns for backward compat with 200+ pages
- v1.1: Market rate slider is the only client-side recalculation; everything else swaps pre-computed sets
- v1.1: Combo scenario composes existing calculation functions, built last
- v1.1: Claude Vision OCR uses raw fetch (no SDK), deterministic shaped curve seeded from purchasePrice
- v1.1: getConditionValues clamped to -3%/+5% from baseValue (asymmetric, SB7-informed)
- v1.1: Graceful fallback -- when valueLow/valueHigh are null, all 3 variants use currentValue
- v1.1: Counter-roll uses Motion animate() imperative API with easeInOut for odometer feel
- v1.1: Cascade timing: 0/500/1000/1500/2000/2500ms across 6 sections for ~3s total
- v1.1: AppreciationChart uses key={condition} for smooth remount morph, shaped fallback replaces linear curve
- v1.1: FRED API MORTGAGE30US for current rate with DEFAULTS.interestRate fallback
- v1.1: LeverageROI returns 0 for zero/negative down payment (cash purchase guard)
- v1.1: calcRateLockSavings reuses calcMortgagePayment for consistent amortization math
- v1.1: S&P one-liner conditional on home outperforming S&P (hidden when S&P wins)
- v1.1: Rent contrast conditional on sub-24-month ownership only
- v1.1: Rate lock callout triple-gated: negative appreciation AND rate gap >= 0.5%
- v1.1: Kept SP500Callout.tsx and RentVsOwn.tsx as dead code for reference (not deleted)
- [Phase 07]: Refi in Hold & Rent recalculates remaining balance at marketRate for new 30yr term
- [Phase 07]: ScenarioCards owns slider state via useState, not lifted to HomeownerPage
- [Phase 07]: ScenarioCard uses children prop for custom expanded content (rate lock, sell-to-buy edu)
- [Phase 07]: HorizonTabs stopPropagation prevents card collapse on tab click
- [Phase 07]: Inline tel/sms links instead of button CTA for warm SB7 guide closing
- [Phase 07]: Rotated XAxis labels (-45deg) + reduced interval + bottom margin for mobile Cromford chart readability
- [Phase 08]: conditionCompLinks enriched with optional soldPrice, plumbed server->HomeownerPage->CurrentValue
- [Phase 08]: soldPrice optional for backward compat with 200+ existing records
- [Phase 09]: Card-local sell costs (6%+1%) separate from global DEFAULTS (5%+2%)
- [Phase 09]: Bisection tolerance $0.01, 100 max iterations for calcEffectiveInterestRate
- [Phase 09]: HorizonYears = 0|5|10|15, NEW_PROJECTION_HORIZONS includes year 0
- [Phase 09]: StayAndBuildCard defaults to 5yr tab (not Now) per research pitfall #6
- [Phase 09]: MiniMortgageCalculator owns all state locally via useState -- never lifted to parent
- [Phase 09]: ScenarioCard.horizonContent uses Partial<Record<HorizonYears, ...>> for backward compat
- [Phase 09]: SellAndMoveUpCard CTA uses inline tel link per SB7 warm pattern
- [Phase 09]: StayAndBuildCard defaults to 5yr tab (not Now) per research pitfall #6
- [Phase 09]: MiniMortgageCalculator owns all state locally via useState -- never lifted to parent
- [Phase 09]: ScenarioCard.horizonContent uses Partial<Record<HorizonYears, ...>> for backward compat
- [Phase 09]: Effective rate only shown when rental cash flow is positive -- negative cash flow gets advisory message
- [Phase 09]: MoveAndKeepAsRentalCard defaults to Now tab -- effective rate best grounded in current numbers
- [Phase 09]: STR Analyzer URL as placeholder constant pointing to Vercel primary deploy
- [Phase 09]: ComboScenario file kept as dead code per convention, only import removed from ScenarioCards
- [Phase 07]: PaycheckHook as separate component below hero with charcoal background for readability (user design feedback)
- [Phase 07]: HeroSection kept clean: photo + name + address + branding only, no number overlays

### Pending Todos

None yet.

### Blockers/Concerns

- Combo scenario requirements need Josh's input (HELOC vs sale proceeds for down payment)
- Condition value mapping (original=low, updated=40th pct, remodeled=high) needs Josh's validation
- Cromford data city coverage needs verification (which Phoenix metro cities have data)

## Session Continuity

Last session: 2026-03-14T14:10:10.859Z
Stopped at: Completed 07-01-PLAN.md
Resume file: None
