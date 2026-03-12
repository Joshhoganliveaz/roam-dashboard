# Phase 1: Foundation + Admin - Context

**Gathered:** 2026-03-11
**Status:** Ready for planning

<domain>
## Phase Boundary

Project scaffolding, Supabase schema, calculation engine, admin input form with ARMLS CSV upload, and slug-based URL system. Josh can enter homeowner data (manually or via CSV) and the system computes all financial metrics correctly — ready for page rendering in Phase 2.

</domain>

<decisions>
## Implementation Decisions

### Admin Form Fields
- Core fields: address, client name, purchase price, purchase date, current value
- Loan details: original loan amount, interest rate, loan term (optional with smart defaults — Claude decides best approach for speed vs accuracy)
- Monthly rent estimate (optional with smart defaults)
- Photo upload: support both homeowner photo and home photo (either or both)
- Personal note: free-text field for a personal message or context
- ARMLS CSV/PDF upload that auto-fills property data and value — all fields remain editable after auto-fill

### Slug & URL Design
- Name-based slugs: `/h/smith-family` — personal, memorable, easy to text
- Duplicate handling: Claude's discretion (auto-suffix or editable slug)
- Domain: figure out later — build domain-agnostic, point it when ready

### Value Sourcing
- Primary source: ARMLS listing PDF/CSV upload that auto-fills current value and property details
- Fallback: manual entry for when CSV isn't handy
- All CSV-populated fields are editable before saving — Josh always has final say on the number
- Reuses parsing patterns from existing CMA tool (pdfplumber, ARMLS format)

### Page Management
- One URL per homeowner, forever — update existing pages rather than creating snapshots
- Updates add new chapters to the story over time (Year 1 is short, Year 5 is rich)
- "Last updated" date visible on the page so homeowners know data is fresh
- Page lifecycle states: Claude's discretion (draft/published or always-live)
- Search/sort/filter for 200+ pages: Claude's discretion on best approach

### Claude's Discretion
- Whether loan details and rent estimate are required or optional with smart defaults
- Duplicate slug handling strategy
- Page lifecycle states (draft → published vs always-live)
- Admin list view design (search, sort, filter approach)
- Exact form layout and field grouping

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `~/stranalyzer/str-analyzer/src/lib/calculator.ts`: Mortgage payment calculation (`calcMortgagePayment`), amortization schedule (`calcAmortizationSchedule`), IRR calculation — directly reusable for equity, mortgage balance, and scenario projections
- `~/stranalyzer/str-analyzer/src/lib/types.ts`: PropertyData interface with slug, status, address fields — pattern to follow for homeowner schema
- `~/stranalyzer/str-analyzer/src/lib/supabase/`: Supabase client setup with `@supabase/ssr` — reuse connection pattern
- `~/LiveAZCo-CMA/app.py`: ARMLS PDF parsing via `parse_armls_plano()` — Python, but logic can inform TypeScript parser
- `~/Projects/liveaz-idea-pipeline/src/middleware.ts`: Password-based auth middleware — reusable pattern for admin protection

### Established Patterns
- STR Analyzer: Pure calculation functions (no side effects) → unit testable. Follow this for all financial calcs
- STR Analyzer: Supabase with RLS + slug-based dynamic routes (`/analysis/[slug]`) — same pattern needed here
- Idea Pipeline: Cookie-based password auth protecting admin routes — reuse for admin form

### Integration Points
- New standalone Next.js project (greenfield) — no integration with existing apps needed
- Supabase: new project or new tables in existing Supabase instance (Claude decides)
- Vercel deployment with ISR for public pages

</code_context>

<specifics>
## Specific Ideas

- Josh wants CSV upload from day one to speed up page creation — this is a production workflow requirement, not a v2 nice-to-have
- Pages are living documents that get updated over time, not disposable snapshots — the story should grow richer with each update
- The same URL gets re-sent to homeowners at anniversaries, market shifts, or whenever Josh feels there's value to share

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-foundation-admin*
*Context gathered: 2026-03-11*
