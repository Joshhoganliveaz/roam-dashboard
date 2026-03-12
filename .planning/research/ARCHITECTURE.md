# Architecture Patterns

**Domain:** Personalized homeowner journey page generation system
**Researched:** 2026-03-11

## Recommended Architecture

**Pattern: Admin-driven static generation with on-demand ISR via Next.js App Router + Supabase**

This system has five distinct components connected by a clear data pipeline: Admin Input Form --> Supabase Database --> Calculation Engine --> Narrative Engine --> Rendered Page. The admin creates a homeowner record, calculations run server-side at render time, the narrative engine selects and populates SB7 templates based on computed data, and Next.js serves the result as a statically-generated page with on-demand revalidation.

```
+------------------+       +------------------+       +------------------+
|   Admin Form     |  -->  |    Supabase      |  -->  | Next.js Dynamic  |
|   /admin/new     |       |   homeowners     |       | Route /h/[slug]  |
|   /admin/[id]    |       |   table          |       |                  |
+------------------+       +------------------+       +------------------+
                                    |                         |
                                    v                         v
                           +------------------+    +-------------------+
                           | Calculation      |    | Narrative Engine  |
                           | Engine           |--> | (SB7 Templates)   |
                           | (equity, apprc,  |    | Duration-adaptive |
                           |  scenarios)      |    | section selection |
                           +------------------+    +-------------------+
                                                          |
                                                          v
                                                 +-------------------+
                                                 | Scrolling Page    |
                                                 | Mobile-first      |
                                                 | Branded, sharable |
                                                 +-------------------+
```

### Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| **Admin Form** (`/admin`) | Input homeowner data (name, address, purchase price, purchase date); edit existing records; trigger revalidation | Supabase (write), Next.js revalidation API (POST) |
| **Supabase Database** | Store homeowner records, slugs, timestamps; row-level security (admin write, public read) | Admin Form (receives writes), Dynamic Route (serves reads) |
| **Calculation Engine** (`src/lib/calculations.ts`) | Pure functions: equity computation, appreciation over time, S&P comparison, rental break-even, move-up scenarios, HELOC potential | Receives raw homeowner data, outputs structured calculation results |
| **Narrative Engine** (`src/lib/narrative.ts`) | Select SB7 story beats based on ownership duration; populate templates with calculated values; generate section ordering and copy | Receives calculation results + homeowner metadata, outputs structured page content |
| **Page Renderer** (`/h/[slug]/page.tsx`) | Server Component that fetches data, runs calculations, generates narrative, renders scrolling timeline | Supabase (read), Calculation Engine, Narrative Engine |

### Data Flow

```
1. ADMIN INPUT
   Josh fills form: { name, address, purchasePrice, purchaseDate, currentValue }
   Form POST --> Supabase insert/update
   Form triggers: POST /api/revalidate?slug=smith-123-main-st

2. DATA STORAGE (Supabase)
   homeowners table: {
     id, slug, client_name, address, city, state, zip,
     purchase_price, purchase_date, current_value,
     loan_amount, interest_rate, loan_term,
     created_at, updated_at, status
   }

3. PAGE REQUEST (visitor clicks link from text/email)
   GET /h/smith-123-main-st
   --> Next.js dynamic route [slug]
   --> Server Component fetches from Supabase by slug
   --> Passes raw data to Calculation Engine
   --> Passes results to Narrative Engine
   --> Renders React components
   --> Cached as static HTML (ISR)

4. CALCULATION ENGINE (pure functions, no side effects)
   Input: { purchasePrice, purchaseDate, currentValue, loanDetails }
   Output: {
     equityGained, appreciationPct, appreciationAnnualized,
     equityPerMonth, equityPerWeek,
     vsRenting: { totalRentPaid, equityDifference },
     vsSP500: { sp500Return, homeReturn },
     scenarios: {
       holdAndRent: { monthlyRent, mortgageCoverage, breakEvenDate },
       moveUp: { netProceeds, buyingPower, upgradeOptions },
       equityPlay: { helocAvailable, leverageScenario }
     },
     timeline: [ { date, event, value } ... ]
   }

5. NARRATIVE ENGINE (SB7 template selection)
   Input: calculationResults + { ownershipDuration, clientName }
   Logic:
     - duration < 1yr  --> "Where You're Headed" (forward-looking)
     - duration 1-3yr  --> "Building Momentum" (balanced)
     - duration 3-5yr  --> "The Story So Far" (retrospective + forward)
     - duration 5yr+   --> "Your Journey" (rich retrospective)
   Output: ordered array of section objects with:
     { sectionType, headline, body, dataPoints, calloutBox }

6. RENDERING
   Server Components map narrative sections to React components.
   Each section type has its own component:
     TimelineSection, EquitySection, ComparisonSection,
     ScenarioCard, CalloutBox, HeroSection, CTASection
```

