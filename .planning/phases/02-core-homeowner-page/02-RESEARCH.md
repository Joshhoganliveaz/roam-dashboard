# Phase 2: Core Homeowner Page - Research

**Researched:** 2026-03-12
**Domain:** Next.js 16 server/client component architecture, Recharts data visualization, mobile-first responsive design, SB7 narrative framing
**Confidence:** HIGH

## Summary

Phase 2 transforms the placeholder `/h/[slug]` page into a complete, branded, mobile-first homeowner experience. The foundation is solid: types, calculation engine, Supabase queries, transforms, and defaults all exist from Phase 1. The primary work is (1) building the server component data pipeline, (2) creating 9 client-side section components with SB7 narrative copy, (3) adding an area chart via Recharts, and (4) ensuring mobile-first responsive styling throughout.

The architecture follows a clear server/client split: the `page.tsx` server component fetches from Supabase, runs calculations, and passes computed metrics to a client-side `HomeownerPage` layout component that orchestrates the 9 narrative sections. Recharts requires `"use client"` so the chart must live in its own client component. All styling uses the existing Tailwind 4 CSS-first config with brand color utilities already defined.

**Primary recommendation:** Build as a server component shell that fetches + computes, passing props to a client component tree of 9 narrative sections. Install Recharts for the appreciation chart. Use alternating cream/white backgrounds for visual rhythm. Prioritize mobile viewport for all sizing decisions.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Full-width home photo as hero banner with homeowner name and address overlaid
- Falls back to branded gradient (olive/canyon) when no home photo is uploaded
- Small Live AZ Co logo in top corner of hero
- No data numbers in the hero -- keep it clean and emotional, data reveals on scroll
- Personal note from Josh appears as intro paragraph below the hero, before data sections
- "Last updated" date as subtle stamp near the personal note
- Build-up-to-punchline narrative arc -- start with context, build to the financial climax
- Section order: Hero, Personal note, Purchase story, Current value, Equity gained, Earnings per week, Chart + S&P callout, Rent vs own, Net proceeds
- Subtle alternating background colors (cream/white) between sections for visual rhythm -- no hard dividers
- Area chart (Recharts AreaChart) with filled gradient (olive to gold)
- Interpolated smooth curve from purchase price to current value using annualized appreciation rate
- Annotated markers at purchase point and today
- S&P 500 comparison as separate callout card below chart, not a second line
- Hero number + narrative pattern for every data section
- Escalating narrative energy matching the arc
- Earnings reframing: per-week as the hero number
- Rent vs own: side-by-side contrast with SB7 problem/triumph framing
- Net proceeds: single headline number with expandable breakdown

