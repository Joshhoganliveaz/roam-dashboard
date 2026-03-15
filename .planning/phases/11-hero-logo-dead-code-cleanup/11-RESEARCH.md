# Phase 11: Hero Logo Fix & Dead Code Cleanup - Research

**Researched:** 2026-03-15
**Domain:** Next.js component refactoring, asset management, dead code removal
**Confidence:** HIGH

## Summary

Phase 11 is a focused cleanup phase with two objectives: (1) replace the text-based "LIVE AZ CO" logo in HeroSection.tsx with the existing `public/logo-white.png` PNG asset on both desktop and mobile breakpoints (closing HERO-06), and (2) delete 4 dead code component files that are no longer imported anywhere.

The logo asset already exists at `public/logo-white.png` (200x197px RGBA PNG, ~15KB). Desktop already had an `<img>` tag for this in a previous iteration but was replaced with text in a later refactor. The current HeroSection.tsx uses text `<p>` elements with text-shadow for both desktop and mobile logo positions. The fix requires replacing these `<p>` elements with `<img>` (or Next.js `<Image>`) tags pointing to `/logo-white.png`.

The 4 dead code files (SP500Callout.tsx, RentVsOwn.tsx, ComboScenario.tsx, NetProceeds.tsx) total 331 lines. None are imported by any other file. The calculation functions they previously consumed (calcNetProceeds, calcRentVsOwn, calcComboScenarioProjections) remain in calculations.ts and are still used by other active components -- those must NOT be deleted.

**Primary recommendation:** Replace both desktop and mobile logo text with `<img>` tags using `/logo-white.png`, update the HeroSection test to assert an `<img>` tag instead of text, then delete the 4 dead component files and verify all 245 tests still pass.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| HERO-06 | Live AZ Co white logo renders in hero section (resized PNG from brand assets) | Logo asset exists at public/logo-white.png (200x197px). HeroSection.tsx lines 64-66 (desktop) and 115-117 (mobile) currently render text "LIVE AZ CO" -- replace with img tags. Test in HeroSection.test.tsx line 67-71 checks for text "LIVE AZ CO" -- update to check for img element. |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router framework | Project stack |
| React | 19.2.3 | UI rendering | Project stack |
| TypeScript | ^5 | Type safety | Project stack |
| Tailwind CSS | 4 | Styling | Project stack |
| Vitest | ^4.0.18 | Testing | Project stack |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| next/image | (bundled) | Optimized images | Could use for logo, but plain img is fine for 15KB asset per existing project convention |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Plain `<img>` tag | `next/image` Image component | next/image adds optimization overhead for a tiny 15KB logo; project already uses eslint-disable for this pattern in HeroSection |

**Installation:**
No new dependencies needed.

## Architecture Patterns

### Current HeroSection Structure
```
HeroSection.tsx
├── Desktop layout (hidden md:block)
│   ├── Logo text <p> (line 64)     ← REPLACE with <img>
│   └── Bottom frosted panel
├── Mobile layout (md:hidden)
│   ├── Logo text <p> (line 115)    ← REPLACE with <img>
│   └── Bottom frosted panel
└── Cream strip at bottom
```

### Pattern: Logo Image Replacement
**What:** Replace `<p>` text elements with `<img>` tags for the logo
**When to use:** Both desktop (line 64-66) and mobile (line 115-117)
**Example:**
```typescript
// BEFORE (current -- text fallback)
<p className="... text-[13px] uppercase tracking-[2px] text-[rgba(250,247,242,0.5)] [text-shadow:0_1px_4px_rgba(0,0,0,0.5)]">
  LIVE AZ CO
</p>

// AFTER (PNG logo)
{/* eslint-disable-next-line @next/next/no-img-element */}
<img
  src="/logo-white.png"
  alt="Live AZ Co"
  className="h-[28px] w-auto opacity-50"
/>
```

### Dead Code File Inventory
| File | Lines | Replaced By | Safe to Delete |
|------|-------|-------------|---------------|
| SP500Callout.tsx | 52 | InvestmentComparison.tsx | YES -- zero imports found |
| RentVsOwn.tsx | 85 | InvestmentComparison.tsx | YES -- zero imports found |
| ComboScenario.tsx | 82 | 4 new scenario cards | YES -- zero imports found |
| NetProceeds.tsx | 112 | SellAndMoveUpCard.tsx | YES -- zero imports found |

### Important: Do NOT Delete
- `calcNetProceeds` in calculations.ts -- still used by SellAndMoveUpCard.tsx and computeHomeownerMetrics
- `calcRentVsOwn` in calculations.ts -- still used by computeHomeownerMetrics
- `calcComboScenarioProjections` in calculations.ts -- still used by computeHomeownerMetrics
- `ComboScenarioProjection` type in types.ts -- still referenced by calculations
- Related tests in calculations.test.ts -- still testing active calculation functions

### Anti-Patterns to Avoid
- **Deleting calculation functions along with components:** The component files are dead code, but the underlying calculation functions are still actively used by other parts of the app
- **Using next/image for the logo:** The project has an established pattern of using plain `<img>` with eslint-disable for this small logo asset -- stay consistent

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Logo sizing for mobile/desktop | Custom resize logic | Tailwind responsive classes (h-[Xpx]) | Simple, consistent with project patterns |

## Common Pitfalls

