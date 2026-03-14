---
phase: 7
slug: future-scenarios-rework
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-14
---

# Phase 7 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.0.18 + Testing Library React 16.3.2 |
| **Config file** | `~/Projects/homeowner-journey-map/vitest.config.ts` |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Estimated runtime** | ~10 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 07-01-01 | 01 | 1 | SCEN-04 | unit | `npx vitest run src/lib/calculations.test.ts -t "projection"` | Partial | ⬜ pending |
| 07-01-02 | 01 | 1 | SCEN-05 | unit | `npx vitest run src/lib/calculations.test.ts -t "market rate"` | ❌ W0 | ⬜ pending |
| 07-01-03 | 01 | 1 | SCEN-08 | unit | `npx vitest run src/lib/calculations.test.ts -t "holdAndRent projection"` | ❌ W0 | ⬜ pending |
| 07-01-04 | 01 | 1 | SCEN-10 | unit | `npx vitest run src/lib/calculations.test.ts -t "equityPlay"` | Partial | ⬜ pending |
| 07-01-05 | 01 | 1 | SCEN-07 | unit | `npx vitest run src/lib/calculations.test.ts -t "combo"` | ❌ W0 | ⬜ pending |
| 07-02-01 | 02 | 2 | SCEN-05 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "slider"` | ❌ W0 | ⬜ pending |
| 07-02-02 | 02 | 2 | SCEN-06 | unit+component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "rate lock"` | Partial | ⬜ pending |
| 07-02-03 | 02 | 2 | SCEN-09 | component | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "move-up"` | Partial | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] Add projection function test stubs to `src/lib/calculations.test.ts` — covers SCEN-04, SCEN-05, SCEN-08, SCEN-10
- [ ] Add combo scenario test stubs to `src/lib/calculations.test.ts` — covers SCEN-07
- [ ] Add rate lock conditional test stubs to `src/components/homeowner/ScenarioCards.test.tsx` — covers SCEN-06
- [ ] Add slider interaction test stubs to `src/components/homeowner/ScenarioCards.test.tsx` — covers SCEN-05

*Existing infrastructure covers framework install — Vitest already configured.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Slider drag performance on mobile | SCEN-05 | Perceived jank is subjective, device-dependent | Drag slider on older iPhone, confirm no visible stutter |
| SB7 narrative tone in scenario copy | SCEN-06, SCEN-09 | Content quality is subjective | Read scenario text aloud, verify homeowner-as-hero framing |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
