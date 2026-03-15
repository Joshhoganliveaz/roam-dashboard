# Phase 10: Verification & Tracking Closure - Research

**Researched:** 2026-03-14
**Domain:** Documentation gap closure, requirement tracking, project artifacts
**Confidence:** HIGH

## Summary

Phase 10 is a purely documentation-focused phase. No code changes are required. The v1.1 milestone audit (v1.1-MILESTONE-AUDIT.md) identified specific gaps: three phases (05, 07.1, 08) are missing VERIFICATION.md files, two requirements (COMP-01, COMP-04) are implemented in code but not tracked in SUMMARY frontmatter or checked in REQUIREMENTS.md, and the ROADMAP.md progress table is stale.

The two requirements assigned to this phase (COMP-01, COMP-04) are documentation gaps only -- the code implementing them already exists and was confirmed by the integration checker. The work is to update tracking artifacts so all three documentation sources (VERIFICATION.md, SUMMARY frontmatter, REQUIREMENTS.md) agree.

**Primary recommendation:** This phase should be a single plan with three tasks: (1) create missing VERIFICATION.md files for phases 05/07.1/08, (2) fix COMP-01/COMP-04 tracking in 08-01-SUMMARY.md and REQUIREMENTS.md, (3) update ROADMAP.md progress table to reflect actual state.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| COMP-01 | Condition picker reframed as "How does your home compare to your neighbors?" | Documentation gap only -- ConditionPicker.tsx already uses neighbor comparison framing. Fix: add COMP-01 to 08-01-SUMMARY.md requirements_completed, check [x] in REQUIREMENTS.md |
| COMP-04 | Comp display kept minimal and clean -- informs selection without visual clutter | Documentation gap only -- InlineComps renders clean table. Fix: add COMP-04 to 08-01-SUMMARY.md requirements_completed, check [x] in REQUIREMENTS.md |
</phase_requirements>

## Standard Stack

No libraries or code dependencies. This phase uses only markdown files.

### Tools Required
| Tool | Purpose | Why |
|------|---------|-----|
| Text editor / Write tool | Create and edit .md files | All work is markdown artifact creation/editing |
| Git | Commit documentation updates | Track changes to planning artifacts |

## Architecture Patterns

### Documentation Artifact Structure

The project uses a three-source verification model. All three must agree for a requirement to be "fully satisfied":

```
Source 1: VERIFICATION.md    (per-phase, verifier creates)
Source 2: SUMMARY.md          (per-plan, frontmatter requirements_completed field)
Source 3: REQUIREMENTS.md     (project-level, checkbox [x] per requirement)
```

### VERIFICATION.md Format (from existing examples)

Every VERIFICATION.md follows this structure:
```
---
phase: {phase-slug}
verified: {ISO timestamp}
status: passed | human_needed
score: X/Y success criteria verified
must_haves:
  truths: [list of success criteria strings]
  artifacts: [list of {path, status} objects]
requirements:
  - id: REQ-ID
    status: satisfied | partial
human_verification: [list of manual test items]
---

# Phase X: Name Verification Report

## Goal Achievement
### Observable Truths (table)
### Required Artifacts (table)
### Key Link Verification (table)
### Requirements Coverage (table)
### Anti-Patterns Found (table)
### Human Verification Required (numbered sections)
### Gaps Summary
```

### SUMMARY.md Frontmatter Pattern

The `requirements_completed` field in SUMMARY frontmatter tracks which requirements a plan satisfied:
```yaml
requirements-completed: [REQ-01, REQ-02]
```

### REQUIREMENTS.md Checkbox Pattern

Each requirement has a checkbox that must be `[x]` when complete:
```markdown
- [x] **REQ-ID**: Description
```

And the Traceability table at the bottom must show correct Phase and Status.

### ROADMAP.md Progress Table Pattern

The progress table at the bottom of ROADMAP.md tracks completion:
```markdown
| Phase | Milestone | Plans Complete | Status | Completed |
```

Plan checkboxes within phase details also need updating:
```markdown
- [x] 09-01-PLAN.md -- Description
```

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Verification format | Invent new format | Copy exact structure from 01-VERIFICATION.md or 06-VERIFICATION.md | Consistency across all phases |
| Success criteria | Make them up | Pull from ROADMAP.md phase details | Already defined in roadmap |

## Common Pitfalls

### Pitfall 1: Missing the Three-Source Update
**What goes wrong:** Updating REQUIREMENTS.md but forgetting SUMMARY frontmatter (or vice versa)
**Why it happens:** The three-source model requires updates in three different files for each requirement
**How to avoid:** Checklist approach -- for each requirement, verify all three sources updated
**Warning signs:** Audit still shows gaps after "completion"

### Pitfall 2: Stale ROADMAP Checkboxes
**What goes wrong:** Phase status updated but individual plan checkboxes left unchecked
**Why it happens:** ROADMAP has both a progress table AND per-phase plan lists with checkboxes
**How to avoid:** Update both the progress table row AND each plan checkbox within the phase details section
**Warning signs:** Progress table says "Complete" but plans still show `- [ ]`

### Pitfall 3: VERIFICATION.md for Phase with Known Gaps
**What goes wrong:** Phase 07.1 has HERO-06 as a known partial -- the VERIFICATION.md must note this as partial, not satisfied
**Why it happens:** Temptation to mark everything as passed
**How to avoid:** HERO-06 should be status: partial in the 07.1 VERIFICATION.md, noting it is deferred to Phase 11
**Warning signs:** Marking HERO-06 as satisfied when text logo is used instead of PNG

