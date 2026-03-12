# Stack Research

**Domain:** Personalized, data-driven, shareable homeowner journey pages at scale
**Researched:** 2026-03-11
**Confidence:** HIGH

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Next.js (App Router) | 16.1.6 | Framework, routing, page generation | Josh already runs Next.js 16.1.6 on STR Analyzer. Same framework means shared knowledge, shared deploy pipeline, and no new learning curve. App Router with dynamic routes (`/[slug]`) + ISR gives static-level performance with on-demand regeneration when homeowner data changes. |
| React | 19.x | UI rendering | Already in use across projects. Server Components let us fetch homeowner data server-side and ship zero JS for data-fetching logic. |
| TypeScript | 5.x | Type safety | Already in use. Type interfaces for homeowner data, calculation results, and page props prevent bugs at scale (200+ pages with varying data shapes). |
| Tailwind CSS | 4.x | Styling | Already in use on STR Analyzer. CSS-first config in v4 is simpler. Live AZ Co brand colors map directly to CSS custom properties. Mobile-first utility classes are the fastest way to build responsive scrolling pages. |
| Supabase | 2.x (@supabase/supabase-js) | Database + storage | Already in use on STR Analyzer with RLS. PostgreSQL gives relational structure for homeowners, properties, and page metadata. Row-Level Security is overkill for v1 (Josh is the only editor) but sets up correctly for future. Free tier handles this volume easily. |

### Page Generation Strategy

| Approach | When | Why |
|----------|------|-----|
| Dynamic routes with ISR | All homeowner pages (`/journey/[slug]`) | `revalidate = 3600` (1 hour) gives static performance. On-demand revalidation via `revalidateTag()` when Josh updates homeowner data. Pages are generated on first visit and cached. 200+ pages is trivial for this approach. |
| Server Components | Data fetching + page rendering | Homeowner pages are read-only for visitors. Server Components fetch from Supabase, render HTML, ship minimal JS. Only interactive elements (animations, charts) need client components. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Motion (formerly Framer Motion) | 12.x | Scroll-triggered animations, section reveals | The narrative timeline needs scroll-driven entrance animations for each "chapter" of the homeowner story. Motion has first-class `useScroll`, `useInView`, and scroll-linked animation APIs. 32KB gzipped. React-native declarative API fits this project perfectly -- no GSAP timeline complexity needed. |
| Recharts | 3.8.x | Equity growth charts, appreciation visualization | Declarative React components for line/area charts. ResponsiveContainer handles mobile sizing automatically. Already popular (3.6M weekly downloads), well-documented, SVG-based so it renders crisply on all devices. |
| @supabase/ssr | 0.8.x | Server-side Supabase client for Next.js | Proper cookie-based auth for the admin input form. Already in use on STR Analyzer. |
| Zustand | 5.x | Admin form state management | For the page-creation form where Josh inputs homeowner data. Already in use on STR Analyzer. Lightweight, no boilerplate. |
| date-fns | 4.x | Date calculations | Ownership duration, monthly appreciation, timeline generation. Tree-shakeable (only import what you use), unlike moment.js. |

### Infrastructure

| Technology | Purpose | Why |
|------------|---------|-----|
| Vercel | Primary hosting + deployment | Next.js ISR works natively on Vercel with zero config. Auto-deploys on push. Edge caching gives fast page loads globally. Free tier supports this volume. Josh already deploys STR Analyzer here. |
| Cloudflare Pages | Backup/alternative deploy target | Already configured for STR Analyzer via @opennextjs/cloudflare. Note: 25MB worker bundle limit can be a constraint. Use Vercel as primary. |
| Supabase (hosted) | PostgreSQL database | Free tier: 500MB database, 1GB file storage, 50K monthly active users. More than enough for 200+ homeowner records. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| ESLint 9 + eslint-config-next | Linting | Already configured in STR Analyzer |
| Prettier | Formatting | Standard for Next.js projects |
| Vitest | Testing | Already used in Idea Pipeline project. Fast, Vite-native, good Next.js compatibility |

