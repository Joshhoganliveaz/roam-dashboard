# Phase 9: Scenario Cards Rework - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning
**Source:** Discussion with Josh

<domain>
## Phase Boundary

Replace the current three scenario cards + combo with four new scenarios that match real homeowner decision paths. Add market rate slider, dynamic date-based horizons, mini mortgage calculators, and the effective interest rate concept.

</domain>

<decisions>
## Implementation Decisions

### Four New Scenarios
The old scenarios (Hold & Rent, Move-Up, Equity Play, Combo) are replaced with:

1. **Stay & Build Wealth** — Do nothing, let appreciation and equity grow
   - They live: in current home
   - Their current home: appreciates, mortgage pays down
   - Shows: home value, mortgage balance, equity at each horizon
   - Narrative: "Your home is already working for you"
   - This is the long game — builds trust now, sets up HELOC or sell conversation later

2. **Sell & Move Up** — Cash out equity, buy a bigger/better home
   - They live: in new home
   - Their current home: sold
   - Shows: net proceeds (labeled "cost to sell*" with "(title fees & broker fees)"), mini mortgage calculator
   - Commission: 6%, Seller closing costs: 1%
   - Mini mortgage calculator: 30-year fixed, rate defaults to market rate (adjustable), down payment presets of 5% | 10% | 15% | 20% (buttons, not free-form), new home price input, output is P&I monthly payment
   - CTA: "Want to know exactly what you qualify for? Let's connect you with a lender"
   - Note: "Purchasing power" was rejected — it depends on DTI which only a lender can determine

3. **Stay & Invest** — Tap HELOC to buy a rental/investment property
   - They live: in current home (stay put)
   - Their current home: funds an investment property via HELOC
   - Shows: available HELOC at each horizon, monthly HELOC cost (interest-only at market rate)
   - Investment angle: frame HELOC as down payment on a rental property
   - Tax advantages callout: W-2 earners can offset income through depreciation (cost segregation, bonus depreciation)
   - CTAs: "Try our STR Analyzer" (links to Josh's STR Analyzer tool), "Josh and Jacqui invest in short-term rentals themselves — let's talk about how your equity could work for you"

4. **Move & Keep as Rental** — Current home becomes rental, buy new primary residence
   - They live: in new home
   - Their current home: becomes rental property generating income
   - Shows: rental income (from monthlyRentEstimate in admin data), cash flow (rent minus old mortgage), effective interest rate as HERO METRIC
   - Mini mortgage calculator: new home price input, interest rate defaults to market (adjustable), down payment presets 5% | 10% | 15% | 20%, shows P&I payment
   - HELOC for down payment is optional — they might use savings or other funds
   - **Effective Interest Rate concept**: Take new home mortgage payment, subtract rental cash flow = out-of-pocket payment. What interest rate would produce that out-of-pocket payment on the same loan? That's the effective rate. Example: $3,200 mortgage - $800 cash flow = $2,400 out-of-pocket. The rate that makes a $3,200 payment equal $2,400 is their effective rate (~4.2% vs actual 6.5%). This reframes the "rates are too high" objection.

### Market Rate Slider
- One slider above all four scenario cards
- Controls the baseline market rate for all calculations
- Individual mini mortgage calculators inside cards can override the rate independently
- Range: 3% to 10% (keeping from old plan)
- Default: today's actual market rate

### Dynamic Date Horizons
- Horizons are calculated from the date the homeowner views the page
- Four tabs: current date, +5 years, +10 years, +15 years
- First choice label format: **Mar '26 | Mar '31 | Mar '36 | Mar '41** (3-letter month abbreviation + 2-digit year)
- Fallback if too tight on mobile: **2026 | 2031 | 2036 | 2041** (years only)
- Build with month abbreviation first, test on mobile, fall back if needed
- "Today/Now" horizon shows current numbers — grounds them in reality before showing projections

### Rental Income Source
- Rental estimate comes from existing admin data (monthlyRentEstimate field)
- Future admin enhancement: Zillow Zestimate API to auto-fill rent estimate (not this phase)
- Homeowner does NOT adjust the rental estimate on their page — it's set by Josh in admin

### Claude's Discretion
- Exact projection calculation implementations (reuse/extend existing calc functions)
- Component architecture for the four new scenario cards
- Styling details within branded guidelines
- How the effective interest rate is calculated (reverse mortgage payment formula)
- How to handle edge cases (zero mortgage, negative equity, etc.)

</decisions>

<specifics>
## Specific Ideas

- Effective interest rate is the hero metric for Move & Keep as Rental — it's the conversation changer
- "Your home earned you $327 this week" (from Phase 7) primes them for the investing scenarios
- STR Analyzer cross-sell is a natural fit for Stay & Invest scenario
- Josh and Jacqui are both avid real estate investors with short-term rentals — this is authentic, not a sales pitch
- Down payment presets as buttons (5% | 10% | 15% | 20%) not a slider or free input — clean and simple

</specifics>

<deferred>
## Deferred Ideas

- Zillow Zestimate API for rent estimate auto-fill in admin — future admin enhancement
- DTI-based affordability calculation — that's the lender's job, the CTA drives them there

</deferred>

---

*Phase: 09-scenario-cards-rework*
*Context gathered: 2026-03-13 via discussion with Josh*
