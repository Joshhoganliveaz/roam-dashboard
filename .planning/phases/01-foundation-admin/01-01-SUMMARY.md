---
phase: 01-foundation-admin
plan: 01
subsystem: infra
tags: [nextjs, supabase, tailwind, typescript, auth, slug, vitest]

# Dependency graph
requires: []
provides:
  - "Next.js 16 project scaffold with Tailwind 4 and Live AZ Co branding"
  - "TypeScript interfaces: HomeownerRecord, HomeownerInput, HomeownerMetrics, AdminFormState"
  - "Smart defaults for optional financial fields"
  - "Supabase browser and server clients with SSR cookie handling"
  - "schema.sql for homeowners table with RLS and indexes"
  - "Cookie-based password auth middleware protecting /admin routes"
  - "Login page and API route with 7-day session cookie"
  - "Slug generation with tested uniqueness handling"
  - "Route stubs: /admin, /login, /h/[slug]"
  - "Vitest test infrastructure"
affects: [01-02, 01-03, 02-01, 02-02]

# Tech tracking
tech-stack:
  added: [next@16.1.6, react@19.2.3, tailwindcss@4, "@supabase/supabase-js@^2.99", "@supabase/ssr@^0.8", date-fns@4, papaparse@5, vitest@4, jsdom@28, "@vitejs/plugin-react@5"]
  patterns: [supabase-ssr-cookies, cookie-password-auth, pure-function-defaults, tdd-slug-generation]

key-files:
  created:
    - src/lib/types.ts
    - src/lib/defaults.ts
    - src/lib/slugify.ts
    - src/lib/slugify.test.ts
    - src/lib/supabase/client.ts
    - src/lib/supabase/server.ts
    - src/lib/supabase/schema.sql
    - src/middleware.ts
    - src/app/login/page.tsx
    - src/app/api/login/route.ts
    - src/app/h/[slug]/page.tsx
    - src/app/admin/page.tsx
    - vitest.config.ts
    - .env.local.example
  modified:
    - src/app/layout.tsx
    - src/app/page.tsx
    - src/app/globals.css
    - package.json
    - .gitignore

key-decisions:
  - "Source Serif 4 for headings, DM Sans for body via next/font/google"
  - "Tailwind 4 CSS-first config with brand colors as CSS custom properties"
  - "Cookie-based password auth (session=authenticated) with 7-day expiry"
  - "Root / redirects to /admin, protected by middleware"
  - "Supabase server client uses @supabase/ssr createServerClient with async cookies"
  - "Schema uses gen_random_uuid() for IDs, RLS allows public SELECT on published rows only"
  - "Slug generation strips non-alphanumeric chars, ensureUniqueSlug uses LIKE query with numeric suffixes"

patterns-established:
  - "Supabase SSR: createServerClient with cookie getAll/setAll from next/headers"
  - "Cookie auth: middleware checks session cookie, redirects unauthenticated to /login"
  - "Smart defaults: applyDefaults() converts nullable record fields to complete HomeownerInput"
  - "TDD: write failing tests first, then implement, for utility functions"

requirements-completed: [PROD-02]

# Metrics
duration: 7min
completed: 2026-03-12
---

# Phase 1 Plan 01: Project Scaffold Summary

**Next.js 16 project with Tailwind 4 branding, TypeScript interfaces, Supabase SSR clients, cookie auth middleware, and TDD-tested slug generation with uniqueness handling**

## Performance

- **Duration:** 7 min
- **Started:** 2026-03-12T08:04:45Z
- **Completed:** 2026-03-12T08:12:02Z
- **Tasks:** 3
- **Files modified:** 19

## Accomplishments
- Complete Next.js 16 project with Tailwind 4, Source Serif 4 + DM Sans fonts, and Live AZ Co brand color palette
- TypeScript interfaces defining the full homeowner data contract (HomeownerRecord, HomeownerInput, HomeownerMetrics, AdminFormState)
- Supabase browser and server clients with SSR cookie handling, plus full schema.sql with RLS policies
- Cookie-based password auth: middleware protects /admin, login page with branded form, API route sets httpOnly cookie
- Slug generation with 12 passing tests covering edge cases and uniqueness handling via Supabase LIKE query

## Task Commits

Each task was committed atomically:

1. **Task 1: Project scaffold, config, types, and defaults** - `b6d3728` (feat)
2. **Task 2: Supabase clients, auth middleware, login flow, and route stubs** - `5afd033` (feat)
3. **Task 3: Slug generation (TDD RED)** - `851de6d` (test)
4. **Task 3: Slug generation (TDD GREEN)** - `74cd297` (feat)

