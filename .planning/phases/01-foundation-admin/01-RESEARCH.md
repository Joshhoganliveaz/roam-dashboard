# Phase 1: Foundation + Admin - Research

**Researched:** 2026-03-11
**Domain:** Next.js 16 project scaffolding, Supabase schema, financial calculation engine, admin form with ARMLS CSV upload, slug-based URL system
**Confidence:** HIGH

## Summary

Phase 1 builds the greenfield Next.js 16 project that powers the Homeowner Journey Map. It delivers four things: (1) a Supabase-backed data schema for homeowner records with slug-based URLs, (2) a pure calculation engine for equity, appreciation, net proceeds, earnings-per-period, S&P comparison, rent-vs-own, and three future scenario projections, (3) a password-protected admin interface where Josh can create/edit homeowner pages with manual entry or ARMLS CSV upload, and (4) a page management list view with search/sort/filter for 200+ records.

The stack mirrors the STR Analyzer exactly -- Next.js 16.1.6, React 19, TypeScript 5, Tailwind CSS 4, Supabase, Zustand -- so there is zero learning curve. The calculation engine follows the proven pure-function pattern from `calculator.ts` in the STR Analyzer. The admin auth reuses the cookie-based password middleware from the Idea Pipeline. The Supabase schema follows the RLS pattern from the STR Analyzer but adapted for single-admin with public read access.

**Primary recommendation:** Scaffold the project first, then build the calculation engine with full unit test coverage, then the Supabase schema + admin form, and finally the CSV upload. The calculation engine is the foundation everything else depends on and the most testable component.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Core admin form fields: address, client name, purchase price, purchase date, current value
- Loan details: original loan amount, interest rate, loan term (optional with smart defaults)
- Monthly rent estimate (optional with smart defaults)
- Photo upload: support both homeowner photo and home photo (either or both)
- Personal note: free-text field for a personal message or context
- ARMLS CSV/PDF upload that auto-fills property data and value -- all fields remain editable after auto-fill
- Name-based slugs: `/h/smith-family` -- personal, memorable, easy to text
- Domain-agnostic build (figure out domain later)
- Primary value source: ARMLS listing PDF/CSV upload; fallback: manual entry
- All CSV-populated fields are editable before saving
- One URL per homeowner, forever -- updates add new chapters, not new pages
- "Last updated" date visible on the page
- CSV upload from day one (production workflow requirement, not v2)
- Pages are living documents that grow richer with each update

### Claude's Discretion
- Whether loan details and rent estimate are required or optional with smart defaults
- Duplicate slug handling strategy
- Page lifecycle states (draft -> published vs always-live)
- Admin list view design (search, sort, filter approach)
- Exact form layout and field grouping

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| PROD-01 | Admin input form creates a new page in under 5 minutes | Admin form design with minimal required fields, CSV auto-fill for speed, smart defaults for optional fields, Zustand for form state, Supabase insert on save |
| PROD-02 | Each homeowner page has a unique shareable URL | Name-based slug generation (`/h/smith-family`), unique constraint in Supabase, duplicate handling with auto-suffix |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js (App Router) | 16.1.6 | Framework | Exact version from STR Analyzer. App Router for dynamic routes, Server Components, API routes for admin |
| React | 19.2.3 | UI | Required by Next.js 16. Server Components for public pages, Client Components for admin form |
| TypeScript | 5.x | Type safety | Already in use. Interfaces for HomeownerRecord, CalculationResults, AdminFormState |
| Tailwind CSS | 4.x | Styling | CSS-first config in v4. Brand colors as CSS custom properties |
| @supabase/supabase-js | ^2.97.0 | Database client | Exact version from STR Analyzer |
| @supabase/ssr | ^0.8.0 | Server-side Supabase for Next.js | Cookie-based client for App Router. Exact version from STR Analyzer |
| Zustand | ^5.0.11 | Admin form state | Lightweight state management for form with many fields. Already proven in STR Analyzer |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| date-fns | 4.x | Date calculations | Ownership duration, months between dates, date formatting. Tree-shakeable |
| Vitest | ^4.0.x | Unit testing | Calculation engine tests. Already used in Idea Pipeline with working config to reference |
| @vitejs/plugin-react | latest | Vitest React support | Required for Vitest with React/Next.js |
| papaparse | ^5.x | CSV parsing | ARMLS CSV file parsing in the browser. Well-established, handles edge cases (quoted fields, encoding) |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| papaparse | Built-in FileReader + manual CSV parsing | PapaParse handles encoding, quoted fields, header detection. Manual parsing breaks on edge cases |
| Zustand | React Hook Form | React Hook Form is more form-specific but Josh already knows Zustand from STR Analyzer |
| Vitest | Jest | Vitest is faster, already configured in Idea Pipeline, native ESM support |

