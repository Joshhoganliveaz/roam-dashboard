---
phase: 8
slug: condition-picker-comp-context
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-14
---

# Phase 8 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.0.18 |
| **Config file** | `vitest.config.ts` |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts src/lib/comp-analyzer.test.ts` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts src/lib/comp-analyzer.test.ts`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 08-01-01 | 01 | 1 | COMP-03 | unit | `npx vitest run src/lib/calculations.test.ts` | ❌ W0 | ⬜ pending |
| 08-01-02 | 01 | 1 | COMP-02 | unit | `npx vitest run src/lib/comp-analyzer.test.ts` | ❌ W0 | ⬜ pending |
| 08-02-01 | 02 | 2 | COMP-01 | manual | Visual inspection of ConditionPicker.tsx + CurrentValue.tsx | N/A | ⬜ pending |
| 08-02-02 | 02 | 2 | COMP-02 | manual | Visual inspection of InlineComps | N/A | ⬜ pending |
| 08-02-03 | 02 | 2 | COMP-04 | manual | Visual inspection -- minimal/clean comp display | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/lib/calculations.test.ts` — add tests for clamped getConditionValues (COMP-03)
- [ ] `src/lib/comp-analyzer.test.ts` — add tests for soldPrice in representative comps (COMP-02)
- [ ] No new framework install needed — Vitest already configured and working

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Condition picker text reframed as neighbor comparison | COMP-01 | UI text copy — no logic to test | Verify ConditionPicker.tsx labels read "Similar to most" / "Better than most" / "Best on the block" and CurrentValue.tsx heading reads "How does your home compare to your neighbors?" |
| Comp display is minimal and clean | COMP-04 | Subjective design quality | Visual inspection: comp list should be compact, no cards/borders/shadows, address + price only, no layout shift on pages without comps |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 10s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