## Files Created/Modified
- `src/lib/types.ts` - HomeownerRecord, HomeownerInput, HomeownerMetrics, AdminFormState interfaces
- `src/lib/defaults.ts` - DEFAULTS constant and applyDefaults function for optional fields
- `src/lib/slugify.ts` - generateSlug (pure) and ensureUniqueSlug (async, queries Supabase)
- `src/lib/slugify.test.ts` - 12 tests for slug generation and uniqueness
- `src/lib/supabase/client.ts` - Browser Supabase client using createBrowserClient
- `src/lib/supabase/server.ts` - Server Supabase client using createServerClient with cookie handling
- `src/lib/supabase/schema.sql` - homeowners table with RLS, indexes, updated_at trigger
- `src/middleware.ts` - Password auth protecting /admin routes, allowing /h/*, /login
- `src/app/login/page.tsx` - Branded login form with error handling
- `src/app/api/login/route.ts` - POST endpoint validating SITE_PASSWORD, setting httpOnly cookie
- `src/app/h/[slug]/page.tsx` - Public route placeholder for Phase 2
- `src/app/admin/page.tsx` - Admin dashboard placeholder for Plan 03
- `src/app/layout.tsx` - Root layout with Source Serif 4 + DM Sans fonts
- `src/app/page.tsx` - Redirects to /admin
- `src/app/globals.css` - Tailwind 4 with Live AZ Co brand colors as CSS custom properties
- `vitest.config.ts` - Vitest 4 with jsdom, React plugin, path aliases
- `.env.local.example` - Documents all required environment variables
- `package.json` - All dependencies, test script
- `.gitignore` - Exception for .env.local.example

## Decisions Made
- Source Serif 4 for headings, DM Sans for body -- matches STR Analyzer convention
- Tailwind 4 CSS-first config with `@theme inline` block for brand colors
- Cookie-based password auth with `session=authenticated` value and 7-day httpOnly cookie
- Root `/` redirects to `/admin` (protected by middleware, so unauthenticated users land on login)
- Supabase schema uses `gen_random_uuid()` for IDs, RLS allows public SELECT only on published rows
- Slug generation strips all non-alphanumeric characters; `ensureUniqueSlug` uses LIKE query with auto-incrementing numeric suffix

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Installed missing jsdom dependency**
- **Found during:** Task 3 (TDD GREEN phase)
- **Issue:** Vitest configured with `environment: "jsdom"` but jsdom not installed
- **Fix:** `npm install -D jsdom`
- **Files modified:** package.json, package-lock.json
- **Verification:** All 12 tests pass
- **Committed in:** `74cd297` (Task 3 GREEN commit)

**2. [Rule 3 - Blocking] Added .env.local.example exception to .gitignore**
- **Found during:** Task 1 (commit stage)
- **Issue:** `.env*` gitignore pattern excluded `.env.local.example` from version control
- **Fix:** Added `!.env.local.example` exception to `.gitignore`
- **Files modified:** .gitignore
- **Verification:** File tracked by git
- **Committed in:** `b6d3728` (Task 1 commit)

---

**Total deviations:** 2 auto-fixed (2 blocking)
**Impact on plan:** Both fixes necessary for basic functionality. No scope creep.

## Issues Encountered
None beyond the auto-fixed deviations above.

## User Setup Required

**External services require manual configuration before the app can connect to a database.** The plan's frontmatter documents the required setup:

1. **Supabase Project:** Create a new Supabase project, then add the following to `.env.local`:
   - `NEXT_PUBLIC_SUPABASE_URL` (Project Settings > API > Project URL)
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY` (Project Settings > API > anon/public key)
   - `SUPABASE_SERVICE_ROLE_KEY` (Project Settings > API > service_role key)
2. **Database Schema:** Run `src/lib/supabase/schema.sql` in Supabase Dashboard > SQL Editor
3. **Storage Bucket:** Create a `photos` bucket with public access in Supabase Dashboard > Storage
4. **Admin Password:** Set `SITE_PASSWORD` in `.env.local` to any password

See `.env.local.example` for the complete template.

## Next Phase Readiness
- All TypeScript interfaces defined and ready for the calculation engine (Plan 02)
- Smart defaults established for optional fields
- Supabase clients ready for database operations once project is configured
- Auth middleware in place -- admin form (Plan 03) will be protected automatically
- Slug generation tested and ready for use in admin form
- Vitest infrastructure ready for calculation engine tests

## Self-Check: PASSED

All 14 files verified present. All 4 commit hashes verified in git log.

---
*Phase: 01-foundation-admin*
*Completed: 2026-03-12*
