---
phase: 6
slug: investment-comparison-narrative-cleanup
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-13
---

# Phase 6 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.0.18 |
| **Config file** | `vitest.config.ts` (exists, jsdom environment) |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 5 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 06-01-01 | 01 | 1 | INVS-02 | unit | `npx vitest run src/lib/calculations.test.ts -t "calcLeverageROI"` | ❌ W0 | ⬜ pending |
| 06-01-02 | 01 | 1 | INVS-04 | unit | `npx vitest run src/lib/calculations.test.ts -t "calcRateLockSavings"` | ❌ W0 | ⬜ pending |
| 06-01-03 | 01 | 1 | INVS-04 | unit | `npx vitest run src/lib/calculations.test.ts -t "rateLock"` | ❌ W0 | ⬜ pending |
| 06-02-01 | 02 | 2 | INVS-01 | smoke | Visual check in browser | N/A | ⬜ pending |
| 06-02-02 | 02 | 2 | INVS-03 | unit | `npx vitest run src/lib/calculations.test.ts -t "calcLeverageROI"` | ❌ W0 | ⬜ pending |
| 06-03-01 | 03 | 2 | SCEN-11 | unit | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx` | ✅ (needs update) | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/lib/calculations.test.ts` — add test blocks for `calcLeverageROI` and `calcRateLockSavings` (file exists, needs new tests)
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` — update to verify cliffQuestion removal (file exists, needs update)
- [ ] FRED API key — register at https://fred.stlouisfed.org/docs/api/api_key.html and add to `.env.local`

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| InvestmentComparison renders correctly, SP500Callout/RentVsOwn removed | INVS-01 | Visual layout verification | Load homeowner page, confirm single Investment Comparison section replaces old sections |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 5s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