### Pitfall 4: Inconsistent Traceability Table
**What goes wrong:** REQUIREMENTS.md checkbox updated but Traceability table at bottom still shows old Phase/Status
**Why it happens:** The Traceability table is a separate section from the checkboxes
**How to avoid:** Update both the checkbox AND the corresponding row in the Traceability table
**Warning signs:** `[x]` checkbox but "Pending" in traceability

## Specific Gap Inventory (from v1.1 Audit)

### Gap 1: Missing VERIFICATION.md Files

| Phase | Directory | Requirements to Verify | Success Criteria Source |
|-------|-----------|----------------------|----------------------|
| 05-data-foundation | .planning/phases/05-data-foundation/ | VALU-01..04, CROM-01..04 (8 reqs) | ROADMAP.md Phase 5 success criteria (5 items) |
| 07.1-hero-section-redesign | .planning/phases/07.1-hero-section-redesign/ | HERO-01..07 (7 reqs, HERO-06 partial) | ROADMAP.md Phase 07.1 success criteria (7 items) |
| 08-condition-picker-comp-context | .planning/phases/08-condition-picker-comp-context/ | COMP-01..04 (4 reqs) | ROADMAP.md Phase 8 success criteria (4 items) |

### Gap 2: COMP-01/COMP-04 Tracking

**Current state in 08-01-SUMMARY.md:**
```yaml
requirements-completed: [COMP-02, COMP-03]
```
**Should be:**
```yaml
requirements-completed: [COMP-01, COMP-02, COMP-03, COMP-04]
```

**Current state in REQUIREMENTS.md:**
```markdown
- [ ] **COMP-01**: Condition picker reframed...
- [ ] **COMP-04**: Comp display kept minimal...
```
**Should be:**
```markdown
- [x] **COMP-01**: Condition picker reframed...
- [x] **COMP-04**: Comp display kept minimal...
```

**Traceability table updates needed:**
```markdown
| COMP-01 | Phase 8, 10 | Pending |  -->  | COMP-01 | Phase 8 | Complete |
| COMP-04 | Phase 8, 10 | Pending |  -->  | COMP-04 | Phase 8 | Complete |
```

### Gap 3: ROADMAP.md Progress Table

**Current stale entries:**
- Phase 07.1: shows `- [ ]` for plans, needs `- [x]`
- Phase 8: shows `0/0` plans, needs `1/1`
- Phase 9: shows `- [ ]` for plan checkboxes, needs `- [x]`
- Phase 10 row: will need updating after this phase completes

## Code Examples

Not applicable -- this phase is documentation only. See the VERIFICATION.md format above and the existing examples at:
- `.planning/phases/01-foundation-admin/01-VERIFICATION.md` (full reference)
- `.planning/phases/06-investment-comparison-narrative-cleanup/06-VERIFICATION.md` (v1.1 reference)
- `.planning/phases/09-scenario-cards-rework/09-VERIFICATION.md` (most recent)

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | N/A -- documentation-only phase |
| Config file | N/A |
| Quick run command | N/A |
| Full suite command | `grep -c '\[ \]' .planning/REQUIREMENTS.md` (should be 1 -- only HERO-06) |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| COMP-01 | Checkbox and tracking updated | manual | `grep 'COMP-01' .planning/REQUIREMENTS.md` | N/A |
| COMP-04 | Checkbox and tracking updated | manual | `grep 'COMP-04' .planning/REQUIREMENTS.md` | N/A |

### Sampling Rate
- **Per task:** Verify grep for updated checkboxes
- **Phase gate:** All v1.1 requirements except HERO-06 show `[x]` in REQUIREMENTS.md; all phases 05-09 have VERIFICATION.md

### Wave 0 Gaps
None -- no test infrastructure needed for documentation phase.

## Open Questions

1. **Should VERIFICATION.md files for phases 05/07.1/08 involve re-reading source code?**
   - What we know: The integration checker in the audit already confirmed all requirements are WIRED
   - What's unclear: How thorough the verification needs to be given audit already ran
   - Recommendation: Create VERIFICATION.md files using audit findings as evidence, noting integration checker results. No need to re-inspect every file if audit already confirmed wiring.

2. **Should Phase 10 update STATE.md?**
   - What we know: STATE.md tracks current position and decisions
   - Recommendation: Yes, update current position and progress metrics after all gaps are closed.

## Sources

### Primary (HIGH confidence)
- `.planning/v1.1-MILESTONE-AUDIT.md` -- Definitive gap inventory
- `.planning/REQUIREMENTS.md` -- Current requirement status
- `.planning/ROADMAP.md` -- Current progress table state
- `.planning/phases/01-foundation-admin/01-VERIFICATION.md` -- VERIFICATION.md format reference
- `.planning/phases/08-condition-picker-comp-context/08-01-SUMMARY.md` -- Current SUMMARY state showing gap

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no code, pure documentation
- Architecture: HIGH -- format patterns well-established from 10+ existing artifacts
- Pitfalls: HIGH -- gaps explicitly enumerated in audit report

**Research date:** 2026-03-14
**Valid until:** 2026-03-21 (stable -- documentation patterns do not change)
