# Phase 3: Narrative Experience - Context

**Gathered:** 2026-03-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Transform the existing data page into a cinematic scroll-driven narrative experience. Scroll-triggered animations reveal data sections theatrically, the story adapts its voice based on ownership duration, three future scenario sections spark conversations, milestone markers anchor the story in real moments, and a warm team CTA closes the page. The page structure and data sections from Phase 2 remain unchanged — this phase layers animation, adaptive narrative, scenarios, and the closing experience on top.

</domain>

<decisions>
## Implementation Decisions

### Scroll Animation Feel
- Cinematic & theatrical — sections reveal with noticeable motion, staggered element entrances, counters that build anticipation
- Hybrid stagger approach: key data sections (equity, earnings, net proceeds) get element-level stagger (SB7 intro fades in → hero number counts up → supporting copy appears, 200-300ms delays). Quieter sections (purchase story, personal note) animate as a unit
- Hero number counters use ease-in acceleration — start slow, accelerate to the final number, land with impact
- Animate once only — sections stay visible after first reveal, no replay on scroll-back
- Chart draws itself on scroll trigger (Claude's discretion on line-trace vs area-grow approach)

### Ownership Duration Adaptation
- Three narrative voices based on tenure:
  - Recent (under 2 years): forward-looking, "where you're headed" tone
  - Mid-term (2-7 years): balanced past and future
  - Long-term (7+ years): rich retrospective, "look what you've built" tone
- Copy changes only — same sections, same order, same structure for all three voices
- SB7 intro lines and supporting narrative text adapt per voice
- Future scenario cards get tenure-appropriate lead-in framing but same data presentation

### Future Scenario Presentation
- Card cluster approach — grouped in one section after Net Proceeds
- Section header: "Where could you go from here?" (or similar — Claude's discretion on exact copy)
- Three cards: Hold & Rent, Move-Up, Equity Play
- Cards stagger-animate in on scroll
- Tap-to-expand interaction on each card:
  - **Collapsed:** Hero metric + one-line implication
  - **Expanded:** 2-3 supporting details that make the scenario feel real and personal
  - Deliberately stops at an information cliff — the next question requires Josh
- Card content by scenario:
  - Hold & Rent: breakeven timeline, estimated rental income, monthly cash flow snapshot. Gap: "What would my actual numbers look like?"
  - Move-Up: down payment amount, price range it unlocks, estimated monthly payment. Gap: "What could I actually get in the areas I want?"
  - Equity Play: available equity, HELOC amount, what it could leverage as investment property down payment. Gap: "Is this actually a good move for my situation?"
- Cards are NOT full calculators — they tease the possibility, Josh explains the reality

### Milestone Markers
- Inline callouts within relevant sections (no side rail — too heavy on mobile)
- 2-3 milestones max per page, dynamically selected based on what's most impressive for each homeowner
- Possible milestones: purchase anniversary, equity thresholds ($50K, $100K), appreciation percentage, earnings milestones
- A 10-year owner with $200K equity gets different callouts than a 2-year owner with $30K
- Claude's discretion on selection logic and visual treatment

### CTA Design & Closing
- Branded closing section (canyon or olive background) set apart from data sections
- Josh + Jacqui photos side by side — team feel
- Warm narrative sign-off copy: "Ready to explore your options? Let's talk." (Claude's discretion on exact wording)
- Single `sms:` text link to Josh's number — "Text us" action
- NOT a sales pitch — feels like a natural conclusion to the narrative
- Photos sourced/styled at Claude's discretion

### Claude's Discretion
- Animation library choice (Framer Motion vs lighter alternatives — technical implementation decision)
- Exact SB7 copy for all three narrative voices across all sections
- Chart draw-in animation approach (line trace vs area grow)
- Milestone selection logic and threshold values
- Milestone visual treatment (badge, tag, highlighted callout)
- CTA photo styling and layout details
- Exact scenario card expanded content and copy
- Section transition timing and easing curves
- Mobile animation performance optimization

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `src/components/homeowner/HomeownerPage.tsx`: Client-side orchestrator rendering all 9 sections — Phase 3 wraps these in animation triggers
- `src/components/homeowner/SectionWrapper.tsx`: Alternating cream/white backgrounds — animation logic can be added here for section-level triggers
- `src/lib/calculations.ts`: `calcHoldAndRent()` (line 202), `calcMoveUp()` (line 250), `calcEquityPlay()` (line 269) — all return `ScenarioResult` with keyMetric, description, details
- `src/lib/types.ts`: `HomeownerMetrics` already includes `holdAndRent`, `moveUp`, `equityPlay` scenario data
- `src/components/homeowner/AppreciationChart.tsx`: Recharts AreaChart — needs scroll-trigger and draw animation
- Brand colors as CSS custom properties: `--olive`, `--canyon`, `--gold`, `--cream`, `--charcoal`

### Established Patterns
- Hero number + SB7 narrative intro + supporting copy pattern in every data section
- SB7 color coding: canyon for narrative intros, charcoal for hero numbers, charcoal/70 for supporting copy
- Server component fetches data → passes to client HomeownerPage orchestrator
- Pure calculation functions with deterministic outputs
- Mobile-first responsive with text-4xl/5xl/6xl scaling for hero numbers

### Integration Points
- `HomeownerPage.tsx` is already "use client" — animation hooks can be added directly
- `computeHomeownerMetrics()` already computes scenario data — no new calculations needed
- Motion library (if chosen) is already in package.json dependencies (listed but unused)
- Ownership duration computable from `purchaseDate` field on `HomeownerRecord`
- New components needed: ScenarioCards, MilestoneCallout, CTASection

</code_context>

<specifics>
## Specific Ideas

- The scenario cards should work like a movie trailer — enough to get excited, but you need Josh to get the full picture
- Each expanded card stops at a deliberate information cliff where the next question is "...but should I actually do this?" — and that's Josh's conversation
- Primary viewing context remains a homeowner opening a text message link on their phone — animations must feel smooth on mobile, not janky
- The CTA with both photos signals "you're working with a team" — personal and trustworthy
- Single text link to Josh's number keeps it simple from the homeowner's perspective

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 03-narrative-experience*
*Context gathered: 2026-03-12*
