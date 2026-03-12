# Phase 2: Core Homeowner Page - Context

**Gathered:** 2026-03-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Branded, mobile-first page at `/h/[slug]` displaying all financial data — current value, equity gained, appreciation chart, earnings reframing, S&P comparison, rent-vs-own comparison, and net proceeds estimate. Uses scrolling narrative timeline with SB7 framing. Scroll animations, adaptive narrative by ownership duration, future scenarios, and CTA are Phase 3.

</domain>

<decisions>
## Implementation Decisions

### Page Hero & First Impression
- Full-width home photo as hero banner with homeowner name and address overlaid
- Falls back to branded gradient (olive/canyon) when no home photo is uploaded
- Small Live AZ Co logo in top corner of hero
- No data numbers in the hero — keep it clean and emotional, data reveals on scroll
- Personal note from Josh appears as intro paragraph below the hero, before data sections
- "Last updated" date as subtle stamp near the personal note

### Section Flow & Narrative Arc
- Build-up-to-punchline narrative arc — start with context, build to the financial climax
- Section order:
  1. Hero (name, home photo, address)
  2. Personal note intro
  3. "Your story begins..." (purchase price, purchase date)
  4. "Today, your home is worth..." (current value reveal)
  5. "That means you've built..." (equity gained)
  6. "Another way to look at it..." (earnings per week)
  7. "Your home vs the market" (appreciation chart + S&P callout)
  8. "Owning vs renting" (rent comparison)
  9. "If you sold today..." (net proceeds)
- Subtle alternating background colors (cream/white) between sections for visual rhythm — no hard dividers

### Appreciation Chart
- Area chart (Recharts AreaChart) with filled gradient (olive to gold)
- Interpolated smooth curve from purchase price to current value using annualized appreciation rate
- Annotated markers at purchase point ("Purchased: $X") and today ("Today: $X")
- S&P 500 comparison displayed as a separate callout card below the chart, not as a second line on the chart
- S&P callout format: "Your home gained $X vs $Y if you'd put the same money in the S&P 500"

### Data Presentation Style
- Hero number + narrative pattern for every data section: short SB7 intro line, big bold number, 1-2 sentences of supporting narrative
- Escalating narrative energy matching the arc: early sections calm and contextual, middle sections build confidence, final sections triumphant
- Earnings reframing: per-week as the hero number ("Your home earned $847/week")
- Rent vs own: side-by-side contrast — "If you rented: $X gone" vs "Because you own: $X built"
- Net proceeds: single headline number ("You'd walk away with $X") with expandable breakdown (gross, commission, closing costs, mortgage payoff)

### Claude's Discretion
- Exact SB7 copy for each section's intro and supporting narrative
- Loading state and error state design
- Exact spacing, typography sizing, and responsive breakpoints
- Chart tooltip behavior and axis formatting
- Empty/missing field handling (what to show when optional data wasn't entered)
- Whether to show all sections or conditionally hide sections with insufficient data

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `computeHomeownerMetrics()` in `src/lib/calculations.ts`: Main orchestrator returning all metrics needed for every section — equity, earnings, S&P comparison, rent vs own, net proceeds
- `applyDefaults()` in `src/lib/defaults.ts`: Fills in smart defaults for optional fields before calculation
- `rowToHomeownerRecord()` in `src/lib/supabase/transforms.ts`: Converts DB row to TypeScript interface
- `HomeownerRecord`, `HomeownerInput`, `HomeownerMetrics` types in `src/lib/types.ts`: All interfaces ready
- Supabase server client in `src/lib/supabase/server.ts`: For server-side data fetching with RLS
- Brand colors as CSS custom properties: `--olive`, `--canyon`, `--gold`, `--cream`, `--charcoal`
- Fonts loaded via next/font/google: Source Serif 4 (headings), DM Sans (body)

### Established Patterns
- Server components fetch data, pass to client components (see admin pages)
- Tailwind 4 CSS-first config with brand color utilities (`text-olive`, `bg-cream`, etc.)
- Supabase RLS: public SELECT allowed on published rows only — `/h/[slug]` can query directly
- Pure calculation functions with deterministic outputs — no side effects

### Integration Points
- `/src/app/h/[slug]/page.tsx`: Placeholder exists — this becomes the main page server component
- Data flow: Supabase query → `rowToHomeownerRecord()` → `applyDefaults()` → `computeHomeownerMetrics()` → render
- Recharts for charting (already in dependencies)
- Motion for future animations (Phase 3, but available)

</code_context>

<specifics>
## Specific Ideas

- The page should feel personal and cinematic, not like a report or dashboard
- "Build up to punchline" arc mirrors how Josh would tell the story in person — context first, then the impressive numbers
- Side-by-side rent vs own contrast uses SB7 problem/triumph framing — renting is the villain (money gone), owning is the victory (money built)
- Net proceeds expandable breakdown gives transparency without overwhelming the primary view
- Primary viewing context is a homeowner opening a text message link on their phone — every design choice should optimize for that moment

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-core-homeowner-page*
*Context gathered: 2026-03-12*
