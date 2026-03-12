# Feature Landscape

**Domain:** Personalized homeowner journey pages / narrative equity storytelling
**Researched:** 2026-03-11

## Table Stakes

Features users expect. Missing = product feels incomplete or untrustworthy.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Current home value estimate | Every competitor (Homebot, Zillow, Hometap, Movoto) leads with this. Homeowners open the page expecting to see a number. | Low | Manual input for v1 is fine -- Homebot's AVM accuracy is a top complaint, so a curated value from Josh actually has more credibility. |
| Equity gained since purchase | The single most emotionally resonant data point. Purchase price vs current value = the "wow" moment. | Low | Simple math: current value - remaining mortgage balance. Already modeled in STR Analyzer's calculator.ts. |
| Agent branding and contact info | Every touchpoint tool brands the sending agent. Homebot, Buyside, and CMA tools all do this. If it looks generic, homeowners won't connect it to Josh. | Low | Live AZ Co color palette, logo, Josh's photo and contact -- already defined in CLAUDE.md conventions. |
| Mobile-responsive design | PROJECT.md says most open via text. Homebot's digest is mobile-first. Every competitor is mobile-optimized. | Medium | Must be designed mobile-first, not adapted after. Scroll-driven narrative actually works better on mobile than dashboard grids. |
| Shareable unique URL | The whole distribution model depends on this. Text a link, homeowner opens it. Homebot uses email; this uses direct links -- that's an advantage. | Low | Unique slug per homeowner (e.g., `/journey/smith-family`). No auth required. |
| Appreciation visualization | Homebot shows home value over time. Buyside shows equity dashboards. Homeowners expect to see their gains visually, not just as a number. | Medium | Animated counter or chart showing value growth. Does not need to be interactive -- a curated visual is on-brand. |
| Net proceeds estimate ("What would I walk away with?") | Homebot's Equity Explorer and every CMA tool includes this. Homeowners want to know their real number after commissions, closing costs, mortgage payoff. | Medium | Already built in CMA tool's equity calculations. Reuse the math, present it narratively. |
| Call-to-action to contact agent | Every drip marketing tool and personalized page funnels to a CTA. Without it, the page is informational but not actionable. | Low | Soft CTA, not pushy. "Ready to explore your options? Let's talk." with phone/text link. SB7-aligned: the guide offers a plan. |

## Differentiators

Features that set the Homeowner Journey Map apart from Homebot, Zillow, and generic CMA reports. These are where the product wins.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Scrolling narrative timeline (scrollytelling) | **The core differentiator.** No competitor tells a story. Homebot is a dashboard/digest. Zillow is a lookup tool. CMA reports are PDFs. This is a cinematic, scroll-driven experience that unfolds the homeowner's journey from purchase to present to future. Scrollytelling is a proven web design pattern (GSAP ScrollTrigger, parallax, reveal animations) but nobody applies it to real estate homeowner engagement. | High | This is the hardest feature to build and the hardest for competitors to copy. Use scroll-triggered animations (fade-in sections, animated counters, parallax backgrounds). GSAP ScrollTrigger is free since Webflow's acquisition and is the industry standard. |
| SB7 narrative framing | Every section names a problem or promise. The homeowner is the hero. Live AZ Co is the guide. This emotional framing makes the page feel personal and empowering, not transactional. No competitor does narrative framing -- they show data without story. | Medium | Not a technical feature -- it's a content/copy framework applied to every section. The complexity is in writing the templates, not building the UI. Already proven in the CMA tool. |
| Ownership-duration-adaptive narrative | Recent buyers (0-2 years) get a forward-looking "where you're headed" story emphasizing future projections. Mid-term owners (2-5 years) get a balanced past/future view. Long-term owners (5+ years) get a rich retrospective with "look how far you've come" beats. No competitor adapts tone by ownership stage. | Medium | Conditional content rendering based on purchase date. 3 narrative tiers with different section ordering and copy emphasis. |
| Reframing comparisons | "Your home earned $X per week while you slept." "Your equity grew faster than the S&P 500." "You've saved $X vs renting." These reframings turn abstract equity into visceral, shareable moments. Homebot shows raw numbers; this product contextualizes them. | Medium | Requires: S&P 500 return data (historical averages are fine), local rent estimates (manual input), and date math for per-week/per-month breakdowns. The math is simple; the copy design is the value. |
| Future scenario projections (3 paths) | **Hold & Rent:** When rental income would cover the mortgage. **Move-Up:** Net proceeds + what that buys in today's market. **Equity Play:** HELOC potential, investment property leverage. Homebot's Equity Explorer shows opportunities as modules. This product presents them as narrative branches -- "Here's where your story could go next." | High | Heavy calculation work, but STR Analyzer already has mortgage, amortization, rental projection, and HELOC math. Reuse those formulas. Each scenario is a scrollytelling section with its own visual treatment. |
| Animated data reveals on scroll | Numbers count up, charts draw themselves, equity bars grow as you scroll past them. This transforms static data into moments of delight. No real estate tool does this -- it's borrowed from editorial data journalism (NYT, Bloomberg, Pudding). | Medium | GSAP ScrollTrigger handles this natively. CountUp.js or similar for number animations. CSS transitions for chart drawing. Performance-sensitive on mobile -- test heavily. |
| Home anniversary / milestone markers | Timeline markers for purchase anniversary, equity milestones ($50K, $100K), market events. These create natural re-send moments: "Happy 3-year home anniversary -- look at your journey." No competitor ties engagement to ownership milestones. | Low | Date math from purchase date. Milestone triggers are simple threshold checks. The value is in the re-engagement hook, not the technical implementation. |
| Print/share-optimized social card | When shared via text/social, the link preview shows a branded card: "The Smith Family's Home Journey -- See how their home has grown." Open Graph meta tags with dynamic image generation. | Medium | Dynamic OG image generation (e.g., @vercel/og or Satori). Shows homeowner name, equity gain headline, Live AZ Co branding. Makes the link worth clicking when forwarded. |

