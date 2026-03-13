---
phase: 5
slug: data-foundation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-13
---

# Phase 5 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.0.18 + jsdom |
| **Config file** | `vitest.config.ts` (exists) |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Estimated runtime** | ~10 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 05-01-01 | 01 | 1 | VALU-01 | manual-only | N/A (admin form UI) | N/A | ⬜ pending |
| 05-01-02 | 01 | 1 | VALU-02 | unit | `npx vitest run src/lib/calculations.test.ts` | Extend existing | ⬜ pending |
| 05-01-03 | 01 | 1 | VALU-03 | unit | `npx vitest run src/lib/calculations.test.ts` | Extend existing | ⬜ pending |
| 05-01-04 | 01 | 1 | VALU-04 | manual-only | N/A (visual animation) | N/A | ⬜ pending |
| 05-02-01 | 02 | 2 | CROM-01 | manual-only | N/A (form UI) | N/A | ⬜ pending |
| 05-02-02 | 02 | 2 | CROM-02 | unit | `npx vitest run src/lib/cromford.test.ts` | ❌ W0 | ⬜ pending |
| 05-02-03 | 02 | 2 | CROM-03 | manual-only | N/A (Recharts visual) | N/A | ⬜ pending |
| 05-02-04 | 02 | 2 | CROM-04 | unit | `npx vitest run src/lib/cromford.test.ts` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/lib/cromford.test.ts` — stubs for CROM-02 (data extraction) and CROM-04 (curve shifting)
- [ ] Extend `src/lib/calculations.test.ts` — stubs for VALU-02 (condition value mapping) and VALU-03 (pre-computed variants)
- [ ] `src/lib/hooks/useCounterRoll.test.ts` — covers counter-roll animation logic (unit testable portion)

*Existing vitest infrastructure covers framework install.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Admin form accepts low/high comp values | VALU-01 | Admin UI interaction | Enter low/high values in admin form, verify save |
| Cascading animation on condition change | VALU-04 | Visual animation | Change condition, observe counter-roll animations |
| Cromford upload in admin form | CROM-01 | File upload UI | Upload screenshot, verify preview renders |
| Chart renders with Cromford data | CROM-03 | Recharts visual output | View homeowner page, verify chart shows data points |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 10s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
