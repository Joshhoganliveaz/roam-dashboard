# Roadmap: Homeowner Journey Map

## Milestones

- ✅ **v1.0 Foundation** - Phases 1-4 (shipped 2026-03-12)
- 🚧 **v1.1 Enhanced Intelligence** - Phases 5-7 (in progress)

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

<details>
<summary>✅ v1.0 Foundation (Phases 1-4) - SHIPPED 2026-03-12</summary>

- [x] **Phase 1: Foundation + Admin** - Project scaffolding, Supabase schema, calculation engine, admin input form, and slug-based URL system
- [x] **Phase 2: Core Homeowner Page** - Branded, mobile-first page at /h/[slug] displaying all financial data
- [x] **Phase 2.1: Financial Data Audit & Edge Cases (INSERTED)** - Formatting fixes, S&P real data, edge case handling, admin preview
- [x] **Phase 3: Narrative Experience** - Scroll-triggered animations, SB7 adaptive narrative, future scenario projections, milestone markers, and CTA
- [x] **Phase 4: Documentation & Tracking Cleanup** - Close audit documentation gaps

### Phase 1: Foundation + Admin
**Goal**: Josh can enter homeowner data through a streamlined admin form and the system computes all financial metrics correctly
**Depends on**: Nothing (first phase)
**Requirements**: PROD-01, PROD-02
**Plans**: 3/3 complete

Plans:
- [x] 01-01: Scaffold project, types, Supabase, auth middleware, slug generation
- [x] 01-02: Calculation engine (TDD) with full unit test coverage
- [x] 01-03: Admin form, CSV upload, photo upload, list view

### Phase 2: Core Homeowner Page
**Goal**: Homeowners can visit their unique URL and see a complete, branded, mobile-first page showing their home's financial story
**Depends on**: Phase 1
**Requirements**: DATA-01, DATA-02, DATA-03, DATA-04, DATA-05, DATA-06, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07
**Plans**: 3/3 complete

Plans:
- [x] 02-01: Format utilities, server component pipeline, hero section, personal note, page shell
- [x] 02-02: Six data section components with SB7 narrative
- [x] 02-03: Appreciation chart (Recharts), S&P callout card, visual verification

### Phase 2.1: Financial Data Audit & Edge Cases (INSERTED)
**Goal**: All financial data displays correctly for edge cases, formatting matches production standards, S&P uses real data
**Depends on**: Phase 2
**Requirements**: DATA-06, DATA-01, DATA-02, DATA-04, DATA-05, PROD-01, FMT-01, FMT-02, FMT-03, SP5-01, SP5-02, SP5-03, SP5-04, EDGE-01, EDGE-02, VAL-01, VAL-02
**Plans**: 4/4 complete

Plans:
- [x] 02.1-01: Formatting fixes, validation module, edge case tests
- [x] 02.1-02: S&P 500 real data file, lookup module, calculation wiring
- [x] 02.1-03: SB7 underwater narrative for all data section components
- [x] 02.1-04: Admin validation warnings and preview-before-publish workflow

### Phase 3: Narrative Experience
**Goal**: Page transforms from data display into cinematic narrative experience with scroll animations, adaptive storytelling, and future scenarios
**Depends on**: Phase 2
**Requirements**: NARR-03, NARR-04, NARR-05, NARR-08, SCEN-01, SCEN-02, SCEN-03
**Plans**: 2/2 complete

Plans:
- [x] 03-01: Animation infrastructure, adaptive narrative voice, milestone markers
- [x] 03-02: Future scenario cards, branded CTA section, visual verification

### Phase 4: Documentation & Tracking Cleanup
**Goal**: Close all documentation and tracking gaps from v1.0 milestone audit
**Depends on**: Phase 3
**Requirements**: DATA-03, DATA-07, NARR-01, NARR-02, NARR-06, NARR-07
**Plans**: 1/1 complete

Plans:
- [x] 04-01: Create missing summaries, Phase 2 verification, update all tracking artifacts

</details>

### 🚧 v1.1 Enhanced Intelligence (In Progress)

**Milestone Goal:** Transform the page from a static snapshot into an interactive, data-rich experience -- real market data, homeowner-driven value selection, deeper scenario projections, and a merged investment comparison story.

- [ ] **Phase 5: Data Foundation** - Schema migration, value range selector with condition picker, Cromford appreciation chart with real market data, cascading recalculation architecture
- [x] **Phase 6: Investment Comparison + Narrative Cleanup** - Merged Investment Comparison section replacing separate S&P and Rent vs Own, rate lock callout, information cliffs removed across all sections (completed 2026-03-14)
- [x] **Phase 7: Page Flow & Hook Rework** - Weekly paycheck hook to hero position, page section reorder (scenarios up, data down), simple CTA, Cromford mobile fix, remove external links (completed 2026-03-14)
- [ ] **Phase 07.1: Hero Section Redesign (INSERTED)** - Premium frosted-glass hero with Playfair Display gold stat, full-bleed photo, responsive mobile layout, logo integration
- [ ] **Phase 8: Condition Picker & Comp Context** - Reframe condition picker as neighbor comparison, inline comp data from CSV, tighter value range
- [ ] **Phase 9: Scenario Cards Rework** - Four new scenarios (Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep as Rental), market rate slider, dynamic date horizons, mini mortgage calculators, effective interest rate