## Anti-Features

Features to explicitly NOT build. These would dilute the product's differentiation or add complexity without value.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Interactive calculators or user-adjustable inputs | PROJECT.md explicitly scopes this out. This is a curated experience, not a tool. Homebot lets users adjust their home value -- that's useful but makes it feel like a utility, not a gift. The Homeowner Journey Map should feel like something Josh crafted for them. | Present the numbers as authoritative. If the homeowner wants to explore scenarios, that's the conversation starter -- they call Josh. |
| Automated recurring email updates | Homebot's entire model is monthly automated digests. Competing on automation means competing with Homebot's infrastructure (data vendors, AVMs, email delivery). Instead, each page is a moment-in-time touchpoint that Josh sends personally. | Manual send via text/email. The personal touch IS the differentiator. Josh can re-send updated pages at anniversaries or when market shifts create a new story. |
| Real-time MLS data integration | PROJECT.md scopes this out for v1. API complexity, data licensing, and accuracy concerns (Homebot's #1 complaint is inaccurate AVMs). Manual data input at 20-30 pages/month is manageable. | Admin input form where Josh enters address, purchase price, current value, purchase date, and a few data points. Under 5 minutes per page. |
| User accounts or login | Adding auth creates friction. The homeowner gets a link, opens it, sees their story. No signup, no password, no app download. This is how texts work -- tap and see. | Public unique URLs. Security through obscurity is fine -- these pages contain no sensitive financial data (no SSN, no account numbers, no exact mortgage balance beyond what the homeowner already knows). |
| Neighborhood market reports | Homebot and Zillow already do this well. Building market analytics means maintaining data feeds. The Homeowner Journey Map is about YOUR home, not the market. | Include 1-2 contextual market data points ("Homes in [area] appreciated X% since you bought") as supporting narrative, not as a standalone section. |
| PDF export / printable report | The CMA tool already generates print-ready reports. This product is a web experience. Making it printable would compromise the scroll-driven design and duplicate existing tooling. | If a homeowner wants a printed report, Josh can generate a CMA. The Journey Map is the digital complement. |
| Homeowner self-service page creation | The value is that Josh creates each page personally. Self-service would turn it into a commodity tool. The 5-minute admin input time is fast enough for 20-30/month volume. | Admin-only creation interface. Josh or team enters data, page generates, link gets sent. |
| Complex data dashboards or charts | Homebot is a dashboard. Zillow is a dashboard. This is NOT a dashboard. It's a narrative. Multiple chart types, toggles, and filters break the storytelling flow. | Use simple, beautiful, single-purpose visualizations: one equity growth line, one animated counter, one comparison bar. Let the scroll do the work. |

## Feature Dependencies

```
Shareable URL system → Everything (pages need to exist at URLs)
  |
  ├── Admin input form → Page generation (need data to create pages)
  |     |
  |     ├── Equity calculation engine → Appreciation visualization
  |     |     |
  |     |     ├── Reframing comparisons (depends on equity math)
  |     |     └── Net proceeds estimate (depends on equity math)
  |     |
  |     └── Ownership duration detection → Adaptive narrative (needs purchase date)
  |
  ├── Scrollytelling framework (GSAP) → Animated data reveals
  |     |
  |     └── Timeline layout → Milestone markers
  |
  ├── Future scenario calculations → Scenario projection sections
  |     (can reuse STR Analyzer math)
  |
  ├── SB7 copy templates → All narrative content
  |
  └── OG image generation → Social sharing card
```

## MVP Recommendation

**Phase 1 -- Ship to validate (core narrative experience):**
1. Admin input form (address, purchase price, current value, purchase date, client name)
2. Scrolling narrative timeline with SB7 framing
3. Equity gained since purchase (animated reveal)
4. Appreciation visualization (simple line or growth bar)
5. Reframing comparisons (home vs S&P 500, equity per week, vs renting)
6. Ownership-duration-adaptive narrative (3 tiers)
7. Agent branding and CTA
8. Mobile-first responsive design
9. Shareable unique URL

**Phase 2 -- Deepen the story:**
1. Future scenario projections (Hold & Rent, Move-Up, Equity Play)
2. Net proceeds estimate
3. Home anniversary / milestone markers
4. Dynamic OG social sharing card

**Defer:**
- Print/PDF anything: CMA tool handles print. This is digital-native.
- Market data integration: Manual input is fine at current volume. Revisit if volume exceeds 50/month.
- Automated sending: Josh's personal touch is the differentiator. Automation is a v3 concern if ever.

**Rationale:** Phase 1 delivers the "wow" moment -- a homeowner opens a link and sees their home's story told beautifully. That alone is differentiated from everything in the market. Phase 2 adds the conversation starters (scenarios that make homeowners call Josh). The dependency chain supports this ordering: equity math must exist before scenarios can build on it.

## Sources

- [Homebot Home Digest Overview](https://help.homebotapp.com/en/articles/3975961-overview-of-the-home-digest) -- Homebot's core feature set and modules
- [Homebot Q4 2025 Product Update](https://homebot.ai/blog/q4-2025-product-update) -- Latest Homebot features and integrations
- [Homebot Q1 2025 Product Update](https://homebot.ai/blog/what-we-launched-in-q1-2025) -- Listing alerts, pre-qualification features
- [Homebot vs Alternatives (2026)](https://homebot.ai/blog/homebot-vs-alternatives-choosing-the-right-mortgage-marketing-tool) -- Competitive landscape from Homebot's perspective
- [Homebot Review - The Close](https://theclose.com/homebot-reviews/) -- Agent-focused pros/cons analysis
- [Homebot Review - Unify](https://unifyrealestate.com/market-reports/homebot/) -- Features, pricing, and AVM accuracy concerns
- [Hometap Home Equity Dashboard](https://www.hometap.com/dashboard) -- Competitor equity visualization approach
- [Buyside Home Valuation Pages](https://www.housingwire.com/articles/buyside-launches-a-new-home-valuation-pages-feature/) -- Competitor personalized page feature
- [Movoto Homeowner Dashboard](https://www.movoto.com/homeowner/) -- Competitor homeowner tools
- [GSAP ScrollTrigger](https://gsap.com/scroll/) -- Industry-standard scroll animation library (free since Webflow acquisition)
- [BSMNT Scrollytelling (React + GSAP)](https://github.com/basementstudio/scrollytelling) -- React wrapper for scroll-driven narratives
- [Web Design Trends 2025: Scrollytelling](https://whales.marketing/blog/web-design-trends-of-2025-scrollytelling-colors-fonts-and-a-bit-of-ai/) -- Scrollytelling as dominant design pattern
- [Luxury Real Estate Website Design Trends 2026](https://www.dmrmedia.org/blog/Real-Estate-Website-Design-Trends) -- Cinematic storytelling, scroll-triggered animations
- [Scrolling Visualizations](https://vallandingham.me/scroll_talk/examples/) -- Data visualization scrollytelling patterns
