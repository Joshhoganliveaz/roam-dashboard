---
phase: 3
slug: narrative-experience
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-12
---

# Phase 3 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Vitest 4.x with jsdom |
| **Config file** | `vitest.config.ts` (exists from Phase 1) |
| **Quick run command** | `npm test` |
| **Full suite command** | `npm test` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run `npm test`
- **After every plan wave:** Run `npm test && npm run build`
- **Before `/gsd:verify-work`:** Full suite must be green + visual review of animations on mobile viewport
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 03-01-01 | 01 | 0 | NARR-03 | unit (hook) | `npx vitest run src/lib/hooks/useCountUp.test.ts` | No -- Wave 0 | ⬜ pending |
| 03-01-02 | 01 | 0 | NARR-04 | unit | `npx vitest run src/lib/narrative.test.ts` | No -- Wave 0 | ⬜ pending |
| 03-01-03 | 01 | 0 | NARR-05 | unit | `npx vitest run src/lib/milestones.test.ts` | No -- Wave 0 | ⬜ pending |
| 03-01-04 | 01 | 0 | NARR-08 | smoke (render) | `npx vitest run src/components/homeowner/CTASection.test.tsx` | No -- Wave 0 | ⬜ pending |
| 03-01-05 | 01 | 0 | SCEN-01 | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "hold"` | No -- Wave 0 | ⬜ pending |
| 03-01-06 | 01 | 0 | SCEN-02 | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "move"` | No -- Wave 0 | ⬜ pending |
| 03-01-07 | 01 | 0 | SCEN-03 | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "equity"` | No -- Wave 0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `src/lib/narrative.test.ts` — voice selection boundary tests (23mo, 24mo, 83mo, 84mo) covers NARR-04
- [ ] `src/lib/milestones.test.ts` — milestone selection and ranking logic covers NARR-05
- [ ] `src/lib/hooks/useCountUp.test.ts` — counter easing and final value accuracy covers NARR-03
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` — render tests for all 3 cards covers SCEN-01/02/03
- [ ] `src/components/homeowner/CTASection.test.tsx` — render test for sms link covers NARR-08
- [ ] Motion install: `npm install motion` — not yet in dependencies

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Scroll animation feel (counters count up, charts draw, sections reveal) | NARR-03 | Visual animation timing/easing requires human judgment | Scroll through page on desktop and mobile; verify animations trigger once, feel smooth, no jank |
| Narrative tone matches ownership duration | NARR-04 | Copy quality and emotional tone are subjective | Compare recent buyer (<2yr) vs long-term owner (5+yr) pages; verify language shift feels natural |
| CTA feels like natural narrative conclusion | NARR-08 | "Not a sales pitch" is a subjective quality gate | Read full page top-to-bottom; CTA should feel like a conversation closer, not an ad |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