**Installation:**
```bash
# Core
npm install next@16.1.6 react@19.2.3 react-dom@19.2.3
npm install @supabase/supabase-js@^2.97.0 @supabase/ssr@^0.8.0
npm install zustand@^5.0.11 date-fns papaparse

# Dev
npm install -D tailwindcss@^4 @tailwindcss/postcss@^4 typescript@^5
npm install -D @types/react@^19 @types/react-dom@^19 @types/node@^20
npm install -D eslint@^9 eslint-config-next@16.1.6
npm install -D vitest @vitejs/plugin-react @types/papaparse
```

## Architecture Patterns

### Recommended Project Structure
```
src/
├── app/
│   ├── layout.tsx              # Root layout with fonts, metadata
│   ├── page.tsx                # Redirect to /admin or landing
│   ├── admin/
│   │   ├── layout.tsx          # Admin layout (password-protected via middleware)
│   │   ├── page.tsx            # Admin list view -- all homeowner pages
│   │   ├── new/
│   │   │   └── page.tsx        # Create new homeowner page
│   │   └── [id]/
│   │       └── page.tsx        # Edit existing homeowner page
│   ├── login/
│   │   └── page.tsx            # Password login page
│   ├── api/
│   │   ├── login/
│   │   │   └── route.ts        # POST: validate password, set cookie
│   │   └── revalidate/
│   │       └── route.ts        # POST: trigger ISR revalidation for a slug
│   └── h/
│       └── [slug]/
│           └── page.tsx        # Public homeowner page (Phase 2 renders here)
├── lib/
│   ├── calculations.ts         # Pure calculation functions (no side effects)
│   ├── calculations.test.ts    # Unit tests for all calculations
│   ├── types.ts                # TypeScript interfaces
│   ├── slugify.ts              # Slug generation and duplicate handling
│   ├── csv-parser.ts           # ARMLS CSV parsing logic
│   ├── defaults.ts             # Smart defaults for optional fields
│   ├── supabase/
│   │   ├── client.ts           # Browser client (admin form)
│   │   ├── server.ts           # Server client (page rendering, API routes)
│   │   └── schema.sql          # Database schema (reference)
│   └── store.ts                # Zustand store for admin form
├── components/
│   └── admin/
│       ├── HomeownerForm.tsx   # Main form component
│       ├── CSVUpload.tsx       # CSV upload + auto-fill component
│       ├── PhotoUpload.tsx     # Photo upload component
│       ├── PageList.tsx        # Admin list view with search/sort/filter
│       └── SlugPreview.tsx     # Live slug preview as name is typed
└── middleware.ts               # Password auth protecting /admin routes
```

### Pattern 1: Pure Calculation Engine (Proven in STR Analyzer)
**What:** All financial math in a single file of pure functions. No database calls, no React, no side effects.
**When to use:** Always -- this is the most testable and reusable component.
**Example:**
```typescript
// src/lib/calculations.ts
// Source: Pattern from ~/stranalyzer/str-analyzer/src/lib/calculator.ts

export interface HomeownerInput {
  purchasePrice: number;
  purchaseDate: Date;
  currentValue: number;
  loanAmount?: number;       // optional -- defaults if not provided
  interestRate?: number;     // optional -- defaults if not provided
  loanTerm?: number;         // optional -- defaults if not provided
  monthlyRentEstimate?: number; // optional -- defaults if not provided
}

export interface HomeownerMetrics {
  // Equity & Appreciation
  equityGained: number;
  appreciationPercent: number;
  annualizedAppreciation: number;
  currentMortgageBalance: number;

  // Earnings reframing
  earningsPerMonth: number;
  earningsPerWeek: number;
  earningsPerDay: number;

  // Comparisons
  sp500Comparison: { homeReturn: number; sp500Return: number; difference: number };
  rentVsOwn: { totalRentWouldHavePaid: number; equityBuilt: number; netBenefit: number };

  // Net proceeds
  netProceeds: { gross: number; agentCommission: number; closingCosts: number; mortgagePayoff: number; net: number };

  // Future scenarios
  holdAndRent: { monthlyRent: number; monthlyMortgage: number; cashFlow: number; breakEvenDate: Date | null };
  moveUp: { netProceeds: number; downPaymentAt20Pct: number; purchasingPower: number };
  equityPlay: { availableEquity: number; helocAt80Pct: number; investmentLeverage: number };

  // Meta
  ownershipMonths: number;
  ownershipYears: number;
}

export function computeHomeownerMetrics(input: HomeownerInput): HomeownerMetrics {
  // Pure function -- all math, no side effects
}

// Reusable from STR Analyzer pattern
export function calcMortgagePayment(loan: number, annualRate: number, termYears: number): number {
  if (loan <= 0 || annualRate <= 0) return 0;
  const r = annualRate / 12;
  const n = termYears * 12;
  return loan * (r * Math.pow(1 + r, n)) / (Math.pow(1 + r, n) - 1);
}

export function calcRemainingBalance(
  loanAmount: number, annualRate: number, termYears: number, monthsElapsed: number
): number {
  // Amortization math to find current balance
}
```