### Pitfall 1: Forgetting to Update the Test
**What goes wrong:** HeroSection.test.tsx (line 67-71) currently asserts `screen.getAllByText("LIVE AZ CO")` -- this will fail after replacing text with img tags.
**Why it happens:** Easy to focus on the component and forget the test.
**How to avoid:** Update the test to query for an img element with alt="Live AZ Co" instead of text content.
**Warning signs:** Test suite drops from 245 passing to 244.

### Pitfall 2: Accidentally Removing Active Calculation Functions
**What goes wrong:** Deleting ComboScenario.tsx and also removing `calcComboScenarioProjections` from calculations.ts, breaking the entire computation pipeline.
**Why it happens:** Assuming "dead code cleanup" means removing everything related to these features.
**How to avoid:** Only delete the 4 component .tsx files. Leave calculations.ts, types.ts, and calculations.test.ts completely untouched.
**Warning signs:** Import errors, test failures in calculations.test.ts.

### Pitfall 3: Logo Opacity/Visibility on Light Photos
**What goes wrong:** White logo disappears against light-colored home photos.
**Why it happens:** No contrast protection.
**How to avoid:** Apply similar styling to the original text approach: use `opacity-50` (matching the rgba 0.5 alpha) and optionally `drop-shadow` for readability. The gradient overlay already provides contrast but belt-and-suspenders is fine.

### Pitfall 4: Logo Sizing Mismatch Between Breakpoints
**What goes wrong:** Logo too big on mobile or too small on desktop.
**Why it happens:** The PNG is 200x197px -- rendered at full size it would be too large for the hero position.
**How to avoid:** Use explicit height classes: approximately `h-[28px]` to match the visual weight of the 13px uppercase text it replaces. Desktop and mobile can use the same or slightly different heights.

## Code Examples

### Logo Replacement -- Desktop
```typescript
// Source: HeroSection.tsx line 64-66 replacement
// Desktop logo -- top-left (replaces text <p>)
{/* eslint-disable-next-line @next/next/no-img-element */}
<img
  src="/logo-white.png"
  alt="Live AZ Co"
  className="hidden md:block absolute top-5 left-6 h-[28px] w-auto opacity-50 drop-shadow-[0_1px_4px_rgba(0,0,0,0.5)]"
/>
```

### Logo Replacement -- Mobile
```typescript
// Source: HeroSection.tsx line 115-117 replacement
// Mobile logo -- top-left (replaces text <p>)
{/* eslint-disable-next-line @next/next/no-img-element */}
<img
  src="/logo-white.png"
  alt="Live AZ Co"
  className="md:hidden absolute top-4 left-4 h-[24px] w-auto opacity-50 drop-shadow-[0_1px_4px_rgba(0,0,0,0.5)]"
/>
```

### Updated Test
```typescript
// Source: HeroSection.test.tsx -- updated logo test
it("renders logo image (HERO-06)", () => {
  render(<HeroSection {...baseProps} />);
  const logos = screen.getAllByAltText("Live AZ Co");
  expect(logos.length).toBeGreaterThan(0);
  expect(logos[0].tagName).toBe("IMG");
});
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Text "LIVE AZ CO" with text-shadow | PNG logo-white.png with opacity | Phase 11 (now) | HERO-06 fully satisfied |
| 4 dead component files retained | Files deleted | Phase 11 (now) | 331 lines of dead code removed |

**Context:** The text fallback was a deliberate Phase 07.1 decision for mobile, with the plan to revisit in Phase 11. Desktop had img in an earlier iteration but was also simplified to text during the final 07.1 refactor.

## Open Questions

None -- this is a straightforward refactoring with no ambiguity.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest ^4.0.18 |
| Config file | vitest.config.ts (project root) |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npm test -- --run` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npm test -- --run` |

### Phase Requirements - Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| HERO-06 | PNG logo renders in hero (both breakpoints) | unit | `cd ~/Projects/homeowner-journey-map && npx vitest run src/components/homeowner/HeroSection.test.tsx` | Yes -- needs update |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npm test -- --run`
- **Per wave merge:** Same (single test suite)
- **Phase gate:** All 245+ tests green

### Wave 0 Gaps
- [ ] `HeroSection.test.tsx` -- update "renders logo text" test to assert img element with alt="Live AZ Co" instead of text content

## Sources

### Primary (HIGH confidence)
- Direct code inspection: `src/components/homeowner/HeroSection.tsx` (169 lines, current text logo implementation)
- Direct code inspection: `src/components/homeowner/HeroSection.test.tsx` (78 lines, logo test on line 67-71)
- Direct code inspection: `public/logo-white.png` exists (200x197px RGBA PNG)
- Import grep across `src/` confirmed 4 dead component files have zero imports
- Test run: 245/245 tests passing as baseline

### Secondary (MEDIUM confidence)
- Phase 07.1 SUMMARY and VERIFICATION documents confirming HERO-06 was intentionally deferred
- v1.1 audit confirming 4 dead code files are tech debt items

### Tertiary (LOW confidence)
- None

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - direct inspection of package.json and running code
- Architecture: HIGH - read every relevant source file
- Pitfalls: HIGH - verified imports, test assertions, and calculation dependencies directly

**Research date:** 2026-03-15
**Valid until:** 2026-04-15 (stable -- no external dependencies or API changes involved)