## Phase Details

### Phase 5: Data Foundation
**Goal**: Homeowners see a condition-aware home value (original/updated/remodeled) that cascades through all financial numbers, and the appreciation chart shows real Cromford market data instead of a straight-line estimate
**Depends on**: Phase 4 (v1.0 complete)
**Requirements**: VALU-01, VALU-02, VALU-03, VALU-04, CROM-01, CROM-02, CROM-03, CROM-04
**Success Criteria** (what must be TRUE):
  1. Josh can enter a low comp value and high comp value in the admin form instead of a single current value, and existing pages with only a single value continue to render correctly (no breakage on 200+ published pages)
  2. Homeowner sees three condition choices (Mostly original / Made some updates / Completely remodeled) on their page, and selecting one updates the displayed home value to the corresponding position within the comp range
  3. Changing condition recalculates all downstream numbers on the page -- equity, net proceeds, earnings-per-week, comparisons, and scenario projections -- without a page reload
  4. Josh can upload a Cromford Report screenshot in the admin form, and the appreciation chart on the homeowner page renders real market price-per-sqft data as a branded Recharts chart
  5. When the homeowner changes condition, the appreciation curve shifts proportionally (price-per-sqft times home sqft times condition factor)
**Plans**: 3 plans

Plans:
- [x] 05-01-PLAN.md -- Types, schema, condition value mapping, server pre-computation, admin form value range fields
- [x] 05-02-PLAN.md -- Cromford OCR pipeline, Claude Vision extraction, admin upload with preview/approve
- [x] 05-03-PLAN.md -- Condition picker UI, cascading counter-roll animation, appreciation chart with Cromford data (Task 3 checkpoint pending)

### Phase 6: Investment Comparison + Narrative Cleanup
**Goal**: The separate S&P 500 and Rent vs Own sections merge into a single, powerful Investment Comparison section that tells the leverage story, and all page sections provide full information without artificial cliffs
**Depends on**: Phase 5
**Requirements**: INVS-01, INVS-02, INVS-03, INVS-04, SCEN-11
**Success Criteria** (what must be TRUE):
  1. A single "Investment Comparison" section replaces the separate S&P 500 callout and Rent vs Own sections, showing three key numbers side by side: home ROI with leverage, S&P alternative return on the down payment, and rent drag subtracted from the S&P path
  2. The section includes a personalized leverage callout using the homeowner's specific numbers (e.g., "$25K down controlling a $500K asset = 165% return")
  3. If the homeowner's locked mortgage rate is below current market rates, a rate lock advantage callout quantifies their annual savings vs current rates
  4. All page sections present full information -- no data hidden behind "click to reveal" or truncation -- so homeowners feel empowered to explore on their own terms
**Plans**: 2 plans

Plans:
- [x] 06-01-PLAN.md -- Leverage ROI + rate lock savings calculations (TDD), FRED API market rate utility, type extensions
- [x] 06-02-PLAN.md -- InvestmentComparison component, page wiring + section reorder, scenario card cliff question removal

### Phase 7: Page Flow & Hook Rework
**Goal**: The weekly paycheck hook leads the page, scenario cards move up closer to the action, data/proof sections move below, and the page feels like a narrative that hooks immediately and drives engagement
**Depends on**: Phase 6 (page structure from investment comparison cleanup)
**Requirements**: PAGE-01, PAGE-02, PAGE-03, PAGE-04, PAGE-05, PAGE-06
**Success Criteria** (what must be TRUE):
  1. "Your home earned $X this week" is the first compelling number a homeowner sees on page load, positioned in or near the hero section
  2. Scenario cards appear before the appreciation chart, investment comparison, and mortgage details sections -- the opportunity comes before the proof
  3. Bottom CTA is simple: Josh & Jacqui contact info + photo + "Let's talk about your options" (replaces previous CTA style)
  4. No external links take the homeowner off the page (the "see a similar home" link is removed)
  5. Cromford chart date labels are readable on mobile (no squished/overlapping text)
  6. Net proceeds no longer appears as a standalone data section (it will live inside the Sell & Move Up scenario card in Phase 9)
**Plans**: 2 plans

Plans:
- [x] 07-01-PLAN.md -- Hero paycheck hook, page section reorder, remove net proceeds and external links
- [x] 07-02-PLAN.md -- Simple CTA replacement, Cromford chart mobile date label fix