### Pattern 2: Cookie-Based Password Auth (Proven in Idea Pipeline)
**What:** Simple password middleware protecting admin routes. Public routes (`/h/[slug]`) are unprotected.
**When to use:** Admin routes only.
**Example:**
```typescript
// src/middleware.ts
// Source: Pattern from ~/Projects/liveaz-idea-pipeline/src/middleware.ts

import { NextRequest, NextResponse } from "next/server";

export function middleware(request: NextRequest) {
  const session = request.cookies.get("session");

  // Public routes: /h/[slug], /login, /api/login, static assets
  if (
    request.nextUrl.pathname.startsWith("/h/") ||
    request.nextUrl.pathname.startsWith("/login") ||
    request.nextUrl.pathname.startsWith("/api/login")
  ) {
    return NextResponse.next();
  }

  // Admin routes require auth
  if (request.nextUrl.pathname.startsWith("/admin")) {
    if (session?.value !== "authenticated") {
      return NextResponse.redirect(new URL("/login", request.url));
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico).*)"],
};
```

### Pattern 3: Supabase Server Client (Proven in STR Analyzer)
**What:** Server-side Supabase client using `@supabase/ssr` with cookie handling.
**When to use:** All server-side data access (Server Components, API routes).
**Example:**
```typescript
// src/lib/supabase/server.ts
// Source: ~/stranalyzer/str-analyzer/src/lib/supabase/server.ts

import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';

export async function createClient() {
  const cookieStore = await cookies();
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return cookieStore.getAll(); },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            );
          } catch { /* Server Component context -- safe to ignore */ }
        },
      },
    }
  );
}
```

### Pattern 4: Name-Based Slug Generation
**What:** Generate URL-safe slugs from client name. Handle duplicates with auto-suffix.
**When to use:** On homeowner record creation.
**Example:**
```typescript
// src/lib/slugify.ts

export function generateSlug(clientName: string): string {
  return clientName
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9\s-]/g, '')  // Remove special chars
    .replace(/\s+/g, '-')           // Spaces to hyphens
    .replace(/-+/g, '-')            // Collapse multiple hyphens
    .replace(/^-|-$/g, '');          // Trim leading/trailing hyphens
}

// Duplicate handling: check Supabase, auto-append suffix
export async function ensureUniqueSlug(
  baseSlug: string,
  supabase: SupabaseClient
): Promise<string> {
  const { data } = await supabase
    .from('homeowners')
    .select('slug')
    .like('slug', `${baseSlug}%`);

  if (!data || data.length === 0) return baseSlug;

  // Find next available suffix
  const existingSlugs = new Set(data.map(d => d.slug));
  let suffix = 2;
  while (existingSlugs.has(`${baseSlug}-${suffix}`)) suffix++;
  return `${baseSlug}-${suffix}`;
}
```

