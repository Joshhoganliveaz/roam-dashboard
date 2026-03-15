# Phase 7: Page Flow & Hook Rework - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning
**Source:** Discussion with Josh

<domain>
## Phase Boundary

Rework the page flow so the weekly paycheck hook leads, scenario cards move up closer to the top, and the data/proof sections move below. Simple CTA at bottom. Remove external links. Fix Cromford chart mobile spacing. Remove standalone net proceeds section.

</domain>

<decisions>
## Implementation Decisions

### Page Flow Reorder
- New section order:
  1. Hero + Weekly Paycheck hook ("Your home earned $X this week") — front and center, the first thing they see
  2. Personal note from Josh
  3. Home value + equity + condition picker
  4. Scenario cards ("Where could you go from here?") — moved UP from position 7 to position 4
  5. Appreciation chart (Cromford real data)
  6. Investment comparison (leverage story, S&P)
  7. Mortgage details
  8. Simple CTA — Josh & Jacqui contact info + signature/team photo + "Let's talk about your options"
- Rationale: Lead with the hook that makes them want to scroll, show the opportunity (scenarios) before the proof (data sections). The weekly paycheck reframes their home as an asset that's paying them.

### Weekly Paycheck Hook
- "Your home earned $X this week" is the headline hero number
- This already exists as DATA-05 but is currently buried in data sections
- Move it to the absolute top of the page, immediately visible on load
- This is the curiosity hook that earns the scroll — W-2 employees instantly relate to "earning a paycheck"
- The whole page narrative flows from this: "Your home is already paying you. Here's how to give yourself a raise."

### CTA Section
- Remove current CTA section
- Replace with simple closing: Josh & Jacqui's contact info, signature photos or team photo, "Let's talk about your options"
- Individual scenario cards will have their own CTAs — the bottom CTA is just a warm close

### External Links
- Remove "See a similar home that recently sold near you" link
- Do not link to anything that takes the homeowner off the page (Zillow, Redfin, etc.)
- Comp data will be shown inline in Phase 8 instead

### Net Proceeds
- Remove net proceeds from its current standalone section above the scenario cards
- Net proceeds only belongs in the "Sell & Move Up" scenario card (Phase 9) — it's a selling concept, not a general data point
- Showing it standalone feels like a sales pitch before they've decided to explore selling

### Cromford Chart Mobile Fix
- Date labels on the x-axis are too squished on mobile
- Fix spacing: show fewer labels on mobile (e.g., every other quarter, or just years), or rotate labels
- This is a styling/responsive fix, not a data change

### Claude's Discretion
- How to implement the section reorder (component prop changes, page layout adjustments)
- Exact styling of the weekly paycheck hero treatment
- How to handle the CTA photo (signature photos vs team photo — Josh to provide asset)
- Specific Cromford chart label strategy for mobile

</decisions>

<specifics>
## Specific Ideas

- Weekly paycheck framing: "Your home earned you $327 this week" — specific dollar amount, conversational tone
- CTA: "Let's talk about your options" with Jacqui and Josh's contact info
- The weekly paycheck ties the whole page together: hook at top → data explains how → scenarios show how to amplify it

</specifics>

<deferred>
## Deferred Ideas

- Condition picker rework ("How does your home compare to your neighbors?") — Phase 8
- Inline comp data from CSV — Phase 8
- Scenario cards rework (four new scenarios, slider, calculators) — Phase 9

</deferred>

---

*Phase: 07-future-scenarios-rework*
*Context gathered: 2026-03-13 via discussion with Josh*
