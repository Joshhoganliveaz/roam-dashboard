---
phase: 9
slug: scenario-cards-rework
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-13
---

# Phase 9 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.0.18 |
| **Config file** | `vitest.config.ts` (root) |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 09-01-01 | 01 | 1 | SCNR-01 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | Exists (needs update) | ⬜ pending |
| 09-01-02 | 01 | 1 | SCNR-02 | unit | `npx vitest run src/lib/calculations.test.ts -x` | ❌ W0 | ⬜ pending |
| 09-01-03 | 01 | 1 | SCNR-03 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | ❌ W0 | ⬜ pending |
| 09-02-01 | 02 | 2 | SCNR-04 | unit | `npx vitest run src/lib/calculations.test.ts -x` | ❌ W0 | ⬜ pending |
| 09-02-02 | 02 | 2 | SCNR-05 | unit + component | `npx vitest run src/lib/calculations.test.ts -x` | Partial | ⬜ pending |
| 09-02-03 | 02 | 2 | SCNR-06 | unit | `npx vitest run src/lib/calculations.test.ts -x` | ❌ W0 | ⬜ pending |
| 09-03-01 | 03 | 2 | SCNR-07 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | ❌ W0 | ⬜ pending |
| 09-03-02 | 03 | 2 | SCNR-08 | unit | `npx vitest run src/lib/calculations.test.ts -x` | ❌ W0 | ⬜ pending |
| 09-03-03 | 03 | 2 | SCNR-09 | unit | `npx vitest run src/lib/calculations.test.ts -x` | ❌ W0 | ⬜ pending |
| 09-04-01 | 04 | 3 | SCNR-10 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -x` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/lib/calculations.test.ts` — new tests for `calcEffectiveInterestRate`, date horizon formatting, 6%+1% commission split
- [ ] `src/lib/calculations.test.ts` — new tests for Stay & Build, Sell & Move Up, Stay & Invest, Move & Keep projection functions
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` — updated for new card titles (Stay & Build Wealth, Sell & Move Up, Stay & Invest, Move & Keep as Rental)
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` — new tests for contextual CTAs, HELOC rendering, market rate slider overrides
- [ ] Edge case tests: zero mortgage, negative equity, zero rent estimate, outOfPocket <= 0

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Mobile date horizon fallback to year-only | SCNR-02 | Visual breakpoint behavior | Resize browser below 640px, verify "2026 \| 2031 \| 2036 \| 2041" instead of "Mar '26 \| Mar '31 \| Mar '36 \| Mar '41" |
| MarketRateSlider drag interaction | SCNR-03 | Pointer interaction + visual | Drag slider, confirm all cards update; override in mini calc, confirm only that card changes |
| Down payment preset button UX | SCNR-05 | Visual toggle states | Click 5/10/15/20% buttons, verify selection highlight and recalculation |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