### Anti-Patterns to Avoid
- **Storing calculation results in the database:** Calculations should be computed from raw inputs at render time. Storing computed values creates stale data and sync bugs.
- **Using Supabase Auth for single-user admin:** Overkill. Cookie-based password auth (like Idea Pipeline) is simpler and sufficient for Josh as sole admin.
- **Building the form as a single monolithic component:** Split into logical sections (property info, loan details, photos, notes) as separate components composed by the form page.
- **Parsing CSV server-side:** Parse in the browser with PapaParse for instant feedback. Only send structured data to Supabase.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| CSV parsing | Custom string splitting | PapaParse | Handles quoted fields, encoding issues, header detection, streaming for large files |
| Mortgage amortization | From-scratch math | Copy from STR Analyzer `calcMortgagePayment` + `calcAmortizationSchedule` | Already battle-tested, handles edge cases (zero rate, zero balance) |
| Slug generation | Simple string replace | Dedicated slugify function with Supabase uniqueness check | Must handle Unicode, special characters, duplicate names |
| Date math | Manual month/year calculations | date-fns `differenceInMonths`, `differenceInYears`, `format` | Leap years, timezone handling, edge cases around month boundaries |
| Form state management | useState for each field | Zustand store with `setField()` | 15+ form fields. Individual useState calls create prop-drilling and re-render issues |
| Image upload | Custom file handling | Supabase Storage with signed URLs | Handles upload, CDN delivery, resize. No custom file server needed |

**Key insight:** Phase 1 is infrastructure. Every calculation, every data access pattern, and every form pattern established here will be reused in Phases 2 and 3. Getting these foundations right prevents rewrites later.

## Common Pitfalls

### Pitfall 1: Optional Fields Without Smart Defaults Break Calculations
**What goes wrong:** Josh skips entering loan amount or rent estimate. The calculation engine receives `undefined` and either errors or produces NaN results that render as broken UI.
**Why it happens:** Optional form fields pass `undefined` to calculations when not filled.
**How to avoid:** Every optional field must have a documented smart default in a `defaults.ts` file. The calculation engine should never receive raw form data -- it receives data after defaults have been applied.
**Recommended defaults:**
- Loan amount: 80% of purchase price (conventional)
- Interest rate: 6.5% (current market approximation)
- Loan term: 30 years
- Monthly rent estimate: 0.8% of current value / 12 (rough Phoenix rental yield)
**Warning signs:** Any `NaN` or `Infinity` in calculation output.

### Pitfall 2: Slug Collisions for Common Names
**What goes wrong:** Josh creates pages for "Smith Family" and "Smith Family" (different clients). Second one silently overwrites or errors.
**Why it happens:** Name-based slugs are not inherently unique.
**How to avoid:** Database UNIQUE constraint on slug column. Application-level duplicate detection with auto-suffix (`smith-family`, `smith-family-2`). Show slug preview in the form so Josh can manually edit if needed.
**Warning signs:** Supabase unique constraint violation errors in production.

### Pitfall 3: CSV Upload Blocking Form Submission
**What goes wrong:** ARMLS CSV format changes slightly (new column name, different date format), the parser fails, and Josh cannot create a page at all.
**Why it happens:** Brittle CSV parsing that expects exact column names/positions.
**How to avoid:** CSV parsing should be best-effort: map known ARMLS columns to form fields, ignore unknown columns, use fuzzy column name matching (e.g., "List Price" OR "ListPrice" OR "list_price"). Always allow manual entry as fallback. CSV errors should show a warning, not block form submission.
**Warning signs:** CSV upload silently produces wrong values or blocks the form.

### Pitfall 4: Photo Upload Blocking the 5-Minute Workflow
**What goes wrong:** Photo upload to Supabase Storage takes 10+ seconds per image, or fails silently, causing Josh to re-upload and waste time.
**Why it happens:** Large phone photos (5-10MB), no client-side compression, no progress indicator.
**How to avoid:** Compress images client-side before upload (target 500KB). Show upload progress. Make photo upload optional and non-blocking -- Josh can save the record and add photos later.
**Warning signs:** Form submission takes more than 3 seconds.

### Pitfall 5: S&P 500 Comparison Using Wrong Data
**What goes wrong:** S&P comparison shows the home outperforming the S&P by 40%, which seems too good. Josh loses credibility.
**Why it happens:** Using point-in-time S&P values without accounting for dividends (total return), or using wrong date ranges.
**How to avoid:** Use average annual total return (approximately 10-11% historically including dividends). For the comparison, calculate what a lump sum equal to the down payment would have grown to over the same period. Be transparent about assumptions.
**Warning signs:** Home consistently "beats" S&P by unrealistic margins.

## Code Examples

