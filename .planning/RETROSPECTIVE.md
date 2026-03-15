# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — Foundation (Completed 2026-03-12)

**Shipped:** 2026-03-12
**Phases:** 5 | **Plans:** 13

### What Was Built
- Admin form with CSV upload, photo upload, preview/publish workflow
- Calculation engine (equity, appreciation, S&P 500, rent vs own, 3 scenarios)
- Branded homeowner page at /h/[slug] with scrolling SB7 narrative
- Scroll-triggered animations, adaptive voice by ownership duration
- Edge case handling (underwater, cash, negative appreciation)

### What Worked
- TDD for calculation engine caught edge cases early
- Single-file computation module kept all math testable and reusable
- Pre-computed server-side metrics eliminated client-side complexity

### What Was Inefficient
- Phase 2.1 was an insertion for edge cases that should have been caught in Phase 2
- Documentation cleanup (Phase 4) was entirely catch-up work

### Patterns Established
- Server-side pre-computation with client-side switching
- SB7 narrative adaptation by ownership duration
- Condition-aware fallbacks for backward compatibility

### Key Lessons
1. Edge case handling should be part of the initial implementation, not a separate phase
2. Documentation-as-you-go prevents cleanup phases

## Milestone: v1.1 — Enhanced Intelligence (Completed 2026-03-15)

**Shipped:** 2026-03-15
**Phases:** 8 | **Plans:** 14

### What Was Built
- Condition-aware value range with 3-variant server pre-computation and cascading counter-roll animations
- Cromford OCR pipeline: screenshot → Claude Vision → branded Recharts chart
- Merged Investment Comparison section with leverage ROI hero number and FRED market rates
- Premium frosted-glass hero with Playfair Display gold stat and full-bleed photo
- Page flow rework: paycheck hook leads, scenarios before proof, warm CTA
- Four reworked scenario cards with date-based horizons, mini mortgage calculators, effective interest rate
- Condition picker reframed as neighbor comparison with inline comp data
- Dead code cleanup: 331 lines removed from 4 superseded components

### What Worked
- Pre-computing 3 condition variants server-side made client switching instant with zero API calls
- TDD for financial calculations (leverage ROI, rate lock, effective interest rate) caught edge cases before UI work
- Phase 07.1 insertion process worked cleanly — hero redesign was scoped tightly and shipped in one plan
- Milestone audit before completion caught HERO-06 gap and dead code, leading to Phase 10+11 cleanup
- Asymmetric clamping (-3%/+5%) was a smart SB7-informed decision that made condition selection feel rewarding

### What Was Inefficient
- Phases 10 and 11 were entirely audit-driven cleanup — verification and dead code removal that could have been done during implementation phases
- Phase 07.1 HERO-06 was marked PARTIAL and required a follow-up phase (11) to close — should have been completed in the original phase
- Some SUMMARY files lacked one_liner fields, making automated extraction return null

### Patterns Established
- FRED API market rate with DEFAULTS fallback for graceful degradation
- Bisection solver for effective interest rate (novel metric)
- Card-local state via useState for mortgage calculators — never lift to parent
- Frosted glass panels with webkit backdrop-filter prefix for premium UI feel
- eslint-disable for plain img tags on brand assets (not next/image)

### Key Lessons
1. Close cosmetic/doc gaps in the same phase they're created — follow-up phases add overhead
2. SUMMARY frontmatter should always include one_liner for automated milestone tools
3. Milestone audit is valuable but should happen earlier to avoid last-minute cleanup phases
4. Pre-computed server-side variants scale well — 3x data per page is negligible at 200+ pages

### Cost Observations
- Sessions: ~6-8 across 2 days
- Notable: 14 plans completed in 2 days with zero test regressions (245 tests passing)

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Phases | Plans | Key Change |
|-----------|--------|-------|------------|
| v1.0 | 5 | 13 | Foundation — established TDD, server pre-computation patterns |
| v1.1 | 8 | 14 | Added milestone audit, phase insertion process, FRED API integration |

### Cumulative Quality

| Milestone | Tests | Key Metric |
|-----------|-------|------------|
| v1.0 | ~100 | Zero edge case failures in production |
| v1.1 | 245 | 35/35 requirements verified across 3 sources |

### Top Lessons (Verified Across Milestones)

1. TDD for financial calculations prevents costly bugs — validated in both v1.0 (edge cases) and v1.1 (leverage/effective rate)
2. Server-side pre-computation eliminates client complexity — used for both initial metrics (v1.0) and condition variants (v1.1)
3. Documentation gaps compound — both milestones had cleanup phases that were avoidable
