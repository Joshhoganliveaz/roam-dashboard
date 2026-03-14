# Phase 6: Investment Comparison + Narrative Cleanup - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning

<domain>
## Phase Boundary

The separate S&P 500 callout and Rent vs Own sections merge into a single Investment Comparison section built around the leverage story. A rate lock callout appears only when home value has declined (silver lining). All page sections present full information — scenario card cliff questions removed, tap-to-expand interaction preserved. The page gets tighter (net section reduction) and more empowering.

</domain>

<decisions>
## Implementation Decisions

### Merged Section Layout
- Single "Investment Comparison" section replaces both SP500Callout and RentVsOwn as standalone sections — net reduction in page sections
- Section placement: after Equity Gained, before Appreciation Chart (new flow: Equity → Investment Comparison → Earnings Reframe → Chart → Net Proceeds → Scenarios)
- Leverage hero number is the centerpiece — bold standalone callout card, designed to be screenshot-worthy
- Personalized leverage callout beneath hero: "$25K down controlling a $500K asset = $125K in equity built"

### S&P 500 Role
- Demoted from standalone section to conditional one-liner beneath the leverage callout
- Only shown when home outperformed S&P — "Even the S&P would've only returned $X on that same $25K"
- Hidden entirely when S&P outperformed the home — no need to highlight a negative
- Not a hero number, not a comparison card — purely a reinforcing detail

### Rent vs Own Handling
- Rent comparison only shown for homeowners who purchased within the last 2 years
- For under-2-year owners: keeps the villain/triumph SB7 contrast ("If you rented: money gone" vs "Because you own: equity built")
- For owners 2+ years: rent contrast removed entirely — they already know renting is throwing money away
- When shown, integrated within the Investment Comparison section (not a separate section)

### Rate Lock Callout
- Only appears when home value has actually gone down (negative appreciation) AND the homeowner has a below-market rate
- NOT shown when home has appreciated — Josh explicitly does not want "golden handcuffs" discouraging homeowners from moving
- Quantifies monthly and annual savings vs current market rates
- Current market rate sourced automatically (research Mortgage News Daily, Freddie Mac PMMS, or similar — researcher to determine best approach)
- When shown, takes a silver lining optimism tone: "Markets move in cycles, but your locked rate is saving you $X/month"

### Information Cliff Removal
- Scenario cards (Hold & Rent, Move-Up, Equity Play): keep tap-to-expand interaction pattern
- Remove cliff questions entirely from expanded state — no "What would my actual numbers look like?" teaser
- When expanded, show ALL details — homeowners get the full picture
- All three cards expandable independently (not accordion — multiple can be open at once)
- Make it visually clear that cards are interactive and homeowners should choose which scenarios interest them
- Net proceeds breakdown: stays expandable (not an information cliff — it's supplementary detail)

### Narrative & SB7 Voice
- Investment Comparison section uses triumphant reveal tone — "Here's what most people miss about your home"
- Builds to the leverage number as a dramatic reveal — this is the "wow, I didn't realize that" moment
- Voice adapts by ownership duration (consistent with Phase 3 pattern):
  - Recent (under 2 yrs): "You're already ahead — here's how"
  - Mid-term (2-7 yrs): "Look what's been building quietly"
  - Long-term (7+ yrs): "This is the payoff of patience"
- Down market / rate lock tone: silver lining optimism — acknowledges reality, finds the genuine win
- SB7 framing: homeowner is the hero who made a smart leveraged investment, Josh is the guide who helps them see it

### Claude's Discretion
- Exact SB7 copy for leverage section intro, supporting narrative, and tenure-adapted variants
- Leverage callout card visual design (colors, borders, typography within brand guidelines)
- S&P one-liner phrasing
- Rate lock callout visual treatment
- How rent contrast is visually integrated within the merged section for under-2-year owners
- Cascade animation delay for the new section's position in the page flow
- Threshold for "meaningfully below market" rate difference (e.g., 0.5%+ gap)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `SP500Callout.tsx`: Current S&P comparison component — will be replaced, but logic for conditional display (homeWon check) is reusable
- `RentVsOwn.tsx`: Current rent-vs-own with villain/triumph layout — villain/triumph cards can be adapted for under-2-year owners within merged section
- `ScenarioCard.tsx`: Tap-to-expand pattern with `cliffQuestion` field — remove cliff question, keep expand/collapse logic
- `calculations.ts`: `calcSP500Comparison()` returns `{ homeReturn, sp500Return, difference, isRealData }` — reuse for conditional S&P line
- `calculations.ts`: `calcRentVsOwn()` returns `{ totalRentWouldHavePaid, equityBuilt, netBenefit }` — reuse for under-2-year rent contrast
- `AnimatedSection` + `CountUpNumber` + `useCounterRoll`: Cascade animation infrastructure from Phase 5

### Established Patterns
- Pre-compute condition variants server-side, switch client-side via useState
- Hero number + SB7 narrative intro + supporting copy pattern in every data section
- SB7 color coding: canyon for narrative, charcoal for hero numbers, charcoal/70 for supporting
- Cascade delays: 0/500/1000/1500/2000/2500ms across sections — will need re-sequencing with new section order
- Three narrative voices by tenure (recent/mid-term/long-term) already implemented in Phase 3
- Condition picker triggers cascade recalculation — new section must participate in cascade

### Integration Points
- `HomeownerPage.tsx`: Remove SP500Callout and RentVsOwn imports, add InvestmentComparison component, reorder sections
- `calculations.ts`: New leverage calculation function (ROI based on down payment vs current equity)
- `calculations.ts`: Rate lock savings calculation (monthly payment at locked rate vs current market rate)
- Admin form or global config: Current market rate input (or auto-fetched)
- `ScenarioCard.tsx`: Remove `cliffQuestion` prop and related rendering
- Supabase schema: May need `current_market_rate` column or global settings table

</code_context>

<specifics>
## Specific Ideas

- The leverage callout should feel like a pull quote — visually distinct, screenshot-worthy, designed to make homeowners text it to friends
- Josh explicitly does NOT want the rate lock to create "golden handcuffs" — it's only a consolation when appreciation is negative, never a reason to stay put
- The S&P comparison is an investor concept that doesn't resonate with most homeowners — demote it to a supporting one-liner at most
- The rent comparison is obvious to anyone who's owned for more than a couple years — only valuable for recent buyers still adjusting to mortgage payments
- Scenario cards should feel clearly interactive — homeowners need to understand they should choose and explore

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 06-investment-comparison-narrative-cleanup*
*Context gathered: 2026-03-13*