## Rendering Strategy: On-Demand ISR

**Why not SSG (full static build)?** At 200+ pages, build times grow linearly. More importantly, pages are created ad-hoc (20-30/month) -- they don't exist at build time. SSG requires a full rebuild for each new page.

**Why not SSR (server-render every request)?** These pages are read-heavy, write-rare. A homeowner might view their page 5-10 times, share it with a spouse -- SSR wastes compute on identical renders. The data changes only when Josh updates it.

**Why on-demand ISR?** Best of both worlds:
- Pages are statically generated on first request (or on admin save via `revalidatePath`)
- Subsequent visits serve cached static HTML (fast, zero compute)
- When Josh edits a record, a revalidation call regenerates just that one page
- No full rebuilds ever needed
- Next.js App Router supports this natively with `dynamicParams = true` and `revalidatePath()`

```typescript
// /h/[slug]/page.tsx
export const dynamicParams = true; // Allow pages not in generateStaticParams

export async function generateStaticParams() {
  // Return empty array -- pages generate on-demand
  return [];
}

export default async function HomeownerPage({ params }: { params: { slug: string } }) {
  const { slug } = await params;
  const supabase = await createClient();
  const { data: homeowner } = await supabase
    .from('homeowners')
    .select('*')
    .eq('slug', slug)
    .eq('status', 'active')
    .single();

  if (!homeowner) notFound();

  const calculations = computeHomeownerMetrics(homeowner);
  const narrative = generateNarrative(calculations, homeowner);

  return <HomeownerJourney narrative={narrative} calculations={calculations} />;
}
```

## Patterns to Follow

### Pattern 1: Pure Calculation Engine (Proven in STR Analyzer)
**What:** All financial math lives in a single file of pure functions with no side effects, no database calls, no React dependencies.
**When:** Always -- this is the core differentiator of the product.
**Why:** The STR Analyzer's `calculator.ts` already proves this pattern works at Josh's scale. Pure functions are testable, portable, and can be reused across admin previews and public pages.

```typescript
// src/lib/calculations.ts
export function computeHomeownerMetrics(input: HomeownerInput): HomeownerMetrics {
  const ownershipMonths = monthsBetween(input.purchaseDate, new Date());
  const equityGained = input.currentValue - input.purchasePrice;
  const appreciationPct = equityGained / input.purchasePrice;
  // ... more calculations
  return { ownershipMonths, equityGained, appreciationPct, /* ... */ };
}
```

### Pattern 2: Narrative Engine as Data, Not Markup
**What:** The narrative engine returns structured data (section objects), not HTML or JSX. Components consume the data.
**When:** Always -- separating narrative logic from presentation allows reusing the same narrative for different outputs (web page, future PDF, future email digest).

```typescript
// src/lib/narrative.ts
interface NarrativeSection {
  type: 'hero' | 'timeline' | 'equity' | 'comparison' | 'scenario' | 'cta';
  headline: string;
  subheadline?: string;
  body: string;
  dataPoints: DataPoint[];
  calloutBox?: { text: string; implication: string };
}

export function generateNarrative(
  metrics: HomeownerMetrics,
  homeowner: HomeownerRecord
): NarrativeSection[] {
  const duration = categorizeOwnership(metrics.ownershipMonths);
  const sections: NarrativeSection[] = [];

  // Always start with hero
  sections.push(buildHeroSection(homeowner, metrics, duration));

  // Duration-adaptive middle sections
  if (duration === 'recent') {
    sections.push(buildWhereYoureHeaded(metrics));
  } else {
    sections.push(buildTimelineSection(metrics, homeowner));
    sections.push(buildEquitySection(metrics));
  }

  // Always include comparisons and scenarios
  sections.push(buildComparisonSection(metrics));
  sections.push(...buildScenarioSections(metrics, duration));
  sections.push(buildCTASection(homeowner));

  return sections;
}
```

