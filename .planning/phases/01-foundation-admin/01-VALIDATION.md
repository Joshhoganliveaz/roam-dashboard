---
phase: 1
slug: foundation-admin
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-11
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.x |
| **Config file** | `vitest.config.ts` (create in Wave 0, pattern from Idea Pipeline) |
| **Quick run command** | `npx vitest run src/lib/calculations.test.ts` |
| **Full suite command** | `npx vitest run` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `npx vitest run`
- **After every plan wave:** Run `npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 01-01-01 | 01 | 0 | -- | setup | `npx vitest run` | ❌ W0 | ⬜ pending |
| 01-02-01 | 02 | 1 | PROD-02 | unit | `npx vitest run src/lib/slugify.test.ts` | ❌ W0 | ⬜ pending |
| 01-02-02 | 02 | 1 | PROD-02 | unit | `npx vitest run src/lib/slugify.test.ts -t "duplicate"` | ❌ W0 | ⬜ pending |
| 01-03-01 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "equity"` | ❌ W0 | ⬜ pending |
| 01-03-02 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "appreciation"` | ❌ W0 | ⬜ pending |
| 01-03-03 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "net proceeds"` | ❌ W0 | ⬜ pending |
| 01-03-04 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "earnings"` | ❌ W0 | ⬜ pending |
| 01-03-05 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "sp500"` | ❌ W0 | ⬜ pending |
| 01-03-06 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "rent"` | ❌ W0 | ⬜ pending |
| 01-03-07 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "mortgage"` | ❌ W0 | ⬜ pending |
| 01-03-08 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "hold"` | ❌ W0 | ⬜ pending |
| 01-03-09 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "move-up"` | ❌ W0 | ⬜ pending |
| 01-03-10 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "equity play"` | ❌ W0 | ⬜ pending |
| 01-03-11 | 03 | 1 | -- | unit | `npx vitest run src/lib/calculations.test.ts -t "defaults"` | ❌ W0 | ⬜ pending |
| 01-04-01 | 04 | 2 | PROD-01 | unit | `npx vitest run src/lib/csv-parser.test.ts` | ❌ W0 | ⬜ pending |
| 01-05-01 | 05 | 2 | PROD-01 | integration | Manual -- requires Supabase connection | N/A | ⬜ pending |
| 01-05-02 | 05 | 2 | PROD-01 | manual-only | Timed test by Josh (< 5 minutes) | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `vitest.config.ts` — Vitest configuration (pattern from Idea Pipeline's config)
- [ ] `src/lib/calculations.test.ts` — stubs for all calculation functions
- [ ] `src/lib/slugify.test.ts` — stubs for slug generation and duplicate handling
- [ ] `src/lib/csv-parser.test.ts` — stubs for ARMLS CSV parsing
- [ ] Framework install: `npm install -D vitest @vitejs/plugin-react`

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Form completes in under 5 minutes | PROD-01 | UX timing requires human operator | Josh times himself creating a homeowner record end-to-end |
| Admin form saves to Supabase | PROD-01 | Requires live Supabase connection | Create a test record, verify in Supabase dashboard |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
