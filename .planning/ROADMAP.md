# Roadmap: Homeowner Journey Map

## Overview

This roadmap delivers a personalized, scrollytelling web page for homeowners that turns equity data into a narrative experience. The build follows three delivery boundaries: first, the data pipeline and admin workflow that lets Josh create pages; second, the core homeowner page with all financial data rendered in a branded, mobile-first layout; third, the scroll-driven narrative experience with animations, adaptive storytelling, future scenarios, and the CTA that makes this product unlike anything Homebot or Zillow offers.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation + Admin** - Project scaffolding, Supabase schema, calculation engine, admin input form, and slug-based URL system
- [ ] **Phase 2: Core Homeowner Page** - Branded, mobile-first page at /h/[slug] displaying all financial data -- value, equity, appreciation chart, comparisons, and net proceeds
- [ ] **Phase 3: Narrative Experience** - Scroll-triggered animations, SB7 adaptive narrative, future scenario projections, milestone markers, and CTA

## Phase Details

### Phase 1: Foundation + Admin
**Goal**: Josh can enter homeowner data through a streamlined admin form and the system computes all financial metrics correctly -- ready for page rendering
**Depends on**: Nothing (first phase)
**Requirements**: PROD-01, PROD-02
**Success Criteria** (what must be TRUE):
  1. Josh can fill out an admin form with minimal fields (address, client name, purchase price, purchase date, current value) and save a homeowner record to Supabase in under 5 minutes
  2. The admin interface is password-protected and shows a list of all created pages with status
  3. Each saved homeowner record generates a unique, URL-safe slug that will serve as the shareable URL
  4. Pure calculation functions return correct results for equity, appreciation, net proceeds, earnings-per-period, S&P comparison, rent-vs-own, and all three scenario projections -- verified by unit tests
**Plans**: 3 plans

Plans:
- [ ] 01-01-PLAN.md — Scaffold project, types, Supabase, auth middleware, slug generation
- [ ] 01-02-PLAN.md — Calculation engine (TDD) with full unit test coverage
- [ ] 01-03-PLAN.md — Admin form, CSV upload, photo upload, list view

### Phase 2: Core Homeowner Page
**Goal**: Homeowners can visit their unique URL and see a complete, branded, mobile-first page showing their home's financial story -- current value, equity gained, appreciation over time, comparisons, and net proceeds
**Depends on**: Phase 1
**Requirements**: DATA-01, DATA-02, DATA-03, DATA-04, DATA-05, DATA-06, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07
**Success Criteria** (what must be TRUE):
  1. Visiting /h/[slug] renders a page showing current home value, equity gained since purchase, an appreciation chart over time, net proceeds estimate, earnings-per-week reframing, S&P 500 comparison, and ownership-vs-renting comparison
  2. The page uses a scrolling narrative timeline structure with SB7 framing -- homeowner is the hero, Live AZ Co is the guide, every section opens with narrative context rather than raw numbers
  3. The page is fully branded with Live AZ Co design language (Olive, Canyon, Gold, Cream, Charcoal palette; Source Serif 4 headings; DM Sans body)
  4. The page is mobile-first responsive and renders correctly on phone screens (primary viewing context is opening a text message link)
**Plans**: TBD

Plans:
- [ ] 02-01: TBD
- [ ] 02-02: TBD
- [ ] 02-03: TBD

### Phase 3: Narrative Experience
**Goal**: The page transforms from a data display into a cinematic narrative experience -- sections animate on scroll, the story adapts to how long the homeowner has owned their home, future scenarios spark conversations, and a soft CTA brings them back to Josh
**Depends on**: Phase 2
**Requirements**: NARR-03, NARR-04, NARR-05, NARR-08, SCEN-01, SCEN-02, SCEN-03
**Success Criteria** (what must be TRUE):
  1. Data sections animate on scroll -- counters count up, charts draw themselves, sections reveal as the user scrolls down
  2. The narrative adapts based on ownership duration: recent buyers (under 2 years) see forward-looking "where you are headed" language; long-term owners (5+ years) see a rich retrospective of what they have built
  3. Three future scenario sections are visible and populated: Hold and Rent (when rental income covers the mortgage), Move-Up (net proceeds and what that buys as a down payment), and Equity Play (HELOC potential for investment property leverage)
  4. Timeline includes milestone markers (purchase anniversary, equity milestones) that anchor the homeowner's story in real moments
  5. The page ends with a soft CTA to contact Josh ("Ready to explore your options? Let's talk.") that feels like a natural conclusion to the narrative, not a sales pitch
**Plans**: TBD

Plans:
- [ ] 03-01: TBD
- [ ] 03-02: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation + Admin | 0/3 | Not started | - |
| 2. Core Homeowner Page | 0/3 | Not started | - |
| 3. Narrative Experience | 0/2 | Not started | - |
