# Phase 8: Condition Picker & Comp Context - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning
**Source:** Discussion with Josh

<domain>
## Phase Boundary

Rework the condition picker from "Select your home's condition" to "How does your home compare to your neighbors?" Show 2-3 nearby recent sales inline from CSV comp data. Tighten the value range so selections don't wildly change numbers. Keep it minimal and clean — not busy.

</domain>

<decisions>
## Implementation Decisions

### Condition Picker Reframe
- Change from "Select your home's condition" (Mostly original / Made some updates / Completely remodeled) to "How does your home compare to your neighbors?"
- This taps into natural curiosity and competitiveness rather than asking them to judge their own home negatively
- Homeowners always think their home is worth more — the range should reflect this reality

### Value Range
- Tighten to +/-3-5% from base value (currently the range between low comp and high comp can be too wide)
- The number shouldn't move too much regardless of selection — it's more about engagement than precision
- Example: $500K base → range of ~$485K to ~$515K, not $450K to $550K

### Inline Comp Data
- Show 2-3 nearby recent sales from the CSV data Josh already uploads
- Each comp: address, sold price, maybe one photo (if available from MLS data)
- Minimal styling — small cards or a simple list
- NO external links — everything stays on-page
- The comps give context for their selection without overwhelming the page
- Josh is already running CSV files for comps with addresses and sold prices

### Keep It Clean
- Josh specifically called out not wanting it to get "overly busy"
- The comp data should inform their choice, not overwhelm the page
- A quick glance, pick where they fit, move on

### Claude's Discretion
- Exact UI layout for comp display (cards vs list vs other)
- How to source comp photos (if available in CSV or MLS data)
- How the tighter range maps to the selection options
- Whether to keep three options or adjust the number of choices

</decisions>

<specifics>
## Specific Ideas

- "How does your home compare to your neighbors?" as the framing question
- 2-3 comps max — enough for context, not enough to overwhelm
- The condition picker is as much an engagement tool as a value adjustment tool

</specifics>

<deferred>
## Deferred Ideas

- Zillow Zestimate API integration for auto-filling rent estimates in admin dashboard — future admin enhancement
- More sophisticated comp selection/filtering — v2

</deferred>

---

*Phase: 08-condition-picker-comp-context*
*Context gathered: 2026-03-13 via discussion with Josh*
