# Homeowner Journey Map

## What This Is

A personalized, shareable web page for homeowners that tells the story of their home — past, present, and future. Built for the Live AZ Co real estate team to send to past clients, new buyers, and sphere contacts via text or email. Each page is unique to the homeowner, auto-generated from minimal input (address, purchase price, purchase date, client name), and framed using the StoryBrand SB7 narrative framework where the homeowner is the hero and Live AZ Co is the guide.

Unlike Homebot, Zillow, or automated home value emails, this isn't a report — it's a narrative experience. A scrolling timeline that starts with their purchase, shows what's happened since, and then reveals possible futures for their property: hold and rent, move up, pull equity, or sell. It turns data into a story that homeowners want to share and come back to.

## Core Value

Show homeowners the full journey of their home — where they've been, where they are, and where they could go — so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.

## Current Milestone: v1.1 Enhanced Intelligence

**Goal:** Transform the page from a static snapshot into an interactive, data-rich experience — real market data, homeowner-driven value selection, deeper scenario projections, and a merged investment comparison story.

**Target features:**
- Value range selector with condition picker (original/updated/remodeled)
- Cromford appreciation chart via Tableau scraper + screenshot fallback
- Merged Investment Comparison section (S&P + Rent vs Own leverage story)
- Future scenarios rework: 5/10/15yr projections, market rate slider, rate lock callout, 4th combo scenario
- UX polish: cascading animations, information cliffs removed

## Requirements

### Validated

- ✓ Personalized page per homeowner generated from minimal input — v1.0
- ✓ Scrolling narrative timeline from purchase date to present — v1.0
- ✓ SB7 framing throughout — v1.0
- ✓ Equity and appreciation visualization — v1.0
- ✓ Reframing comparisons (home vs S&P 500, ownership cost vs renting) — v1.0
- ✓ Future scenario projections: Hold & Rent, Move-Up, Equity Play — v1.0
- ✓ Shareable via unique URL — v1.0
- ✓ Adaptive narrative by ownership duration — v1.0
- ✓ Production speed under 5 minutes — v1.0
- ✓ Live AZ Co branding — v1.0
- ✓ Mobile-first design — v1.0
- ✓ Volume: 200+ pages, 20-30 new/month — v1.0

### Active

- [ ] Value range selector: Josh enters low/high comp values, homeowner picks condition (original/updated/remodeled)
- [ ] Cascading recalculation when condition changes (equity, scenarios, all downstream numbers)
- [ ] Cromford appreciation chart with real market data (Tableau scraper primary, screenshot OCR fallback)
- [ ] Merged Investment Comparison section replacing separate S&P and Rent vs Own sections
- [ ] Future scenarios: 5/10/15 year projection timelines for all three scenarios
- [ ] Future scenarios: market rate slider affecting all scenarios differently
- [ ] Future scenarios: rate lock advantage callout
- [ ] Future scenarios: 4th combo scenario ("Keep, rent, and buy another")
- [ ] Enhanced Hold & Rent: projected cash flow and appreciation at 5/10/15yr
- [ ] Enhanced Move-Up: education on sell-to-buy paths
- [ ] Enhanced Equity Play: HELOC with rate-adjusted costs
- [ ] UX: cascading ripple animation on condition change
- [ ] Information cliffs removed — full information, conversations happen when ready

### Out of Scope

- Real-time MLS data integration — manual data input plus Cromford chart is sufficient
- User accounts or login for homeowners — they just get a link
- Automated email/text sending — Josh sends the links manually for now
- Homebot-style automated recurring updates — each page is a moment-in-time touchpoint
- Appraisal-grade accuracy on appreciation chart — narrative, not legal

## Context

- Josh Hogan runs Live AZ Co, a real estate team in Phoenix, Arizona
- Existing tools: CMA report generator (Flask/ARMLS parsing), STR Analyzer (Next.js/financial modeling), Idea Pipeline, Commission Calculator
- Established brand design language: Olive (#5C6B4F), Canyon (#8B6F5C), Gold (#C9953E), Cream (#FAF7F2), Charcoal (#2C2C2C). Fonts: Source Serif 4 (headings), DM Sans (body)
- The STR Analyzer already has mortgage, amortization, and rental projection math that could inform future scenario calculations
- ARMLS MLS data parsing exists in the CMA tool
- The audience spans recent buyers (weeks/months), mid-term owners (1-3 years), and long-term clients (5+ years) — the narrative must adapt
- Key differentiator: this is NOT an automated report. It's a personal, narrative, branded experience that feels like it came from their agent, not a computer
- SB7 framework rules: never use the word "just" (undercuts authority), every section intro names a problem or promise, the homeowner is always the hero

## Constraints

- **Volume**: Must be producible at 20-30/month — input-to-page must be fast (under 5 minutes)
- **Mobile-first**: Most recipients will open via text on their phone
- **Brand**: Must use Live AZ Co design language and SB7 narrative framework
- **Data**: v1 uses manually entered data, not live API feeds
- **Differentiation**: Must not feel like Homebot, Zillow Zestimate, or any generic automated home value tool

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Narrative timeline over static report | Differentiation from Homebot/Zillow; SB7 alignment; shareability | — Pending |
| Future scenario projections included | Creates conversations, shows possibilities, drives repeat engagement | — Pending |
| Manual data input for v1 | Speed to build, avoids API complexity, 5-min input is acceptable at current volume | — Pending |
| Single page template that adapts by ownership duration | Simpler to build and maintain than multiple page types; narrative naturally shifts | — Pending |
| Parked ideas documented in IDEAS.md | Five alternative concepts captured for future reference if direction pivots | — Pending |

---
*Last updated: 2026-03-13 after v1.1 milestone start*
