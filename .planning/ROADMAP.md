# Roadmap: Homeowner Journey Map

## Overview

This roadmap delivers a personalized, scrollytelling web page for homeowners that turns equity data into a narrative experience. The build follows three delivery boundaries: first, the data pipeline and admin workflow that lets Josh create pages; second, the core homeowner page with all financial data rendered in a branded, mobile-first layout; third, the scroll-driven narrative experience with animations, adaptive storytelling, future scenarios, and the CTA that makes this product unlike anything Homebot or Zillow offers.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation + Admin** - Project scaffolding, Supabase schema, calculation engine, admin input form, and slug-based URL system
- [x] **Phase 2: Core Homeowner Page** - Branded, mobile-first page at /h/[slug] displaying all financial data -- value, equity, appreciation chart, comparisons, and net proceeds
- [x] **Phase 3: Narrative Experience** - Scroll-triggered animations, SB7 adaptive narrative, future scenario projections, milestone markers, and CTA (completed 2026-03-12)
- [ ] **Phase 4: Documentation & Tracking Cleanup** - Close audit documentation gaps: missing summaries, Phase 2 verification, stale ROADMAP checkboxes, requirements tracking

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
- [x] 01-01-PLAN.md — Scaffold project, types, Supabase, auth middleware, slug generation
- [x] 01-02-PLAN.md — Calculation engine (TDD) with full unit test coverage
- [x] 01-03-PLAN.md — Admin form, CSV upload, photo upload, list view

### Phase 2: Core Homeowner Page
**Goal**: Homeowners can visit their unique URL and see a complete, branded, mobile-first page showing their home's financial story -- current value, equity gained, appreciation over time, comparisons, and net proceeds
**Depends on**: Phase 1
**Requirements**: DATA-01, DATA-02, DATA-03, DATA-04, DATA-05, DATA-06, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07
**Success Criteria** (what must be TRUE):
  1. Visiting /h/[slug] renders a page showing current home value, equity gained since purchase, an appreciation chart over time, net proceeds estimate, earnings-per-week reframing, S&P 500 comparison, and ownership-vs-renting comparison
  2. The page uses a scrolling narrative timeline structure with SB7 framing -- homeowner is the hero, Live AZ Co is the guide, every section opens with narrative context rather than raw numbers
  3. The page is fully branded with Live AZ Co design language (Olive, Canyon, Gold, Cream, Charcoal palette; Source Serif 4 headings; DM Sans body)
  4. The page is mobile-first responsive and renders correctly on phone screens (primary viewing context is opening a text message link)
**Plans**: 3 plans

Plans:
- [x] 02-01-PLAN.md — Format utilities, server component pipeline, hero section, personal note, page shell
- [x] 02-02-PLAN.md — Six data section components (purchase story through net proceeds) with SB7 narrative
- [x] 02-03-PLAN.md — Appreciation chart (Recharts), S&P callout card, visual verification

### Phase 02.1: Financial Data Audit & Edge Cases (INSERTED)

**Goal:** All financial data displays correctly for edge cases (underwater homes, negative appreciation, cash purchases), number formatting matches production standards, S&P 500 comparison uses real historical data, and Josh can preview pages before publishing
**Requirements**: DATA-06, DATA-01, DATA-02, DATA-04, DATA-05, PROD-01, FMT-01, FMT-02, FMT-03, SP5-01, SP5-02, SP5-03, SP5-04, EDGE-01, EDGE-02, VAL-01, VAL-02
**Depends on:** Phase 2
**Plans:** 4/4 plans complete

Plans:
- [x] 02.1-01-PLAN.md — Formatting fixes, validation module, edge case tests
- [x] 02.1-02-PLAN.md — S&P 500 real data file, lookup module, calculation wiring
- [x] 02.1-03-PLAN.md — SB7 underwater narrative for all data section components
- [x] 02.1-04-PLAN.md — Admin validation warnings and preview-before-publish workflow

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
**Plans**: 2 plans

Plans:
- [x] 03-01-PLAN.md — Animation infrastructure, adaptive narrative voice, milestone markers, wired into all sections
- [x] 03-02-PLAN.md — Future scenario cards (Hold & Rent, Move-Up, Equity Play), branded CTA section, visual verification

### Phase 4: Documentation & Tracking Cleanup
**Goal**: Close all documentation and tracking gaps identified by the v1.0 milestone audit so that every requirement has proper verification evidence and all tracking artifacts are accurate
**Depends on**: Phase 3
**Requirements**: DATA-03, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07
**Gap Closure:** Closes gaps from audit
**Success Criteria** (what must be TRUE):
  1. 02-03-SUMMARY.md exists documenting the completed appreciation chart plan
  2. 02-VERIFICATION.md exists and formally verifies all Phase 2 requirements (DATA-03, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07)
  3. DATA-03 is marked [x] Complete in REQUIREMENTS.md with traceability updated
  4. All ROADMAP.md checkboxes and status fields accurately reflect actual execution state
**Plans**: 0 plans

Plans:
- [ ] 04-01-PLAN.md — Create missing summaries, Phase 2 verification, update all tracking artifacts

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 2.1 -> 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation + Admin | 3/3 | Complete | 2026-03-12 |
| 2. Core Homeowner Page | 3/3 | Complete | 2026-03-12 |
| 2.1 Financial Data Audit | 4/4 | Complete | 2026-03-12 |
| 3. Narrative Experience | 2/2 | Complete | 2026-03-12 |
| 4. Documentation & Tracking Cleanup | 0/1 | Not started | - |
