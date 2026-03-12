---
phase: 2
slug: core-homeowner-page
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-12
---

# Phase 2 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.x with jsdom |
| **Config file** | `vitest.config.ts` (exists) |
| **Quick run command** | `npm test` |
| **Full suite command** | `npm test` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `npm test`
- **After every plan wave:** Run `npm test && npm run build`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 02-01-01 | 01 | 1 | DATA-01 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "current value"` | ❌ W0 | ⬜ pending |
| 02-01-02 | 01 | 1 | DATA-02 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "equity"` | ❌ W0 | ⬜ pending |
| 02-01-03 | 01 | 1 | DATA-03 | smoke (render) | `npx vitest run src/components/homeowner/AppreciationChart.test.tsx` | ❌ W0 | ⬜ pending |
| 02-01-04 | 01 | 1 | DATA-04 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "net proceeds"` | ❌ W0 | ⬜ pending |
| 02-01-05 | 01 | 1 | DATA-05 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "earnings"` | ❌ W0 | ⬜ pending |
| 02-01-06 | 01 | 1 | DATA-06 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "S&P"` | ❌ W0 | ⬜ pending |
| 02-01-07 | 01 | 1 | DATA-07 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "rent"` | ❌ W0 | ⬜ pending |
| 02-01-08 | 01 | 1 | NARR-01 | smoke (render) | `npx vitest run src/components/homeowner/HomeownerPage.test.tsx -t "sections"` | ❌ W0 | ⬜ pending |
| 02-01-09 | 01 | 1 | NARR-02 | manual | Visual review | N/A | ⬜ pending |
| 02-01-10 | 01 | 1 | NARR-06 | manual | Visual review | N/A | ⬜ pending |
| 02-01-11 | 01 | 1 | NARR-07 | manual | Browser dev tools responsive mode | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/components/homeowner/HomeownerPage.test.tsx` — render tests covering DATA-01 through DATA-07, NARR-01
- [ ] `src/components/homeowner/AppreciationChart.test.tsx` — chart data generation tests for DATA-03
- [ ] `src/lib/format.test.ts` — currency/percent formatter tests
- [ ] Recharts install: `npm install recharts` — not yet in dependencies

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| SB7 framing (narrative before numbers) | NARR-02 | Subjective narrative quality | Review each section has intro text before data display |
| Brand colors and fonts | NARR-06 | Visual design validation | Verify olive/canyon/gold/cream/charcoal palette; Source Serif 4 headings; DM Sans body |
| Mobile-first responsive | NARR-07 | Layout/viewport testing | Test in Chrome DevTools at 375px, 390px, 428px widths |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