### Phase 07.1: Hero Section Redesign (INSERTED)
**Goal**: The hero section is a premium, screenshot-worthy presentation with full-bleed home photo, frosted glass panels, and an oversized Playfair Display gold stat -- replacing the separate HeroSection + PaycheckHook layout with a single unified component
**Depends on**: Phase 7
**Requirements**: HERO-01, HERO-02, HERO-03, HERO-04, HERO-05, HERO-06, HERO-07
**Success Criteria** (what must be TRUE):
  1. Hero displays full-bleed home photo with warm gradient overlay, or branded gradient fallback when no photo
  2. Desktop shows two asymmetric frosted glass panels: top-left (logo + name + address), bottom-right (weekly paycheck stat in gold)
  3. Mobile shows single full-width frosted panel at bottom with name, gold divider, and stat; logo floats top-left
  4. Gold stat number uses Playfair Display 700 at 112px desktop / 76px mobile
  5. Stat panel is hidden when earningsPerWeek <= 0 -- hero shows photo + gradient + name only
  6. Live AZ Co white logo renders in hero from resized PNG asset
  7. PaycheckHook component is deleted and HomeownerPage passes earningsPerWeek directly to HeroSection
**Plans**: 2 plans

Plans:
- [ ] 07.1-01-PLAN.md -- Font infrastructure, logo asset, HeroSection rewrite, HomeownerPage wiring, tests
- [ ] 07.1-02-PLAN.md -- Visual verification checkpoint (desktop, mobile, Safari, negative equity)

### Phase 8: Condition Picker & Comp Context
**Goal**: The condition picker feels like a natural neighbor comparison rather than a self-assessment, with real comp data providing context, and the value range is tight enough to be credible
**Depends on**: Phase 7 (page flow settled)
**Requirements**: COMP-01, COMP-02, COMP-03, COMP-04
**Success Criteria** (what must be TRUE):
  1. Condition picker is reframed as "How does your home compare to your neighbors?" instead of "Select your home's condition"
  2. 2-3 nearby recent sales from CSV comp data are shown inline (address, sold price, optional photo) with no external links
  3. Value range is tightened to +/-3-5% from base value so selections feel meaningful but don't wildly change downstream numbers
  4. Comp display is minimal and clean -- informs the selection without making the section feel busy
**Plans**: 0 plans

### Phase 9: Scenario Cards Rework
**Goal**: Four scenario cards match real homeowner decision paths (Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep as Rental) with a market rate slider, dynamic date horizons, mini mortgage calculators, and the effective interest rate concept
**Depends on**: Phase 7 (page flow with scenarios in correct position), Phase 8 (condition picker stable)
**Requirements**: SCNR-01, SCNR-02, SCNR-03, SCNR-04, SCNR-05, SCNR-06, SCNR-07, SCNR-08, SCNR-09, SCNR-10
**Success Criteria** (what must be TRUE):
  1. Four scenario cards render: Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental -- each representing a distinct homeowner path
  2. Dynamic date horizons show actual dates (e.g., Mar '26 | Mar '31 | Mar '36 | Mar '41) calculated from the page view date, with year-only fallback on mobile if needed
  3. One market rate slider above all cards sets the baseline rate; mini mortgage calculators inside Sell & Move Up and Move & Keep as Rental can override independently
  4. Sell & Move Up shows net proceeds with "cost to sell*" label (title fees & broker fees), mini mortgage calculator with 30yr term, adjustable rate, and 5/10/15/20% down payment presets
  5. Stay & Invest shows HELOC available, monthly cost, investment property framing, tax advantages callout for W-2 earners, and CTAs to STR Analyzer and Josh & Jacqui
  6. Move & Keep as Rental shows the effective interest rate as the hero metric -- the rate that produces their actual out-of-pocket payment after rental cash flow offsets the new mortgage
  7. Each scenario has its own contextual CTA (lender connect, STR Analyzer, talk to Josh & Jacqui)
**Plans**: 3 plans

Plans:
- [ ] 09-01-PLAN.md -- Types, calcEffectiveInterestRate (TDD), date horizon utility, sell cost constants
- [ ] 09-02-PLAN.md -- HorizonTabs refactor (date-based), MiniMortgageCalculator, StayAndBuildCard, SellAndMoveUpCard
- [ ] 09-03-PLAN.md -- StayAndInvestCard, MoveAndKeepAsRentalCard, wire all four into ScenarioCards.tsx, update tests

## Progress

**Execution Order:**
Phases execute in numeric order: 5 -> 6 -> 7 -> 07.1 -> 8 -> 9

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Foundation + Admin | v1.0 | 3/3 | Complete | 2026-03-12 |
| 2. Core Homeowner Page | v1.0 | 3/3 | Complete | 2026-03-12 |
| 2.1 Financial Data Audit | v1.0 | 4/4 | Complete | 2026-03-12 |
| 3. Narrative Experience | v1.0 | 2/2 | Complete | 2026-03-12 |
| 4. Documentation Cleanup | v1.0 | 1/1 | Complete | 2026-03-12 |
| 5. Data Foundation | v1.1 | 3/3 | Checkpoint | - |
| 6. Investment Comparison | v1.1 | 2/2 | Complete | 2026-03-14 |
| 7. Page Flow & Hook Rework | v1.1 | 2/2 | Complete | 2026-03-14 |
| 07.1 Hero Section Redesign | 1/2 | In Progress|  | - |
| 8. Condition Picker & Comp Context | v1.1 | 0/0 | Not started | - |
| 9. Scenario Cards Rework | v1.1 | 0/3 | Planned | - |
