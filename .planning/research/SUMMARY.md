# Project Research Summary

**Project:** Homeowner Journey Map
**Domain:** Personalized, data-driven homeowner narrative pages (real estate engagement tool)
**Researched:** 2026-03-11
**Confidence:** HIGH

## Executive Summary

The Homeowner Journey Map is a scrollytelling web application that generates personalized, shareable pages for homeowners showing their equity growth story. It competes with Homebot and Zillow not on data depth but on narrative experience -- no competitor tells a story. The recommended approach leverages Josh's existing Next.js/Supabase/Tailwind stack (identical to STR Analyzer) with the addition of Motion for scroll animations and Recharts for equity visualization. This eliminates learning curve and lets development focus on the product's actual differentiator: the SB7 narrative engine and scroll-driven presentation.

The architecture is straightforward: Admin Form writes to Supabase, a pure-function Calculation Engine computes equity metrics, a Narrative Engine selects SB7 story templates based on ownership duration, and Next.js ISR serves the result as cached static pages. Every component in this pipeline has a proven pattern in Josh's existing projects. The critical dependency chain is Calculation Engine then Narrative Engine then Page Components -- each layer must be correct before the next can work.

The primary risks are (1) appreciation numbers that conflict with what homeowners believe their home is worth (mitigated by range-based framing and source attribution), (2) the page feeling like "another Homebot" instead of a narrative experience (mitigated by enforcing scrollytelling structure over dashboard patterns), and (3) mobile scroll performance death from animation overhead (mitigated by mobile-first development with Intersection Observer and GPU-composited-only animations). All three risks are preventable with deliberate design decisions in the correct phases.

## Key Findings

### Recommended Stack

The entire core stack mirrors STR Analyzer: Next.js 16.1.6, React 19, TypeScript 5, Tailwind CSS 4, Supabase, Zustand. This is deliberate -- zero new learning curve for framework-level decisions. New additions are limited to three libraries: Motion 12.x for scroll-triggered animations, Recharts 3.8.x for equity visualization charts, and date-fns 4.x for ownership duration calculations. Deployment targets Vercel (primary, native ISR support) with Cloudflare Pages as fallback.

**Core technologies:**
- **Next.js 16 (App Router + ISR):** On-demand static generation. Pages generate on first visit, cache as static HTML, revalidate when Josh updates data. Scales to 1,000+ pages trivially.
- **Supabase:** Same instance as STR Analyzer, new table. PostgreSQL with RLS. Free tier handles 200+ homeowner records easily.
- **Motion 12.x:** Scroll-triggered section reveals and animated counters. React-idiomatic API, 32KB gzipped. Replaces need for GSAP complexity.
- **Recharts 3.8.x:** SVG-based responsive charts for equity growth visualization. Crisp on high-DPI mobile screens.
- **Zustand 5.x:** Admin form state management, same pattern as STR Analyzer.

### Expected Features

**Must have (table stakes):**
- Current home value estimate with source attribution
- Equity gained since purchase (the "wow" moment)
- Agent branding and contact info (Live AZ Co identity)
- Mobile-responsive scrolling design (most open via text)
- Shareable unique URL (the distribution mechanism)
- Appreciation visualization (chart or animated counter)
- Net proceeds estimate ("what would I walk away with?")
- Call-to-action to contact Josh

**Should have (differentiators):**
- Scrolling narrative timeline (scrollytelling) -- the core differentiator
- SB7 narrative framing (hero/guide structure)
- Ownership-duration-adaptive narrative (3 tiers: recent, mid-term, long-term)
- Reframing comparisons (vs S&P 500, vs renting, equity per week)
- Animated data reveals on scroll
- Future scenario projections (Hold & Rent, Move-Up, Equity Play)
- Home anniversary / milestone markers
- Dynamic OG social sharing card

**Defer (v2+):**
- Interactive calculators or user-adjustable inputs (curated experience, not a tool)
- Automated recurring email updates (personal touch is the differentiator)
- Real-time MLS data integration (manual input is fine at 20-30 pages/month)
- User accounts or login (friction-free link sharing)
- PDF export (CMA tool handles print)
- Market data feeds (manual data entry is acceptable at current volume)

### Architecture Approach

Five-component pipeline: Admin Form, Supabase Database, Calculation Engine (pure functions), Narrative Engine (SB7 template selection returning structured data), and Page Renderer (Server Components with ISR). The system is a separate Next.js project from STR Analyzer, sharing the same Supabase instance. On-demand ISR is the correct rendering strategy -- pages are read-heavy, write-rare, and created ad-hoc. The calculation and narrative engines are co-located in the Next.js app (no microservices), imported directly by Server Components.