## Database Schema (Supabase/PostgreSQL)

```sql
-- Core table: one row per homeowner page
homeowner_pages (
  id uuid primary key,
  slug text unique not null,        -- URL slug: "smith-family-123-main-st"
  client_name text not null,
  address text not null,
  city text default 'Phoenix',
  purchase_price numeric not null,
  purchase_date date not null,
  current_value numeric,            -- manually entered or calculated
  annual_appreciation_rate numeric default 0.05,
  monthly_rent_estimate numeric,    -- for "Hold & Rent" scenario
  notes text,                       -- agent notes for narrative customization
  ownership_tier text,              -- 'recent' | 'mid-term' | 'long-term' (computed)
  is_published boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
)
```

## Installation

```bash
# Core (matches STR Analyzer versions)
npm install next@16.1.6 react@19.2.3 react-dom@19.2.3

# Database
npm install @supabase/supabase-js@^2.97.0 @supabase/ssr@^0.8.0

# UI & Animation
npm install motion recharts zustand@^5.0.11

# Utilities
npm install date-fns

# Dev dependencies
npm install -D tailwindcss@^4 @tailwindcss/postcss@^4 typescript@^5 @types/react@^19 @types/react-dom@^19 @types/node@^20 eslint@^9 eslint-config-next@16.1.6

# Deploy (if using Cloudflare as secondary)
npm install @opennextjs/cloudflare@^1.17.1
npm install -D wrangler@^4.68.1
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Next.js App Router + ISR | Astro (static site generator) | If pages were truly static with no server-side data fetching and no admin UI. Astro ships zero JS by default, which is great for pure content sites. But we need: (1) an admin form to create pages, (2) on-demand revalidation when data changes, (3) React component ecosystem for charts/animations. Next.js handles all three. Also: Josh already knows Next.js. |
| Next.js App Router + ISR | Static HTML generation (build script) | If volume were under 20 pages and pages never changed. At 200+ pages with ongoing additions, ISR is the right abstraction. |
| Supabase | SQLite / Turso | If the project had no existing Supabase setup. For a single-user admin creating 200 records, SQLite would work. But Josh already has Supabase running on STR Analyzer, so reuse beats novelty. |
| Supabase | JSON files in repo | If pages were fewer than 50 and data never changed. At 200+ pages with ongoing updates, a database is warranted. |
| Motion | GSAP + ScrollTrigger | If the animations were cinematic/complex (morphing, physics, timeline choreography). For section fade-ins and scroll-triggered reveals, Motion is simpler, more React-idiomatic, and sufficient. GSAP adds 48KB and imperative API complexity. |
| Motion | CSS scroll-driven animations | If browser support were broader. CSS `animation-timeline: scroll()` is powerful but Safari support is still limited (Safari 16.4+ only partial). Motion provides a consistent cross-browser abstraction. |
| Recharts | Visx (Airbnb) | If you needed fully custom chart designs with unusual layouts. Visx gives lower-level D3 primitives but requires more code for standard charts. Recharts gives you a responsive area chart in 10 lines of JSX. |
| Vercel | Self-hosted / VPS | Never for this project. ISR requires edge infrastructure. Vercel provides this natively. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Gatsby | Effectively deprecated. Netlify acquired it and development has stalled. Build times scale poorly with page count. | Next.js |
| Create React App | Deprecated since 2023. No SSR, no ISR, no file-based routing. | Next.js |
| Moment.js | 330KB, mutable API, officially in maintenance mode. | date-fns (tree-shakeable, immutable) |
| Chart.js / react-chartjs-2 | Canvas-based rendering looks blurry on high-DPI mobile screens. Less React-idiomatic than Recharts. | Recharts (SVG-based, responsive) |
| Prisma | Adds ORM complexity unnecessary for this simple schema. Supabase JS client is simpler for CRUD on 1-2 tables. | @supabase/supabase-js direct queries |
| WordPress / headless CMS | Massive overkill. This is not a content management problem -- it is a data-driven page generation problem. A CMS adds auth, content modeling, plugin management overhead for zero benefit. | Supabase + Next.js |
| AOS (Animate On Scroll) | jQuery-era library. No React integration. Limited animation types. | Motion |
| framer-motion (old package name) | Rebranded to "motion" in late 2024. Old package still works but new projects should use the new package name. | motion |

## Stack Patterns by Variant

**If homeowner pages need real-time data updates later (v2+):**
- Add Supabase Realtime subscriptions
- Switch from ISR to SSR for those specific pages
- Because real-time home values would need live data, not cached pages

**If Josh wants other agents to create pages (multi-user admin):**
- Add Supabase Auth with RLS policies
- Add role-based access (admin vs viewer)
- Because RLS is already supported by the Supabase setup

**If page volume exceeds 1,000+:**
- Current ISR approach scales fine -- Next.js handles this natively
- Consider adding a search/index page with pagination
- Because ISR generates pages on-demand, not at build time

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| next@16.1.6 | react@19.x, react-dom@19.x | Must use React 19. React 18 is not compatible with Next.js 16. |
| tailwindcss@4.x | @tailwindcss/postcss@4.x | v4 uses CSS-first config (no tailwind.config.js). PostCSS plugin required. |
| motion@12.x | react@18+ | Works with React 18 and 19. No compatibility issues. |
| recharts@3.x | react@18+, react-dom@18+ | Recharts 3.x supports React 18 and 19. |
| @supabase/ssr@0.8.x | next@14+, @supabase/supabase-js@2.x | Designed for Next.js App Router. Works with 14, 15, and 16. |
| zustand@5.x | react@18+ | No compatibility issues with React 19. |
| @opennextjs/cloudflare@1.x | next@15+, next@16.x | Supports Next.js 16. Next.js 14 support dropped Q1 2026. |

## Why This Stack Works for This Project

1. **Zero new learning curve.** Every core technology is already in Josh's STR Analyzer. The new additions (Motion, Recharts, date-fns) are straightforward libraries, not paradigm shifts.

2. **Fast page creation (under 5 minutes).** Admin form submits to Supabase, triggers ISR revalidation, page is live. No build step, no deploy, no waiting.

3. **Mobile-first by default.** Tailwind CSS utilities + Recharts ResponsiveContainer + Motion's declarative animations all work naturally on mobile viewports.

4. **Scales to 200+ pages trivially.** ISR generates pages on-demand and caches them. No build-time page generation bottleneck.

5. **Shareable URLs.** Each page gets a clean slug (`/journey/smith-family-123-main-st`). No auth required for viewers. Open Graph meta tags make link previews rich when shared via text/email.

## Sources

- Next.js 16 releases: https://github.com/vercel/next.js/releases -- verified current version is 16.1.6 (HIGH confidence)
- Next.js ISR documentation: https://nextjs.org/docs/pages/guides/incremental-static-regeneration (HIGH confidence)
- Motion (formerly Framer Motion): https://motion.dev/ -- rebranded late 2024, current version 12.x (HIGH confidence)
- Recharts npm: https://www.npmjs.com/package/recharts -- current version 3.8.0 (HIGH confidence)
- Tailwind CSS v4: https://tailwindcss.com/blog/tailwindcss-v4 -- current version 4.2.1 (HIGH confidence)
- Supabase + Next.js quickstart: https://supabase.com/docs/guides/getting-started/quickstarts/nextjs (HIGH confidence)
- OpenNext Cloudflare adapter: https://opennext.js.org/cloudflare -- supports Next.js 16 (HIGH confidence)
- STR Analyzer package.json (local): verified exact versions Josh currently uses (HIGH confidence)

---
*Stack research for: Homeowner Journey Map*
*Researched: 2026-03-11*