### Calculation Engine: Equity and Appreciation
```typescript
// src/lib/calculations.ts

import { differenceInMonths } from 'date-fns';

const DEFAULTS = {
  loanToValue: 0.80,
  interestRate: 0.065,
  loanTermYears: 30,
  rentYieldMonthly: 0.008 / 12, // 0.8% annual of value / 12
  agentCommissionRate: 0.05,
  closingCostRate: 0.02,
  sp500AnnualReturn: 0.105, // 10.5% historical average total return
  helocLtvMax: 0.80,
  appreciationRate: 0.04, // 4% annual for projections
};

export function applyDefaults(input: Partial<HomeownerInput>): HomeownerInput {
  const purchasePrice = input.purchasePrice!;
  return {
    purchasePrice,
    purchaseDate: input.purchaseDate!,
    currentValue: input.currentValue!,
    loanAmount: input.loanAmount ?? purchasePrice * DEFAULTS.loanToValue,
    interestRate: input.interestRate ?? DEFAULTS.interestRate,
    loanTerm: input.loanTerm ?? DEFAULTS.loanTermYears,
    monthlyRentEstimate: input.monthlyRentEstimate ??
      Math.round(input.currentValue! * DEFAULTS.rentYieldMonthly),
  };
}

export function calcEquity(currentValue: number, mortgageBalance: number): number {
  return currentValue - mortgageBalance;
}

export function calcAppreciationPercent(purchasePrice: number, currentValue: number): number {
  if (purchasePrice <= 0) return 0;
  return (currentValue - purchasePrice) / purchasePrice;
}

export function calcAnnualizedAppreciation(
  purchasePrice: number, currentValue: number, months: number
): number {
  if (months <= 0 || purchasePrice <= 0) return 0;
  const years = months / 12;
  return Math.pow(currentValue / purchasePrice, 1 / years) - 1;
}

export function calcEarningsPerPeriod(equityGained: number, months: number) {
  const perMonth = months > 0 ? equityGained / months : 0;
  return {
    perMonth,
    perWeek: perMonth / 4.33,
    perDay: perMonth / 30.44,
  };
}
```

### Supabase Schema
```sql
-- src/lib/supabase/schema.sql

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE homeowners (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at      timestamptz DEFAULT now(),
  updated_at      timestamptz DEFAULT now(),

  -- Identity & URL
  slug            text UNIQUE NOT NULL,
  status          text DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),

  -- Client info
  client_name     text NOT NULL,
  personal_note   text,

  -- Property info
  address         text NOT NULL,
  city            text DEFAULT 'Phoenix',
  state           text DEFAULT 'AZ',
  zip             text,

  -- Core financials (required)
  purchase_price  numeric NOT NULL,
  purchase_date   date NOT NULL,
  current_value   numeric NOT NULL,

  -- Loan details (optional -- smart defaults applied in calculation engine)
  loan_amount     numeric,
  interest_rate   numeric,
  loan_term       int,

  -- Rental (optional)
  monthly_rent_estimate numeric,

  -- Photos (Supabase Storage paths)
  homeowner_photo_url text,
  home_photo_url      text,

  -- Metadata
  data_as_of      date DEFAULT CURRENT_DATE,
  notes           text
);

-- Auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER homeowners_updated_at
  BEFORE UPDATE ON homeowners
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

-- Row Level Security
ALTER TABLE homeowners ENABLE ROW LEVEL SECURITY;

-- Public can read published pages by slug (for /h/[slug])
CREATE POLICY "Public can view published homeowners"
  ON homeowners FOR SELECT
  USING (status = 'published');

-- Admin operations use service role key (bypasses RLS)
-- Or: use anon key with a simple policy checking a custom claim

-- Indexes
CREATE INDEX idx_homeowners_slug ON homeowners (slug);
CREATE INDEX idx_homeowners_status ON homeowners (status);
CREATE INDEX idx_homeowners_client_name ON homeowners (client_name);
CREATE INDEX idx_homeowners_created_at ON homeowners (created_at DESC);
```

