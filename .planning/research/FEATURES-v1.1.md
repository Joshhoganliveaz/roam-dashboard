# Feature Landscape: v1.1 Enhanced Intelligence

**Domain:** Interactive homeowner engagement features
**Researched:** 2026-03-13

## Table Stakes for v1.1

Features that must work for the milestone to be considered complete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Value range selector with condition picker | Core v1.1 promise -- homeowner picks original/updated/remodeled | Medium | Three preset values entered by admin, homeowner selects one |
| Cascading recalculation on condition change | Without this, the condition picker is cosmetic only | Medium | Pure function re-run with new currentValue; all downstream metrics update |
| Cromford appreciation chart | Core v1.1 promise -- real market data | Medium | Recharts rendering of manually-entered city-level data from Supabase |
| Merged Investment Comparison section | Replaces two separate sections (S&P + Rent vs Own) | Low-Med | UI restructure + narrative rewrite; calculations already exist |
| 5/10/15 year projection timelines | Core v1.1 promise -- time horizon selection | Medium | New calc functions + UI toggle for time horizon |
| Market rate slider | Core v1.1 promise -- homeowner explores appreciation scenarios | Medium | Single slider feeding into projection calculations |
| Rate lock advantage callout | Shows value of their locked mortgage rate vs current market | Low | Simple comparison: locked rate vs market rate, show monthly savings |
| 4th combo scenario | "Keep, rent, and buy another" | Medium | Combines hold-and-rent + move-up logic into one projection |

## Differentiators

Features that elevate the experience beyond basic functionality.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Cascading ripple animation on condition change | Numbers visually cascade through the page when condition changes -- feels alive | Medium | Motion layout animations + staggered transitions |
| Information cliff removal | Full data shown (not truncated with "contact me" gates) | Low | Content restructuring, not technical |
| Cromford data comparison to homeowner's specific appreciation | "Your home appreciated X% vs the city average of Y%" | Low | Calc function comparing homeowner CAGR to Cromford city average |
| Enhanced Hold & Rent with 5/10/15yr cash flow projections | Shows cumulative wealth building over time, not just monthly snapshot | Medium | New projection functions extending existing holdAndRent calc |
| Enhanced Move-Up with sell-to-buy path education | Explains bridge loans, contingent offers, HELOC-as-downpayment | Low | Narrative content, minimal new calculation |
| Enhanced Equity Play with rate-adjusted HELOC costs | Shows HELOC payment at current rates, not just available amount | Low | Simple monthly payment calc at current HELOC rate |

## Anti-Features

Features to explicitly NOT build for v1.1.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Automated Tableau/Cromford scraping | Fragile, poorly supported in JS, serverless constraints, maintenance burden | Manual admin entry of city-level data into cromford_data table |
| Real-time MLS data integration | Out of scope per PROJECT.md; massive complexity for marginal benefit | Manual data input remains the pattern |
| AI-generated narrative for scenarios | Tempting but adds latency, cost, unpredictability to page load | Hand-crafted SB7 narrative templates with data interpolation |
| PDF export of the interactive page | Interactive features (sliders, condition picker) do not translate to PDF | Keep PDF as a future consideration; the URL IS the deliverable |
| Homeowner login/accounts | Adds auth complexity for no user benefit -- they get a link | Continue with public pages at /h/[slug] |
| Automated Cromford data refresh | Over-engineering for monthly data that changes slowly | Josh updates cromford_data when he notices new Cromford reports |

## Feature Dependencies

```
cromford_data Supabase table --> Cromford appreciation chart
value range columns in homeowners table --> Value range selector
Value range selector --> Cascading recalculation
Cascading recalculation --> Cascading ripple animation
New projection calc functions --> 5/10/15yr timelines
New projection calc functions --> Market rate slider
New projection calc functions --> 4th combo scenario
5/10/15yr timelines --> Enhanced Hold & Rent projections
Merged Investment Comparison --> Rate lock advantage callout (displayed within)
```

## MVP Recommendation for v1.1

Prioritize (must ship):
1. Value range selector + condition picker + cascading recalc (this IS the v1.1 interaction)
2. 5/10/15yr projection timelines (transforms static scenarios into explorable futures)
3. Cromford appreciation chart (real data, even if manually entered)
4. Merged Investment Comparison (cleans up page structure)

Defer if timeline is tight:
- Cascading ripple animation: can ship with simple opacity transitions first, polish later
- 4th combo scenario: most complex scenario to get right; the three existing scenarios are already strong
- OCR fallback for Cromford screenshots: manual entry works; OCR is a nice-to-have
- Enhanced Equity Play with HELOC rate adjustment: low user impact relative to effort

## Sources

- PROJECT.md: /Users/joshuahogan/.planning/PROJECT.md (HIGH confidence)
- Existing calculations.ts: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/calculations.ts (HIGH confidence)
- Existing types.ts: /Users/joshuahogan/Projects/homeowner-journey-map/src/lib/types.ts (HIGH confidence)