### Pattern 3: Slug Generation from Address
**What:** Generate URL-safe slugs from client name + address for unique, readable URLs.
**When:** On admin form submission.

```typescript
// smith-family-123-e-main-st-phoenix
function generateSlug(name: string, address: string, city: string): string {
  return `${name}-${address}-${city}`
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/-+/g, '-')
    .replace(/^-|-$/g, '');
}
```

### Pattern 4: Admin Form with Live Preview
**What:** The admin form shows a simplified preview of the page as data is entered, using the same calculation and narrative engines.
**When:** During page creation -- gives Josh confidence the output is correct before sharing the link.

### Pattern 5: Password-Protected Admin Routes
**What:** Simple cookie-based auth for admin routes, identical to the Idea Pipeline pattern.
**When:** All `/admin` routes. Public `/h/[slug]` routes have no auth.
**Why:** Josh is the only admin. No need for user management, OAuth, or complex auth. The Idea Pipeline already uses this exact pattern.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Client-Side Calculations
**What:** Running calculation engine in the browser via useEffect or useMemo.
**Why bad:** The public page has no interactive inputs -- calculations should run once on the server at render time. Client-side calculation adds bundle size, delays render, and introduces hydration complexity for zero benefit.
**Instead:** Server Components call the calculation engine directly. The result is pre-rendered HTML.

### Anti-Pattern 2: CMS or Headless CMS for 200 Pages
**What:** Using Contentful, Sanity, Strapi, or similar to manage homeowner records.
**Why bad:** Massive overkill. The data model is a single flat table (homeowner records). A CMS adds cost, complexity, vendor dependency, and content modeling overhead for what is essentially a database row + template. Supabase is already in Josh's stack.
**Instead:** Supabase table + custom admin form. Total control, zero monthly CMS cost, same Supabase instance as STR Analyzer.

### Anti-Pattern 3: Separate Microservices for Calc/Narrative
**What:** Deploying calculation and narrative engines as separate API services.
**Why bad:** At 200 pages with write-rare/read-heavy access, the overhead of network calls between services adds latency and deployment complexity for no scaling benefit.
**Instead:** Co-locate everything in one Next.js app. Import calculation and narrative functions directly.

### Anti-Pattern 4: Storing Generated HTML
**What:** Pre-rendering HTML and storing it in the database or S3.
**Why bad:** Creates a cache invalidation problem. When you update the template design, you'd need to regenerate all 200+ pages. With ISR, updating the template code and redeploying automatically serves the new design on next revalidation.
**Instead:** Let Next.js ISR handle caching. The "source of truth" is always data + template, never pre-rendered output.

### Anti-Pattern 5: Heavy Client-Side Animation Libraries
**What:** Using GSAP, Framer Motion, or Three.js for scroll animations.
**Why bad:** Most visitors open via text message on mobile. Heavy JS kills Time-to-Interactive. The page needs to feel instant.
**Instead:** CSS-only scroll animations (`@scroll-timeline`, `animation-timeline: scroll()`, Intersection Observer for reveal effects). Progressive enhancement -- the page works without JS.

## Key Architecture Decisions

### Single App vs. Separate Project
**Recommendation: Separate Next.js project**, not embedded in the STR Analyzer.

Rationale:
- Different domain/branding focus (homeowner-facing vs. investor-facing)
- Independent deployment lifecycle
- Shared code (Supabase client, some calc functions) can be copy-adapted -- at this scale, a monorepo or shared package is overengineering
- Same Supabase instance is fine (separate tables)
- Same tech stack (Next.js 16, React 19, Tailwind 4, Supabase) -- Josh already knows it

### Hosting: Vercel
**Recommendation: Vercel** (primary), with Cloudflare Pages as fallback.