### Claude's Discretion
- Exact SB7 copy for each section's intro and supporting narrative
- Loading state and error state design
- Exact spacing, typography sizing, and responsive breakpoints
- Chart tooltip behavior and axis formatting
- Empty/missing field handling (what to show when optional data wasn't entered)
- Whether to show all sections or conditionally hide sections with insufficient data

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| DATA-01 | Page displays current home value estimate | `computeHomeownerMetrics()` returns `currentValue` via input passthrough; render in "Today, your home is worth..." section |
| DATA-02 | Page shows equity gained since purchase | `metrics.equityGained` computed from current value minus mortgage balance |
| DATA-03 | Page includes appreciation visualization | Recharts AreaChart with gradient fill; generate data points from purchase price to current value using annualized appreciation rate |
| DATA-04 | Page shows net proceeds estimate | `metrics.netProceeds` with `.gross`, `.agentCommission`, `.closingCosts`, `.mortgagePayoff`, `.net` |
| DATA-05 | Page reframes equity as earnings per week | `metrics.earningsPerWeek` -- display as hero number in "Another way to look at it..." section |
| DATA-06 | Page compares home appreciation vs S&P 500 | `metrics.sp500Comparison` with `.homeReturn`, `.sp500Return`, `.difference` -- render as callout card below chart |
| DATA-07 | Page compares cost of ownership vs renting | `metrics.rentVsOwn` with `.totalRentWouldHavePaid`, `.equityBuilt`, `.netBenefit` |
| NARR-01 | Page uses scrolling narrative timeline | 9-section vertical scroll layout with alternating backgrounds; each section is a narrative beat |
| NARR-02 | SB7 framework -- homeowner is hero, Live AZ Co is guide | Every section opens with narrative context, never raw numbers first; escalating energy |
| NARR-06 | Live AZ Co branding throughout | Existing CSS custom properties (--olive, --canyon, --gold, --cream, --charcoal), font-heading / font-body utilities |
| NARR-07 | Mobile-first responsive design | Design for 375px viewport first, scale up; primary context is text message link on phone |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router, server components | Already installed; project foundation |
| React | 19.2.3 | UI rendering | Already installed |
| Recharts | ^3.8 | AreaChart with gradient fill | Specified in CONTEXT.md; composable React chart library |
| Tailwind CSS | 4.x | Utility-first styling | Already configured with brand colors |
| @supabase/ssr | ^0.8.0 | Server-side Supabase client | Already installed; RLS-aware |
| date-fns | ^4.1.0 | Date formatting and math | Already installed; used in calculations |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Zustand | ^5.0.11 | Client state (expandable sections) | If net proceeds breakdown toggle needs state |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Recharts | Chart.js / Nivo | Recharts is user-locked decision; composable React API fits well |

**Installation:**
```bash
cd ~/Projects/homeowner-journey-map && npm install recharts
```

## Architecture Patterns

### Recommended Project Structure
```
src/
├── app/h/[slug]/
│   └── page.tsx              # Server component: fetch, compute, pass props
├── components/homeowner/
│   ├── HomeownerPage.tsx     # Client orchestrator: receives all data, renders sections
│   ├── HeroSection.tsx       # Hero banner with photo/gradient fallback
│   ├── PersonalNote.tsx      # Josh's intro note + last updated stamp
│   ├── PurchaseStory.tsx     # "Your story begins..." section
│   ├── CurrentValue.tsx      # "Today, your home is worth..." section
│   ├── EquityGained.tsx      # "That means you've built..." section
│   ├── EarningsReframe.tsx   # "Another way to look at it..." section
│   ├── AppreciationChart.tsx # Recharts AreaChart + S&P callout card
│   ├── RentVsOwn.tsx         # Side-by-side rent vs own comparison
│   ├── NetProceeds.tsx       # Headline number + expandable breakdown
│   └── SectionWrapper.tsx    # Shared wrapper for alternating backgrounds
```

### Pattern 1: Server Component Data Pipeline
**What:** The `/h/[slug]/page.tsx` server component handles all data fetching and computation. No data fetching in client components.
**When to use:** Always for this page -- public page with no user interaction beyond scrolling.
**Example:**
```typescript
// src/app/h/[slug]/page.tsx
import { createClient } from "@/lib/supabase/server";
import { rowToHomeownerRecord } from "@/lib/supabase/transforms";
import { applyDefaults } from "@/lib/defaults";
import { computeHomeownerMetrics } from "@/lib/calculations";
import { HomeownerPage } from "@/components/homeowner/HomeownerPage";
import { notFound } from "next/navigation";

export default async function Page({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const supabase = await createClient();

  const { data, error } = await supabase
    .from("homeowners")
    .select("*")
    .eq("slug", slug)
    .eq("status", "published")
    .single();

  if (error || !data) return notFound();

  const record = rowToHomeownerRecord(data);
  const input = applyDefaults(record);
  const metrics = computeHomeownerMetrics(input);

  return <HomeownerPage record={record} metrics={metrics} />;
}
```

### Pattern 2: Section Component with SB7 Narrative
**What:** Each section follows the same structure: narrative intro line, hero number, supporting copy.
**When to use:** Every data section (sections 3-9).
**Example:**
```typescript
// "use client" -- part of HomeownerPage client tree
function EquityGained({ equity }: { equity: number }) {
  return (
    <SectionWrapper bg="white">
      <p className="text-canyon font-body text-lg mb-2">
        That means you have built...
      </p>
      <h2 className="text-charcoal font-heading text-5xl md:text-6xl font-bold mb-4">
        {formatCurrency(equity)}
      </h2>
      <p className="text-charcoal/70 font-body text-lg max-w-md mx-auto">
        in equity since you first opened the front door.
      </p>
    </SectionWrapper>
  );
}
```

### Pattern 3: Alternating Section Backgrounds
**What:** Sections alternate between cream and white backgrounds for visual rhythm without hard dividers.
**When to use:** Wrap every narrative section.
**Example:**
```typescript
function SectionWrapper({ bg, children }: { bg: "cream" | "white"; children: React.ReactNode }) {
  return (
    <section className={`px-6 py-16 md:py-24 ${bg === "cream" ? "bg-cream" : "bg-white"}`}>
      <div className="mx-auto max-w-2xl text-center">
        {children}
      </div>
    </section>
  );
}
```

### Pattern 4: Recharts AreaChart with Gradient (Client Component)
**What:** The appreciation chart MUST be a client component (`"use client"`). Generate interpolated data points server-side or in the client component from purchase price, current value, and annualized appreciation rate.
**When to use:** The "Your home vs the market" section.
**Example:**
```typescript
"use client";
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer, ReferenceDot } from "recharts";

// Generate data points from purchase to today
function generateChartData(purchasePrice: number, currentValue: number, purchaseDate: string, annualizedRate: number) {
  const start = new Date(purchaseDate);
  const now = new Date();
  const points = [];
  const totalMonths = differenceInMonths(now, start);

  for (let m = 0; m <= totalMonths; m++) {
    const date = new Date(start.getFullYear(), start.getMonth() + m, 1);
    const years = m / 12;
    const value = purchasePrice * Math.pow(1 + annualizedRate, years);
    points.push({ date: format(date, "MMM yyyy"), value: Math.round(value) });
  }
  // Ensure last point is exact current value
  points[points.length - 1].value = currentValue;
  return points;
}
```

### Anti-Patterns to Avoid
- **Fetching data in client components:** The `/h/[slug]` page is public and read-only. All data should come from the server component and be passed as props. No `useEffect` data fetching.
- **Using Recharts in server components:** Recharts uses browser APIs (SVG, DOM measurement). It MUST be in a `"use client"` component.
- **Hard pixel breakpoints for mobile:** Use Tailwind's responsive prefixes (`md:`, `lg:`) and size with `rem`/`em`, not `px`. Design at 375px width first.
- **SB7 violation -- leading with numbers:** Every section MUST open with a narrative sentence before showing the big number. Never start a section with a dollar amount.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Chart rendering | Custom SVG chart | Recharts AreaChart | Handles responsive sizing, tooltips, gradients, axis formatting |
| Currency formatting | Manual string formatting | `Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' })` | Handles commas, decimals, negative values correctly |
| Date formatting | Manual date string manipulation | `date-fns format()` | Already a dependency; handles edge cases |
| Responsive images | Custom image loading | Next.js `<Image>` component | Handles srcset, lazy loading, blur placeholder, format optimization |
| 404 handling | Custom error rendering | `notFound()` from `next/navigation` | Built-in Next.js pattern; renders nearest not-found.tsx |

**Key insight:** The calculation engine is already built and tested. Phase 2 is purely a rendering/presentation layer. Resist the temptation to add calculation logic in components -- all numbers come from `computeHomeownerMetrics()`.

## Common Pitfalls

### Pitfall 1: Recharts SSR Hydration Mismatch
**What goes wrong:** Recharts uses browser-only APIs. If imported in a server component or if the server render doesn't match client render, you get hydration errors.
**Why it happens:** Next.js App Router defaults to server components. Recharts measures DOM dimensions on mount.
**How to avoid:** Mark the chart component with `"use client"`. Use `<ResponsiveContainer>` for sizing. Consider wrapping in a component that only renders on client if SSR issues persist.
**Warning signs:** "Text content does not match" or "Hydration failed" console errors.

### Pitfall 2: Hero Image Sizing on Mobile
**What goes wrong:** Full-width hero images look great on desktop but can be too tall, too slow, or poorly cropped on mobile.
**Why it happens:** Home photos are uploaded at arbitrary aspect ratios.
**How to avoid:** Use `object-cover` with a fixed height container (e.g., `h-64 md:h-96`). Use Next.js `<Image>` with `fill` and `sizes` prop for responsive loading. Set `priority` on the hero image since it's above the fold.
**Warning signs:** Layout shift on page load, slow LCP.

### Pitfall 3: Large Numbers Overflowing on Mobile
**What goes wrong:** Hero numbers like "$1,247,500" in `text-5xl` or larger can overflow the 375px viewport.
**Why it happens:** Fixed font sizes without responsive scaling.
**How to avoid:** Use responsive text sizing (`text-4xl md:text-5xl lg:text-6xl`). For very large numbers, consider `text-3xl` as the mobile base. Test with 7-digit numbers.
**Warning signs:** Horizontal scroll appearing on the page.

### Pitfall 4: Not-Found Page for Unpublished Slugs
**What goes wrong:** Visiting a valid slug that's in "draft" status shows an error instead of a proper 404.
**Why it happens:** The query filters by `status: 'published'` but the error handling doesn't distinguish between "doesn't exist" and "not published."
**How to avoid:** Both cases should return `notFound()`. The RLS policy already restricts public access to published rows only, so the query will return no data for drafts.
**Warning signs:** Error pages shown for draft records.

### Pitfall 5: Chart Data Generation with Very Short Ownership
**What goes wrong:** A homeowner who purchased 1-2 months ago gets a chart with only 1-2 data points, which looks like a dot instead of a curve.
**Why it happens:** Data points generated monthly, but ownership is too short.
**How to avoid:** Set a minimum of ~6 data points by interpolating at smaller intervals (weekly) for short ownership periods. Or show a simplified "too early for a chart" message.
**Warning signs:** Chart renders as a single dot or straight line segment.

## Code Examples

### Currency Formatter Utility
```typescript
// src/lib/format.ts
export function formatCurrency(value: number, decimals = 0): string {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals,
  }).format(value);
}

export function formatPercent(value: number, decimals = 1): string {
  return new Intl.NumberFormat("en-US", {
    style: "percent",
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals,
  }).format(value);
}
```

### Recharts AreaChart with Olive-to-Gold Gradient
```typescript
"use client";
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

export function AppreciationChart({ data }: { data: { date: string; value: number }[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <AreaChart data={data} margin={{ top: 10, right: 10, left: 0, bottom: 0 }}>
        <defs>
          <linearGradient id="appreciationGradient" x1="0" y1="0" x2="0" y2="1">
            <stop offset="5%" stopColor="#5C6B4F" stopOpacity={0.8} />
            <stop offset="95%" stopColor="#C9953E" stopOpacity={0.1} />
          </linearGradient>
        </defs>
        <XAxis dataKey="date" tick={{ fontSize: 12 }} />
        <YAxis
          tickFormatter={(v) => `$${(v / 1000).toFixed(0)}k`}
          tick={{ fontSize: 12 }}
          width={60}
        />
        <Tooltip formatter={(v: number) => formatCurrency(v)} />
        <Area
          type="monotone"
          dataKey="value"
          stroke="#5C6B4F"
          strokeWidth={2}
          fillOpacity={1}
          fill="url(#appreciationGradient)"
        />
      </AreaChart>
    </ResponsiveContainer>
  );
}
```

### Hero with Photo Fallback
```typescript
import Image from "next/image";

function HeroSection({ clientName, address, homePhotoUrl }: {
  clientName: string;
  address: string;
  homePhotoUrl: string | null;
}) {
  return (
    <section className="relative h-64 md:h-96 w-full overflow-hidden">
      {homePhotoUrl ? (
        <Image
          src={homePhotoUrl}
          alt={`${address}`}
          fill
          className="object-cover"
          priority
          sizes="100vw"
        />
      ) : (
        <div className="h-full w-full bg-gradient-to-br from-olive to-canyon" />
      )}
      {/* Dark overlay for text readability */}
      <div className="absolute inset-0 bg-charcoal/40" />
      {/* Content overlay */}
      <div className="absolute inset-0 flex flex-col justify-end p-6 md:p-12">
        <h1 className="text-white font-heading text-3xl md:text-5xl font-bold mb-2">
          {clientName}
        </h1>
        <p className="text-white/80 font-body text-lg">{address}</p>
      </div>
    </section>
  );
}
```

### Expandable Net Proceeds Breakdown
```typescript
"use client";
import { useState } from "react";

function NetProceeds({ data }: { data: HomeownerMetrics["netProceeds"] }) {
  const [expanded, setExpanded] = useState(false);

  return (
    <SectionWrapper bg="cream">
      <p className="text-canyon font-body text-lg mb-2">
        If you sold today, you would walk away with...
      </p>
      <h2 className="text-charcoal font-heading text-5xl md:text-6xl font-bold mb-4">
        {formatCurrency(data.net)}
      </h2>
      <button
        onClick={() => setExpanded(!expanded)}
        className="text-olive font-body text-sm underline underline-offset-4"
      >
        {expanded ? "Hide breakdown" : "See the breakdown"}
      </button>
      {expanded && (
        <div className="mt-6 text-left max-w-sm mx-auto space-y-2 text-charcoal/70">
          <div className="flex justify-between">
            <span>Sale price</span>
            <span>{formatCurrency(data.gross)}</span>
          </div>
          <div className="flex justify-between">
            <span>Agent commission</span>
            <span>-{formatCurrency(data.agentCommission)}</span>
          </div>
          <div className="flex justify-between">
            <span>Closing costs</span>
            <span>-{formatCurrency(data.closingCosts)}</span>
          </div>
          <div className="flex justify-between">
            <span>Mortgage payoff</span>
            <span>-{formatCurrency(data.mortgagePayoff)}</span>
          </div>
          <hr className="border-charcoal/20" />
          <div className="flex justify-between font-bold text-charcoal">
            <span>Net proceeds</span>
            <span>{formatCurrency(data.net)}</span>
          </div>
        </div>
      )}
    </SectionWrapper>
  );
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Recharts 2.x | Recharts 3.x | 2025 | React 19 compatibility, improved tree-shaking, better TypeScript types |
| `getServerSideProps` | Server Components (App Router) | Next.js 13+ | Data fetching in component body, no API route needed |
| CSS Modules / styled-components | Tailwind CSS 4 (CSS-first) | 2025 | `@theme` directive replaces JS config; CSS custom properties for brand tokens |

**Deprecated/outdated:**
- Recharts 2.x: Still works but 3.x has better React 19 support
- `pages/` directory: Project uses App Router exclusively

## Open Questions

1. **Home photo URL source**
   - What we know: `homePhotoUrl` is stored as a string in the DB. PhotoUpload component exists in admin.
   - What's unclear: Whether this is a Supabase Storage public URL or an external URL. This affects how to construct the `src` for `<Image>`.
   - Recommendation: Check if Supabase Storage is configured; if so, use `supabase.storage.from('photos').getPublicUrl()`. Otherwise, treat as external URL and configure `next.config.ts` `images.remotePatterns`.

2. **SB7 copy specifics**
   - What we know: User wants escalating narrative energy, SB7 framing, never lead with numbers.
   - What's unclear: Exact copy for each section.
   - Recommendation: This is Claude's discretion per CONTEXT.md. Write copy that mirrors how Josh would tell the story in person. Keep it warm, confident, and personal.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.x with jsdom |
| Config file | `vitest.config.ts` (exists) |
| Quick run command | `npm test` |
| Full suite command | `npm test` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| DATA-01 | Current value displayed on page | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "current value"` | No -- Wave 0 |
| DATA-02 | Equity gained displayed | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "equity"` | No -- Wave 0 |
| DATA-03 | Appreciation chart renders | smoke (render) | `npx vitest run src/components/homeowner/AppreciationChart.test.tsx` | No -- Wave 0 |
| DATA-04 | Net proceeds displayed | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "net proceeds"` | No -- Wave 0 |
| DATA-05 | Earnings per week displayed | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "earnings"` | No -- Wave 0 |
| DATA-06 | S&P comparison callout | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "S&P"` | No -- Wave 0 |
| DATA-07 | Rent vs own comparison | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "rent"` | No -- Wave 0 |
| NARR-01 | Scrolling narrative with 9 sections | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "sections"` | No -- Wave 0 |
| NARR-02 | SB7 framing (narrative before numbers) | manual-only | Visual review | N/A |
| NARR-06 | Brand colors and fonts | manual-only | Visual review | N/A |
| NARR-07 | Mobile-first responsive | manual-only | Browser dev tools responsive mode | N/A |

### Sampling Rate
- **Per task commit:** `npm test`
- **Per wave merge:** `npm test && npm run build`
- **Phase gate:** Full suite green + visual review on mobile viewport before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `src/components/homeowner/HomeownerPage.test.tsx` -- render tests covering DATA-01 through DATA-07, NARR-01
- [ ] `src/components/homeowner/AppreciationChart.test.tsx` -- chart data generation tests for DATA-03
- [ ] `src/lib/format.test.ts` -- currency/percent formatter tests
- [ ] Recharts install: `npm install recharts` -- not yet in dependencies

## Sources

### Primary (HIGH confidence)
- Project source code: `~/Projects/homeowner-journey-map/src/` -- all types, calculations, transforms, defaults read directly
- Phase 1 plans and summaries -- architectural decisions and patterns established
- `02-CONTEXT.md` -- user locked decisions on section order, chart type, narrative arc

### Secondary (MEDIUM confidence)
- [Recharts npm](https://www.npmjs.com/package/recharts) -- v3.8.0 latest, React 19 compatible
- [Recharts gradient examples](https://leanylabs.com/blog/awesome-react-charts-tips/) -- linearGradient pattern for AreaChart
- [Next.js server/client component docs](https://nextjs.org/docs/app/getting-started/server-and-client-components) -- Recharts requires "use client"

### Tertiary (LOW confidence)
- None

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all libraries already in project or well-documented (Recharts)
- Architecture: HIGH -- follows established project patterns from Phase 1; server/client split is standard Next.js
- Pitfalls: HIGH -- SSR/hydration issues with chart libraries are well-documented; mobile sizing is straightforward to test

**Research date:** 2026-03-12
**Valid until:** 2026-04-12 (stable stack, no fast-moving dependencies)
