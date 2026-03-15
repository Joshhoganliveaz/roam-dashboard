---
phase: 11
slug: hero-logo-dead-code-cleanup
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-15
---

# Phase 11 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest ^4.0.18 |
| **Config file** | vitest.config.ts |
| **Quick run command** | `cd ~/Projects/homeowner-journey-map && npm test -- --run` |
| **Full suite command** | `cd ~/Projects/homeowner-journey-map && npm test -- --run` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd ~/Projects/homeowner-journey-map && npm test -- --run`
- **After every plan wave:** Run `cd ~/Projects/homeowner-journey-map && npm test -- --run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 11-01-01 | 01 | 1 | HERO-06 | unit | `npx vitest run src/components/homeowner/HeroSection.test.tsx` | ✅ (needs update) | ⬜ pending |
| 11-01-02 | 01 | 1 | HERO-06 | regression | `npm test -- --run` | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `HeroSection.test.tsx` — update "renders logo text" test to assert img element with alt="Live AZ Co" instead of text content

*Existing infrastructure covers all other phase requirements.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Logo visibility on light/dark photos | HERO-06 | Visual contrast depends on photo content | Load app with light and dark hero photos, confirm logo visible on both |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