Rationale:
- Native ISR support (Cloudflare Pages via OpenNext works but ISR revalidation is less mature)
- Josh already deploys STR Analyzer to Vercel
- Free tier handles 200 pages easily
- `revalidatePath()` works out of the box
- Edge runtime for fast mobile loads

### Database: Existing Supabase Instance
**Recommendation: Same Supabase project**, new table(s).

Rationale:
- Already provisioned and paid for
- RLS patterns established in STR Analyzer
- No new vendor, no new credentials
- Homeowner data is simple (one table, maybe a second for market benchmarks)

## Suggested Build Order

Build order follows data flow direction -- each component depends on the one before it:

```
Phase 1: Data Layer + Calculation Engine
  |- Supabase table schema (homeowners)
  |- TypeScript types (HomeownerInput, HomeownerMetrics)
  |- Calculation engine (pure functions, unit tested)
  |  Rationale: Everything downstream depends on correct calculations.
  |  Can be built and tested in isolation with no UI.

Phase 2: Admin Input Form
  |- /admin route with password auth
  |- Form to create/edit homeowner records
  |- Slug generation
  |- Supabase insert/update
  |  Rationale: Need data entry before page rendering.
  |  Validates the data model works in practice.

Phase 3: Narrative Engine + Page Rendering
  |- Narrative engine (SB7 template selection, section generation)
  |- Page components (Hero, Timeline, Equity, Scenarios, CTA)
  |- Dynamic route /h/[slug] with ISR
  |- Mobile-first responsive design
  |  Rationale: This is the core product -- but it needs real data
  |  (from Phase 2) and correct numbers (from Phase 1) to be meaningful.

Phase 4: Polish + Production Readiness
  |- Admin preview (live preview as data is entered)
  |- Revalidation on edit (auto-refresh cached pages)
  |- OG image / social sharing metadata
  |- Analytics (simple view tracking)
  |- Error handling (404 for invalid slugs, error boundaries)
  |  Rationale: Polish layer that makes the product production-ready
  |  but doesn't block initial testing with real homeowner data.
```

**Critical dependency chain:** Calculation Engine --> Narrative Engine --> Page Components. The calculation engine must be correct and tested before narrative templates can reference its outputs. The narrative engine must produce structured data before page components can render it.

## Scalability Considerations

| Concern | At 200 pages (Year 1) | At 1,000 pages (Year 2) | At 5,000+ pages |
|---------|----------------------|------------------------|-----------------|
| Build time | N/A (on-demand ISR) | N/A (on-demand ISR) | N/A (on-demand ISR) |
| Storage | ~200 rows in Supabase (trivial) | ~1,000 rows (trivial) | Consider archiving old pages |
| Hosting cost | Vercel free tier | Vercel free/pro tier | Vercel pro tier |
| Admin UX | List view with search | Pagination + search | Bulk operations, CSV import |
| Page load | Instant (static cache) | Instant (static cache) | Instant (static cache) |
| Data freshness | Manual update + revalidate | Same, or add scheduled revalidation | Consider automated data feeds |

At the target scale of 200-1,000 pages, this architecture has no scaling concerns. ISR means each page is generated once and served from CDN. The database is a single small table. The bottleneck is Josh's input speed, not system performance.

## Sources

- [Next.js generateStaticParams documentation](https://nextjs.org/docs/app/api-reference/functions/generate-static-params) - HIGH confidence
- [Next.js ISR guide](https://nextjs.org/docs/pages/guides/incremental-static-regeneration) - HIGH confidence
- [Next.js rendering strategies comparison (DEV Community)](https://dev.to/rayan2228/nextjs-rendering-strategies-csr-vs-ssr-vs-ssg-vs-isr-complete-guide-26j4) - MEDIUM confidence
- [Builder.io ISR overview](https://www.builder.io/glossary/incremental-static-regeneration) - MEDIUM confidence
- STR Analyzer codebase at `~/stranalyzer/str-analyzer/` - HIGH confidence (direct code review)
- Idea Pipeline auth pattern at `~/Projects/liveaz-idea-pipeline/` - HIGH confidence (existing pattern)
