# Milestones: Homeowner Journey Map

## v1.0 — Foundation (Completed 2026-03-12)

**Goal:** Build a personalized, shareable homeowner page with SB7 narrative, financial data, scroll animations, and future scenario projections.

**Phases:**
- Phase 1: Foundation + Admin (3 plans)
- Phase 2: Core Homeowner Page (3 plans)
- Phase 2.1: Financial Data Audit & Edge Cases (4 plans) — INSERTED
- Phase 3: Narrative Experience (2 plans)
- Phase 4: Documentation & Tracking Cleanup (1 plan)

**Stats:** 5 phases, 13 plans, ~0.60 hours total execution

**What shipped:**
- Admin form with CSV upload, photo upload, preview/publish workflow
- Calculation engine (equity, appreciation, S&P 500 comparison, rent vs own, 3 scenarios)
- Branded homeowner page at /h/[slug] with scrolling SB7 narrative
- Scroll-triggered animations, adaptive voice by ownership duration
- Future scenario cards (Hold & Rent, Move-Up, Equity Play)
- Edge case handling (underwater, cash, negative appreciation)
- Mobile-first responsive design with Live AZ Co branding

**Last phase number:** 4

## v1.1 — Enhanced Intelligence (Completed 2026-03-15)

**Goal:** Transform the page from a static snapshot into an interactive, data-rich experience -- real market data, homeowner-driven value selection, deeper scenario projections, and a merged investment comparison story.

**Phases:**
- Phase 5: Data Foundation (3 plans)
- Phase 6: Investment Comparison + Narrative Cleanup (2 plans)
- Phase 7: Page Flow & Hook Rework (2 plans)
- Phase 07.1: Hero Section Redesign (1 plan) — INSERTED
- Phase 8: Condition Picker & Comp Context (1 plan)
- Phase 9: Scenario Cards Rework (3 plans)
- Phase 10: Verification & Tracking Closure (1 plan)
- Phase 11: Hero Logo Fix & Dead Code Cleanup (1 plan)

**Stats:** 8 phases, 14 plans, 70 commits, 79 files changed (+8,229/-585 lines), 11,103 LOC TypeScript
**Timeline:** 2 days (2026-03-13 → 2026-03-15)

**What shipped:**
- Condition-aware value range with 3-variant server pre-computation and cascading counter-roll animations
- Cromford OCR pipeline: screenshot upload → Claude Vision extraction → branded Recharts chart with real market data
- Merged Investment Comparison section with leverage ROI hero number, S&P conditional one-liner, and rate lock callout via FRED API
- Premium frosted-glass hero with Playfair Display gold stat, full-bleed photo, and PNG logo
- Page flow rework: paycheck hook leads, scenarios before proof, warm CTA with inline contact links
- Four reworked scenario cards (Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep as Rental) with date-based horizons, mini mortgage calculators, and effective interest rate
- Condition picker reframed as neighbor comparison with inline comp data and clamped value range
- Dead code cleanup: 331 lines removed from 4 superseded components

**Audit:** PASSED — 35/35 requirements, 8/8 phases, 5/5 E2E flows, 245 tests passing

**Last phase number:** 11