### ARMLS CSV Parsing
```typescript
// src/lib/csv-parser.ts
import Papa from 'papaparse';

interface ARMLSRecord {
  address?: string;
  city?: string;
  zip?: string;
  listPrice?: number;
  soldPrice?: number;
  beds?: number;
  baths?: number;
  sqft?: number;
  yearBuilt?: number;
}

// Fuzzy column mapping -- handles ARMLS format variations
const COLUMN_MAP: Record<string, keyof ARMLSRecord> = {
  'address': 'address',
  'full address': 'address',
  'street address': 'address',
  'list price': 'listPrice',
  'listprice': 'listPrice',
  'current price': 'listPrice',
  'sold price': 'soldPrice',
  'close price': 'soldPrice',
  'city': 'city',
  'zip': 'zip',
  'zip code': 'zip',
  'postal code': 'zip',
};

export function parseARMLSCSV(file: File): Promise<ARMLSRecord[]> {
  return new Promise((resolve, reject) => {
    Papa.parse(file, {
      header: true,
      skipEmptyLines: true,
      transformHeader: (header: string) => header.trim().toLowerCase(),
      complete: (results) => {
        const records = results.data.map((row: Record<string, string>) => {
          const record: Partial<ARMLSRecord> = {};
          for (const [csvCol, value] of Object.entries(row)) {
            const mappedKey = COLUMN_MAP[csvCol.toLowerCase()];
            if (mappedKey && value) {
              const numericFields = ['listPrice', 'soldPrice', 'beds', 'baths', 'sqft', 'yearBuilt'];
              (record as any)[mappedKey] = numericFields.includes(mappedKey)
                ? parseFloat(value.replace(/[,$]/g, ''))
                : value.trim();
            }
          }
          return record as ARMLSRecord;
        });
        resolve(records);
      },
      error: (error) => reject(error),
    });
  });
}
```