**Major components:**
1. **Admin Form** (`/admin`) -- Password-protected input form for homeowner data entry. Target: under 5 minutes per page.
2. **Calculation Engine** (`src/lib/calculations.ts`) -- Pure functions for equity, appreciation, S&P comparison, rental break-even, scenario projections. Testable in isolation.
3. **Narrative Engine** (`src/lib/narrative.ts`) -- Selects SB7 story beats based on ownership duration, returns structured section data (not markup). Enables future reuse for email or PDF.
4. **Page Renderer** (`/h/[slug]/page.tsx`) -- Server Component that orchestrates data fetch, calculation, narrative generation, and rendering. Cached via ISR.
5. **Supabase Database** -- Single `homeowners` table with slug, financial data, timestamps, and status fields.

### Critical Pitfalls

1. **Appreciation numbers that conflict with homeowner expectations** -- Frame values as ranges with source attribution ("Based on comparable sales, homes like yours have appreciated approximately 18-24%"). Never present a single definitive number. Include "as of [date]" on all value-dependent sections.
2. **Page feels like Homebot / generic dashboard** -- Design as vertical story, not dashboard. No sidebar nav, no data card grids. Every section opens with narrative hook, not a number. Limit to 2-3 charts maximum, each embedded in narrative context.
3. **Mobile scroll performance failure** -- Build mobile-first with CPU throttling in dev. Use Intersection Observer (never scroll listeners). Animate only opacity and transform. Set Lighthouse mobile performance budget above 85. Implement `prefers-reduced-motion`.
4. **SB7 narrative drift (hero/guide confusion)** -- Bake SB7 constraints into templates: homeowner is always the subject of the first sentence, "just" is banned, brand appears only in guide position. Run compliance checklist before every publish.
5. **Admin workflow exceeds 5-minute target** -- Lock minimum viable input to 4-5 fields. Auto-calculate everything derivable. Inline preview, one-click publish. Time the workflow during development.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Data Foundation + Calculation Engine
**Rationale:** Everything downstream depends on correct data schema and accurate calculations. The calculation engine must be built and unit-tested before narrative templates can reference its outputs. The data model must include source attribution, timestamps, and status fields from day one to avoid the staleness and management pitfalls.
**Delivers:** Supabase table schema, TypeScript interfaces, pure calculation functions (equity, appreciation, comparisons, scenarios), unit tests.
**Addresses:** Shareable URL system (slug design), equity calculation, appreciation math, reframing comparisons (S&P, rent, per-week).
**Avoids:** Pitfall 1 (inaccurate data -- schema includes source/date fields), Pitfall 6 (URL chaos -- deterministic slug generation with unique constraint), Pitfall 7 (data staleness -- timestamp fields built in).

### Phase 2: Admin Input Form + Data Entry Workflow
**Rationale:** Need real data entry before page rendering can be meaningful. Validates the data model works in practice. The admin experience determines whether Josh actually creates 20-30 pages/month or abandons the tool.
**Delivers:** Password-protected `/admin` routes, create/edit form, slug generation, Supabase CRUD, admin page list with search/filter/status.
**Addresses:** Admin input form, page management dashboard, publish/archive workflow.
**Avoids:** Pitfall 5 (production bottleneck -- streamlined form with minimal required fields, timed workflow test), Pitfall 6 (management chaos -- admin dashboard from day one).

### Phase 3: Narrative Engine + Scrolling Page
**Rationale:** This is the core product and the hardest phase. It requires real data (Phase 2) and correct calculations (Phase 1) to be meaningful. The narrative engine and scroll-driven presentation are what differentiate this from every competitor.
**Delivers:** SB7 narrative engine, ownership-duration-adaptive content, scroll-triggered section components (Hero, Timeline, Equity, Comparison, Scenario, CTA), mobile-first responsive design, ISR-cached dynamic routes.
**Addresses:** Scrollytelling framework, SB7 narrative framing, adaptive narrative tiers, animated data reveals, mobile-first design, appreciation visualization.
**Avoids:** Pitfall 2 (automated report feel -- enforce narrative structure over dashboard), Pitfall 3 (mobile performance -- Intersection Observer, GPU-only animations, performance budget), Pitfall 4 (SB7 drift -- compliance checklist baked into templates).

### Phase 4: Polish + Production Readiness
**Rationale:** Polish layer that makes the product shareable and production-grade but does not block initial testing with real homeowner data.
**Delivers:** Dynamic OG image generation for social sharing, admin live preview, revalidation on edit, error handling (404s, error boundaries), analytics (simple view tracking), disclaimer language on projections, `noindex` meta tags.
**Addresses:** Social sharing card, admin preview, future scenario projections (Hold & Rent, Move-Up, Equity Play), home anniversary markers.
**Avoids:** Pitfall 7 (data staleness -- "as of" date displayed prominently, refresh workflow), security mistakes (noindex, RLS policies, admin auth verification).

