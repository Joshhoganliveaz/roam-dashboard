# Homeowner Journey Map

## What This Is

A personalized, shareable web page for homeowners that tells the interactive story of their home — past, present, and future. Built for the Live AZ Co real estate team to send to past clients, new buyers, and sphere contacts via text or email. Each page is unique to the homeowner, features condition-aware value selection with cascading recalculation, real Cromford market data, four future scenario cards with mini mortgage calculators, and a premium frosted-glass hero section — all framed using the StoryBrand SB7 narrative framework where the homeowner is the hero and Live AZ Co is the guide.

## Core Value

Show homeowners the full journey of their home — where they've been, where they are, and where they could go — so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.

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
- ✓ Condition-aware value range with 3-variant server pre-computation — v1.1
- ✓ Cascading recalculation on condition change (equity, scenarios, comparisons) — v1.1
- ✓ Cromford appreciation chart with real market data via screenshot OCR — v1.1
- ✓ Merged Investment Comparison section with leverage ROI and rate lock callout — v1.1
- ✓ Premium frosted-glass hero with Playfair Display gold stat — v1.1
- ✓ Page flow rework: paycheck hook leads, scenarios before proof — v1.1
- ✓ Four reworked scenario cards (Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep as Rental) — v1.1
- ✓ Dynamic date horizons and mini mortgage calculators in scenario cards — v1.1
- ✓ Effective interest rate concept for Move & Keep as Rental — v1.1
- ✓ Condition picker reframed as neighbor comparison with inline comp data — v1.1
- ✓ Information cliffs removed — full information, conversations when ready — v1.1
- ✓ FRED API market rate integration — v1.1

### Active

(None — define with `/gsd:new-milestone`)

### Out of Scope

- Real-time MLS data integration — manual data input plus Cromford chart is sufficient
- User accounts or login for homeowners — they get a link, no signup
- Automated email/text sending — Josh sends the links manually for now
- Homebot-style automated recurring updates — each page is a moment-in-time touchpoint
- Appraisal-grade accuracy on appreciation chart — narrative, not legal
- Automated Tableau Public scraping — reverse-engineered API too fragile; screenshot OCR is more reliable

## Context

Shipped v1.1 with 11,103 LOC TypeScript. Tech stack: Next.js 16, React 19, TypeScript, Tailwind CSS 4, Supabase, Recharts, Motion (Framer Motion), FRED API.

200+ homeowner pages in production. Josh creates each page in under 5 minutes via admin form. Pages are sent via text/email to past clients, new buyers, and sphere contacts.

v1.1 transformed the page from a static snapshot to an interactive experience — homeowners can select their home's condition, see cascading recalculation across all sections, explore four distinct future scenarios with interactive calculators, and view real Cromford market data.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Narrative timeline over static report | Differentiation from Homebot/Zillow; SB7 alignment; shareability | ✓ Good |
| Future scenario projections included | Creates conversations, shows possibilities, drives repeat engagement | ✓ Good |
| Manual data input for v1 | Speed to build, avoids API complexity, 5-min input is acceptable at current volume | ✓ Good |
| Single page template that adapts by ownership duration | Simpler to build and maintain than multiple page types | ✓ Good |
| Pre-compute 3 condition variants server-side | Instant switching without API calls; ~3x data per page but negligible at current scale | ✓ Good |
| Cromford via screenshot OCR, not automated scraping | Tableau API too fragile; screenshot + Claude Vision is reliable and fast | ✓ Good |
| FRED API for market rate with fallback | Real-time rate data without paid subscription; graceful degradation | ✓ Good |
| Asymmetric condition clamping (-3%/+5%) | SB7-informed: remodeled should feel rewarding, original shouldn't be punitive | ✓ Good |
| Effective interest rate via bisection solver | Novel metric that makes "keep as rental" concrete; $0.01 tolerance is more than adequate | ✓ Good |
| PaycheckHook absorbed into frosted-glass hero | Single unified component; premium feel; eliminated separate PaycheckHook | ✓ Good |
| Card-local sell costs (6%+1%) | Scenario cards need different assumptions than global DEFAULTS | ✓ Good |

## Constraints

- **Volume**: Must be producible at 20-30/month — input-to-page must be fast (under 5 minutes)
- **Mobile-first**: Most recipients will open via text on their phone
- **Brand**: Must use Live AZ Co design language and SB7 narrative framework
- **Data**: Uses manually entered data plus Cromford screenshot OCR, not live API feeds
- **Differentiation**: Must not feel like Homebot, Zillow Zestimate, or any generic automated home value tool
- **Backward compat**: Schema changes must be nullable/optional to avoid breaking 200+ existing pages

---
*Last updated: 2026-03-15 after v1.1 milestone*