### Admin Form with Zustand
```typescript
// src/lib/store.ts
import { create } from 'zustand';

interface AdminFormState {
  // Required fields
  clientName: string;
  address: string;
  city: string;
  state: string;
  zip: string;
  purchasePrice: number;
  purchaseDate: string;
  currentValue: number;

  // Optional fields
  loanAmount: number | null;
  interestRate: number | null;
  loanTerm: number | null;
  monthlyRentEstimate: number | null;
  personalNote: string;

  // Photos
  homeownerPhotoFile: File | null;
  homePhotoFile: File | null;

  // Slug
  slug: string;
  slugEdited: boolean; // Track if user manually edited slug

  // Actions
  setField: <K extends keyof AdminFormState>(key: K, value: AdminFormState[K]) => void;
  setFromCSV: (data: Partial<AdminFormState>) => void;
  reset: () => void;
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `tailwind.config.js` | CSS-first config (`@theme` in CSS) | Tailwind CSS v4 (Jan 2025) | No JS config file. Brand colors defined as CSS custom properties in `app.css` |
| `framer-motion` package | `motion` package | Late 2024 rebrand | Same API, new package name. Use `motion` for new projects |
| Next.js `getStaticProps` + `getServerSideProps` | Server Components + `generateStaticParams` | Next.js 13+ App Router | Server Components fetch data directly. No prop-passing boilerplate |
| `@supabase/auth-helpers-nextjs` | `@supabase/ssr` | Mid 2024 | New package name, simpler cookie API |
| Vitest 1.x/2.x | Vitest 4.x | 2025 | Faster, better ESM support, improved Next.js compatibility |

**Deprecated/outdated:**
- `framer-motion` package: Use `motion` instead
- `@supabase/auth-helpers-nextjs`: Use `@supabase/ssr` instead
- `tailwind.config.js`: Use CSS-first config in Tailwind v4

## Open Questions

1. **Supabase project: new or existing?**
   - What we know: STR Analyzer uses a Supabase project already. New tables could go there or in a new project.
   - What's unclear: Whether to share the Supabase project or create a separate one
   - Recommendation: Create a NEW Supabase project for isolation. The homeowner data is a separate product. Keeps the free tier usage separate and avoids accidental data leaks between products.

2. **ARMLS CSV exact format**
   - What we know: The CMA tool's `parse_armls_plano()` handles ARMLS PDFs in Python. CSV format may differ.
   - What's unclear: Exact column names in the ARMLS CSV export Josh typically uses
   - Recommendation: Build flexible column mapping (fuzzy matching). Have Josh provide a sample CSV during development to validate the parser against real data.

3. **Photo storage limits on Supabase free tier**
   - What we know: Supabase free tier includes 1GB storage. At 200 homeowners with 2 photos each at ~500KB compressed, that is roughly 200MB.
   - What's unclear: Whether Josh will use higher-resolution photos that need more space
   - Recommendation: Compress client-side to 500KB max. 1GB is sufficient for v1. Monitor usage.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.x |
| Config file | `vitest.config.ts` (create in Wave 0, pattern from Idea Pipeline) |
| Quick run command | `npx vitest run src/lib/calculations.test.ts` |
| Full suite command | `npx vitest run` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| PROD-01 | Admin form saves homeowner record to Supabase | integration | Manual -- requires Supabase connection | Wave 0 |
| PROD-01 | CSV upload auto-fills form fields | unit | `npx vitest run src/lib/csv-parser.test.ts -t "auto-fill"` | Wave 0 |
| PROD-01 | Form completes in under 5 minutes | manual-only | Timed test by Josh | N/A |
| PROD-02 | Slug generation produces URL-safe strings | unit | `npx vitest run src/lib/slugify.test.ts` | Wave 0 |
| PROD-02 | Duplicate slugs get auto-suffix | unit | `npx vitest run src/lib/slugify.test.ts -t "duplicate"` | Wave 0 |
| -- | Equity calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "equity"` | Wave 0 |
| -- | Appreciation calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "appreciation"` | Wave 0 |
| -- | Net proceeds calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "net proceeds"` | Wave 0 |
| -- | Earnings-per-period calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "earnings"` | Wave 0 |
| -- | S&P comparison calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "sp500"` | Wave 0 |
| -- | Rent-vs-own calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "rent"` | Wave 0 |
| -- | Hold & Rent scenario correct | unit | `npx vitest run src/lib/calculations.test.ts -t "hold"` | Wave 0 |
| -- | Move-Up scenario correct | unit | `npx vitest run src/lib/calculations.test.ts -t "move-up"` | Wave 0 |
| -- | Equity Play scenario correct | unit | `npx vitest run src/lib/calculations.test.ts -t "equity play"` | Wave 0 |
| -- | Smart defaults applied when fields omitted | unit | `npx vitest run src/lib/calculations.test.ts -t "defaults"` | Wave 0 |
| -- | Mortgage balance calculation correct | unit | `npx vitest run src/lib/calculations.test.ts -t "mortgage"` | Wave 0 |

### Sampling Rate
- **Per task commit:** `npx vitest run`
- **Per wave merge:** `npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `vitest.config.ts` -- Vitest configuration (pattern from Idea Pipeline's config)
- [ ] `src/lib/calculations.test.ts` -- covers all calculation functions
- [ ] `src/lib/slugify.test.ts` -- covers slug generation and duplicate handling
- [ ] `src/lib/csv-parser.test.ts` -- covers ARMLS CSV parsing
- [ ] Framework install: `npm install -D vitest @vitejs/plugin-react`

## Sources

### Primary (HIGH confidence)
- STR Analyzer `package.json` -- exact library versions Josh uses (local file)
- STR Analyzer `calculator.ts` -- proven pure calculation pattern (local file)
- STR Analyzer `supabase/server.ts` + `client.ts` -- Supabase SSR pattern (local file)
- STR Analyzer `supabase/schema.sql` -- RLS and schema pattern (local file)
- Idea Pipeline `middleware.ts` -- cookie-based password auth pattern (local file)
- Idea Pipeline `vitest.config.ts` -- Vitest configuration pattern (local file)

### Secondary (MEDIUM confidence)
- Project-level STACK.md research -- stack decisions and version compatibility
- Project-level PITFALLS.md research -- domain-specific pitfalls
- Project-level ARCHITECTURE.md research -- data flow and component boundaries
- Project-level FEATURES.md research -- feature priorities and anti-features

### Tertiary (LOW confidence)
- ARMLS CSV format assumptions -- need real sample file to validate parser column mapping
- Phoenix rental yield estimate (0.8% annual) -- approximate, should be verified against local market data

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- exact versions from existing projects, proven patterns
- Architecture: HIGH -- follows established patterns from STR Analyzer and Idea Pipeline
- Calculation engine: HIGH -- math is straightforward, pattern proven in STR Analyzer
- CSV parsing: MEDIUM -- ARMLS format not yet validated against real sample
- Pitfalls: HIGH -- domain research from project-level pitfall analysis

**Research date:** 2026-03-11
**Valid until:** 2026-04-11 (stable -- no fast-moving dependencies)