### Phase Ordering Rationale

- **Data before UI:** The calculation engine is the foundation. Incorrect math makes every downstream component wrong. Pure functions are testable without any UI, so Phase 1 can be validated with unit tests alone.
- **Admin before public pages:** Josh needs to enter data before pages can render. Building the admin form second validates the data model with real input before investing in the narrative experience.
- **Narrative is the product:** Phase 3 is the largest and most complex phase. It intentionally comes after data and admin are solid, so development can focus entirely on the scrollytelling experience without debugging data issues.
- **Polish is separable:** OG images, analytics, and error handling are important but do not change the core product. Deferring them to Phase 4 lets Josh start testing with real homeowners after Phase 3.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 3:** Scroll-triggered animation implementation with Motion on mobile devices. The intersection of performance, accessibility (`prefers-reduced-motion`), and visual impact needs prototyping and real-device testing. Research the Motion `useScroll` and `useInView` APIs for the specific section-reveal patterns needed.
- **Phase 4:** Dynamic OG image generation with `@vercel/og` or Satori. The API for generating branded social cards with homeowner-specific data needs technical spike.

Phases with standard patterns (skip research-phase):
- **Phase 1:** Pure calculation functions and Supabase schema -- well-documented, Josh has done this in STR Analyzer.
- **Phase 2:** Admin form with password auth and Supabase CRUD -- identical pattern to Idea Pipeline. No new territory.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Every core technology is already in Josh's projects. Versions verified against current releases. No speculative choices. |
| Features | HIGH | Competitive landscape well-documented (Homebot, Zillow, Buyside). Clear differentiation strategy. Anti-features list prevents scope creep. |
| Architecture | HIGH | ISR + Server Components + Supabase is a proven pattern. Build order follows clear dependency chain. No architectural risks at target scale (200-1,000 pages). |
| Pitfalls | HIGH | Domain-specific pitfalls identified with concrete prevention strategies and phase mappings. Recovery plans documented for each. |

**Overall confidence:** HIGH

### Gaps to Address

- **Appreciation data sourcing:** The research identifies that manual data entry is fine for v1, but does not specify where Josh will source "current value" estimates. During Phase 1 planning, define the data sourcing workflow (Zillow + Redfin + recent comps? County assessor? Josh's professional opinion?). This affects the credibility framing on the page.
- **SB7 copy templates:** The narrative engine architecture is clear, but the actual copy templates (headline patterns, body text, callout language) need to be written. This is a content creation task, not a code task. Consider drafting 3-5 sample narratives for different ownership durations before Phase 3 development begins.
- **Animation intensity calibration:** Motion is recommended over GSAP, but the research also flags an anti-pattern warning against heavy client-side animation libraries. The right level of animation (enough to feel cinematic, not enough to tank mobile performance) will need real-device testing during Phase 3.
- **Multi-market expansion:** All research assumes Phoenix metro. If Josh wants to create pages for clients in other Arizona markets (Scottsdale, Tucson, Sedona), the hardcoded market data (appreciation rates, rental yields) needs to be parameterized. Design the data constants file for easy market expansion even if v1 is Phoenix-only.

## Sources

### Primary (HIGH confidence)
- Next.js 16 releases and ISR documentation -- verified current version 16.1.6
- STR Analyzer codebase (`~/stranalyzer/str-analyzer/`) -- direct code review of existing patterns
- Idea Pipeline codebase (`~/Projects/liveaz-idea-pipeline/`) -- auth pattern verification
- Motion (formerly Framer Motion) documentation -- https://motion.dev/
- Supabase + Next.js quickstart -- https://supabase.com/docs/guides/getting-started/quickstarts/nextjs
- Tailwind CSS v4 -- https://tailwindcss.com/blog/tailwindcss-v4

### Secondary (MEDIUM confidence)
- Homebot feature analysis -- https://help.homebotapp.com/en/articles/3975961-overview-of-the-home-digest
- Homebot competitive positioning -- https://homebot.ai/blog/homebot-vs-alternatives-choosing-the-right-mortgage-marketing-tool
- Zillow Zestimate accuracy -- https://www.zillow.com/learn/influencing-your-zestimate/ (7% median error on off-market homes)
- Scrollytelling best practices (Pudding) -- https://pudding.cool/process/responsive-scrollytelling/
- StoryBrand SB7 implementation -- https://hughesintegrated.com/tips-storybrand-agency/
- Recharts npm -- https://www.npmjs.com/package/recharts (version 3.8.x verified)

### Tertiary (LOW confidence)
- Luxury real estate web design trends 2026 -- https://www.dmrmedia.org/blog/Real-Estate-Website-Design-Trends (industry trend piece, not technical documentation)

---
*Research completed: 2026-03-11*
*Ready for roadmap: yes*
