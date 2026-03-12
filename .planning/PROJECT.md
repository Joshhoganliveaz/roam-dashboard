# Homeowner Journey Map

## What This Is

A personalized, shareable web page for homeowners that tells the story of their home — past, present, and future. Built for the Live AZ Co real estate team to send to past clients, new buyers, and sphere contacts via text or email. Each page is unique to the homeowner, auto-generated from minimal input (address, purchase price, purchase date, client name), and framed using the StoryBrand SB7 narrative framework where the homeowner is the hero and Live AZ Co is the guide.

Unlike Homebot, Zillow, or automated home value emails, this isn't a report — it's a narrative experience. A scrolling timeline that starts with their purchase, shows what's happened since, and then reveals possible futures for their property: hold and rent, move up, pull equity, or sell. It turns data into a story that homeowners want to share and come back to.

## Core Value

Show homeowners the full journey of their home — where they've been, where they are, and where they could go — so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Personalized page per homeowner generated from minimal input (address, purchase price, purchase date, client name)
- [ ] Scrolling narrative timeline from purchase date to present with data-backed beats
- [ ] SB7 framing throughout — homeowner is the hero, Live AZ Co is the guide
- [ ] Equity and appreciation visualization (how much the home has gained since purchase)
- [ ] Reframing comparisons (home vs S&P 500, ownership cost vs renting, equity earned per week/month)
- [ ] Future scenario projections: "Hold & Rent" path (when rents cover mortgage)
- [ ] Future scenario projections: "Move-Up" path (what they'd net from a sale, what that buys)
- [ ] Future scenario projections: "Equity Play" path (HELOC potential, investment property leverage)
- [ ] Shareable via unique URL (text/email to homeowner)
- [ ] Adapts narrative based on ownership duration (recent buyers get forward-looking "where you're headed" story; long-term owners get rich retrospective)
- [ ] Production speed: create a new page in under 5 minutes
- [ ] Live AZ Co branding (Olive, Canyon, Gold, Cream, Charcoal color palette)
- [ ] Mobile-first design (most will open via text message)
- [ ] Volume: supports 200+ pages, 20-30 new per month

### Out of Scope

- Real-time MLS data integration (manual data input is fine for v1) — keeps it simple and avoids API complexity
- User accounts or login for homeowners — they just get a link
- Automated email/text sending — Josh sends the links manually for now
- Interactive calculators or user-adjustable inputs on the page — this is a curated experience, not a tool
- Homebot-style automated recurring updates — each page is a moment-in-time touchpoint

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
*Last updated: 2026-03-11 after initialization*
