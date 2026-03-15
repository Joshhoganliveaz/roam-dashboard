# Phase 8: Condition Picker & Comp Context - Research

**Researched:** 2026-03-14
**Domain:** UI reframe + comp data integration in existing Next.js 16 / React 19 homeowner page
**Confidence:** HIGH

## Summary

Phase 8 is a focused UI rework of the existing condition picker and a new inline comp display. The current codebase already has the full plumbing for condition-based value selection (three-option picker in `ConditionPicker.tsx`, condition values computed in `getConditionValues()`, pre-computed variant metrics served from the server component). The existing `ConditionCompLinks.tsx` component currently shows a "See a similar home" external Zillow link per condition -- this must be replaced with 2-3 inline comp cards showing address and sold price with no external links (per PAGE-04 from Phase 7 and COMP-02/COMP-04).

The comp data pipeline already exists: Josh uploads an ARMLS comp CSV in the admin form, `parseCompCSV()` extracts `CompSale[]` records (address, soldPrice, sqft, beds, baths, etc.), `deriveValueRange()` scores and selects representative comps, and `conditionCompLinks` is stored in the Supabase `homeowners` table as JSONB. The database and types already support this. What needs to change: (1) store richer comp data (not just address + zillowUrl), (2) pass comp data to the homeowner page, (3) reframe the picker question text, (4) tighten the value range formula, and (5) build a minimal inline comp display.

**Primary recommendation:** This is a contained refactor with no new libraries needed. Modify the existing `ConditionCompLink` type to carry `soldPrice` (and optionally photo URL), update `getConditionValues()` to clamp the range to +/-3-5% of currentValue, rewrite the `CurrentValue.tsx` section text from "Select your home's condition" to "How does your home compare to your neighbors?", and replace `ConditionCompLinks.tsx` with a minimal inline comp list (no external links).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Change from "Select your home's condition" (Mostly original / Made some updates / Completely remodeled) to "How does your home compare to your neighbors?"
- This taps into natural curiosity and competitiveness rather than asking them to judge their own home negatively
- Homeowners always think their home is worth more -- the range should reflect this reality
- Tighten to +/-3-5% from base value (currently the range between low comp and high comp can be too wide)
- The number shouldn't move too much regardless of selection -- it's more about engagement than precision
- Example: $500K base -> range of ~$485K to ~$515K, not $450K to $550K
- Show 2-3 nearby recent sales from the CSV data Josh already uploads
- Each comp: address, sold price, maybe one photo (if available from MLS data)
- Minimal styling -- small cards or a simple list
- NO external links -- everything stays on-page
- The comps give context for their selection without overwhelming the page
- Josh is already running CSV files for comps with addresses and sold prices
- Josh specifically called out not wanting it to get "overly busy"
- The comp data should inform their choice, not overwhelm the page
- A quick glance, pick where they fit, move on

### Claude's Discretion
- Exact UI layout for comp display (cards vs list vs other)
- How to source comp photos (if available in CSV or MLS data)
- How the tighter range maps to the selection options
- Whether to keep three options or adjust the number of choices

### Deferred Ideas (OUT OF SCOPE)
- Zillow Zestimate API integration for auto-filling rent estimates in admin dashboard -- future admin enhancement
- More sophisticated comp selection/filtering -- v2
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| COMP-01 | Condition picker reframed as "How does your home compare to your neighbors?" | CurrentValue.tsx line 57-59 has the text to change; ConditionPicker.tsx option labels/descriptions to update |
| COMP-02 | 2-3 nearby recent sales shown inline from CSV comp data (address, sold price, optional photo) -- no external links | ConditionCompLinks.tsx to be replaced; CompSale type already has address + soldPrice; conditionCompLinks JSONB column needs richer data |
| COMP-03 | Value range tightened to +/-3-5% from base value | getConditionValues() in calculations.ts lines 23-32 is the function to modify; currently maps original=low, updated=mid, remodeled=high*1.03 |
| COMP-04 | Comp display kept minimal and clean -- informs selection without visual clutter | UI design decision; research recommends compact list over cards for cleanliness |
</phase_requirements>

## Standard Stack

### Core (already in project)
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router framework | Already in use |
| React | 19.2.3 | UI components | Already in use |
| Tailwind CSS | 4.x | Styling | Already in use |
| Motion | 12.36.0 | Animations | Already in use for condition cascade |
| Supabase | 2.99.1 | Database + storage | Already in use for homeowner records |

### Supporting (already in project)
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| PapaParse | 5.5.3 | CSV parsing | Already parses comp CSVs |
| Vitest | 4.0.18 | Testing | Already configured |
| date-fns | 4.1.0 | Date formatting | Already in use |

### No New Libraries Needed
This phase requires zero new dependencies. All changes are to existing components, types, and calculation logic.

## Architecture Patterns

### Current Data Flow (unchanged)
```
Admin CSV Upload -> parseCompCSV() -> deriveValueRange() -> conditionCompLinks stored in Supabase
                                                           -> valueLow/valueHigh stored in Supabase

Server Component [slug]/page.tsx -> getConditionValues(valueLow, valueHigh) -> conditionValues
                                 -> computeHomeownerMetrics() x3 -> conditionVariants

Client Component HomeownerPage -> useState(condition) -> swap pre-computed metrics
                               -> CurrentValue -> ConditionPicker + new InlineComps
```

### Recommended Project Structure Changes
```
src/
  components/
    homeowner/
      ConditionPicker.tsx          # UPDATE: new labels + descriptions
      ConditionCompLinks.tsx       # DELETE: replaced by InlineComps
      InlineComps.tsx              # NEW: 2-3 comp cards, no external links
      CurrentValue.tsx             # UPDATE: new framing text + InlineComps
  lib/
    types.ts                       # UPDATE: ConditionCompLink -> add soldPrice
    calculations.ts                # UPDATE: getConditionValues() clamped range
    comp-analyzer.ts               # UPDATE: deriveValueRange() stores soldPrice
```

### Pattern 1: Clamped Value Range
**What:** Instead of mapping condition values directly to raw comp range (which can be 15-20% wide), clamp the spread to +/-3-5% from a base value (currentValue or midpoint).
**When to use:** Always, when computing condition values from valueLow/valueHigh.
**Example:**
```typescript
// Current implementation (calculations.ts:23-32):
export function getConditionValues(valueLow: number, valueHigh: number) {
  return {
    original: valueLow,
    updated: Math.round((valueLow + valueHigh) / 2),
    remodeled: Math.round(valueHigh * 1.03),
  };
}

// New implementation: clamp to +/-3-5% from base
export function getConditionValues(valueLow: number, valueHigh: number) {
  const base = Math.round((valueLow + valueHigh) / 2);
  const LOWER_PCT = 0.03; // -3% for "original"
  const UPPER_PCT = 0.05; // +5% for "remodeled"

  return {
    original: Math.round(base * (1 - LOWER_PCT)),
    updated: base,
    remodeled: Math.round(base * (1 + UPPER_PCT)),
  };
}
```
**Design rationale:** Josh wants the number not to move too much. Example: $500K base yields $485K / $500K / $525K. The asymmetry (3% down, 5% up) reflects the SB7 insight that homeowners always think their home is worth more -- the upside feels slightly more generous.

### Pattern 2: Inline Comp Display (Simple List, Not Cards)
**What:** A minimal, non-busy display of 2-3 nearby recent sales.
**When to use:** Shown below the condition picker question, above the value display.
**Recommendation:** Use a simple vertical list with address + sold price on one line each. No photos for v1 -- ARMLS CSVs do not include photo URLs, and pulling from Zillow would require external links (violates PAGE-04). Photos can be added later if MLS data evolves.

```typescript
// Recommended: compact list, not card-grid
<div className="space-y-1.5 mb-4">
  <p className="text-xs text-charcoal/50 font-body uppercase tracking-wider">
    Recent sales near you
  </p>
  {comps.map((comp) => (
    <div key={comp.address} className="flex justify-between text-sm text-charcoal/70 font-body">
      <span className="truncate mr-4">{comp.address}</span>
      <span className="font-medium text-charcoal/80 whitespace-nowrap">
        {formatCurrency(comp.soldPrice)}
      </span>
    </div>
  ))}
</div>
```

**Why list over cards:** Josh explicitly wants "not busy." A simple address + price list provides context at a glance without competing visually with the condition picker buttons or the value display. Cards with borders/shadows would add visual weight.

### Pattern 3: Reframed Condition Options
**What:** Update labels and descriptions to be neighbor-comparative rather than self-assessment.
**Example:**
```typescript
const OPTIONS: ConditionOption[] = [
  {
    value: "original",
    label: "Similar to most",
    description: "Comparable to the average home in your neighborhood",
  },
  {
    value: "updated",
    label: "Better than most",
    description: "Your home has upgrades most neighbors haven't made",
  },
  {
    value: "remodeled",
    label: "Best on the block",
    description: "Fully updated -- your home stands out from the rest",
  },
];
```

### Anti-Patterns to Avoid
- **External links from comp display:** Josh and PAGE-04 explicitly prohibit links that take homeowners off the page. No Zillow URLs, no "see this home" links.
- **Too many comps:** Josh said 2-3, not 5-6. More comps create analysis paralysis and visual clutter.
- **Photo integration via Zillow scraping:** Fragile, violates TOS, introduces external dependency. Skip photos unless available in CSV data.
- **Client-side comp recalculation:** Comps should be stored server-side and passed down. No client-side CSV parsing on the homeowner page.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| CSV parsing | Custom parser | PapaParse (already in project) | Edge cases with quoting, encoding, delimiters |
| Currency formatting | Template literals | formatCurrency() from lib/format.ts | Already handles $, commas, no cents |
| Comp scoring | Simple price sort | deriveValueRange() from comp-analyzer.ts | Already handles sqft/bath/pool adjustments |
| Animations | CSS transitions | Motion (already in project) | Consistent with existing cascade pattern |

**Key insight:** This phase modifies existing code rather than building new infrastructure. Every building block (CSV parsing, comp scoring, condition variants, animation cascade) already exists.

## Common Pitfalls

### Pitfall 1: Breaking Backward Compatibility for 200+ Existing Pages
**What goes wrong:** Changing `ConditionCompLink` type or `conditionCompLinks` JSONB schema breaks pages that already have data stored in the old format.
**Why it happens:** The `conditionCompLinks` column already stores `{ original: { address, zillowUrl }, updated: {...}, remodeled: {...} }` for existing records.
**How to avoid:** Make `soldPrice` optional in the new type. The server component should handle both old format (address + zillowUrl only) and new format (address + soldPrice). Transform layer in `rowToHomeownerRecord` already handles null gracefully.
**Warning signs:** Existing published pages showing errors or missing comp data after deployment.

### Pitfall 2: Value Range Still Too Wide After Clamping
**What goes wrong:** If valueLow and valueHigh from comp analysis are wildly different (e.g., $400K vs $600K), the "base" midpoint might not align with what Josh entered as currentValue.
**Why it happens:** `getConditionValues()` currently uses valueLow/valueHigh directly. The new clamped version uses their midpoint, but Josh may have manually adjusted currentValue to something different.
**How to avoid:** Consider using `currentValue` as the base for clamping, not the midpoint of valueLow/valueHigh. The admin already enters currentValue as the "best estimate" -- the condition picker should create a tight band around that.
**Warning signs:** The three condition values don't bracket the displayed currentValue.

### Pitfall 3: Comp Data Not Available for Older Records
**What goes wrong:** Pages created before comp CSV upload was added have `conditionCompLinks: null`. The new inline comp display would show nothing.
**Why it happens:** Not all 200+ records have comp CSV data uploaded.
**How to avoid:** Gracefully hide the comp list when no comp data exists. The condition picker still works (just without comp context). Use optional rendering: `{comps && comps.length > 0 && <InlineComps ... />}`.
**Warning signs:** Blank space or layout shift where comps should be on older pages.

### Pitfall 4: Removing External Links Breaks ConditionCompLinks for Existing Pages
**What goes wrong:** Existing records store `zillowUrl` in `conditionCompLinks`. If the component no longer renders links, the data is harmlessly ignored, but the admin CSV Upload component still generates Zillow URLs.
**Why it happens:** `makeRepComp()` in comp-analyzer.ts generates Zillow URLs from address slugs.
**How to avoid:** Keep `zillowUrl` in the type for backward compat but stop rendering it. In CSVUpload.tsx, the `handleApproveComps()` function can continue storing it (harmless) or be updated to include `soldPrice` from the comp result.

## Code Examples

Verified patterns from the existing codebase:

### Current Condition Picker Integration (CurrentValue.tsx)
```typescript
// Source: src/components/homeowner/CurrentValue.tsx:56-66
{hasMultipleValues && (
  <motion.div variants={childVariants}>
    <p className="text-charcoal font-body text-sm font-semibold mb-1">
      Select your home&apos;s condition
    </p>
    <p className="text-charcoal/50 font-body text-xs mb-3">
      Your selection adjusts the estimated value below
    </p>
    <ConditionPicker
      selected={condition}
      onChange={setCondition}
      revealed={true}
    />
  </motion.div>
)}
```

### Current Value Mapping (calculations.ts)
```typescript
// Source: src/lib/calculations.ts:23-32
export function getConditionValues(
  valueLow: number,
  valueHigh: number
): { original: number; updated: number; remodeled: number } {
  return {
    original: valueLow,
    updated: Math.round((valueLow + valueHigh) / 2),
    remodeled: Math.round(valueHigh * 1.03),
  };
}
```

### CompSale Type from CSV Parser
```typescript
// Source: src/lib/comp-analyzer.ts:1-14
export interface CompSale {
  address: string;
  soldPrice: number;
  sqft: number;
  pricePsf: number;
  beds: number;
  baths: number;
  pool: boolean;
  yearBuilt: number;
  stories: number;
  lotSqft: number;
  closeDate: string;
  subdivision: string;
}
```

### How Comps Are Stored (CSVUpload.tsx)
```typescript
// Source: src/components/admin/CSVUpload.tsx:83-93
const handleApproveComps = () => {
  if (!compResult) return;
  store.setField("valueLow", compResult.valueLow);
  store.setField("valueHigh", compResult.valueHigh);
  store.setField("currentValue", compResult.valueMid);
  store.setField("conditionCompLinks", {
    original: { address: compResult.representativeComps.original.address,
                zillowUrl: compResult.representativeComps.original.zillowUrl },
    updated:  { address: compResult.representativeComps.updated.address,
                zillowUrl: compResult.representativeComps.updated.zillowUrl },
    remodeled: { address: compResult.representativeComps.remodeled.address,
                 zillowUrl: compResult.representativeComps.remodeled.zillowUrl },
  });
};
```

### Server-Side Condition Variant Computation
```typescript
// Source: src/app/h/[slug]/page.tsx:44-59
const conditionValues = hasValueRange
  ? getConditionValues(record.valueLow!, record.valueHigh!)
  : { original: record.currentValue, updated: record.currentValue, remodeled: record.currentValue };

const conditionVariants: ConditionVariants = {
  original: computeHomeownerMetrics({ ...baseInput, currentValue: conditionValues.original }),
  updated: computeHomeownerMetrics({ ...baseInput, currentValue: conditionValues.updated }),
  remodeled: computeHomeownerMetrics({ ...baseInput, currentValue: conditionValues.remodeled }),
};
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single currentValue | valueLow/valueHigh with condition picker | Phase 5 (v1.1) | Three pre-computed variants swapped client-side |
| External Zillow comp links | Will become inline comp display (no links) | Phase 8 (this phase) | Keeps homeowner on page, removes friction |
| Wide value range from raw comps | Will clamp to +/-3-5% | Phase 8 (this phase) | More credible, less alarming value swings |
| "Select your condition" framing | Will become "Compare to neighbors" | Phase 8 (this phase) | Engagement over self-assessment |

**Deprecated/outdated:**
- `ConditionCompLinks.tsx`: Will be replaced by new `InlineComps.tsx` (or inlined into `CurrentValue.tsx`)
- `zillowUrl` field in `ConditionCompLink`: No longer rendered, kept for backward compat

## Open Questions

1. **Should getConditionValues use currentValue or midpoint(valueLow, valueHigh) as the base for clamping?**
   - What we know: Josh enters currentValue manually as his best estimate. The comp-derived valueLow/valueHigh bracket it. The midpoint of valueLow/valueHigh may differ from currentValue.
   - What's unclear: Whether to anchor the +/-3-5% band around currentValue (Josh's judgment) or the comp midpoint (data-driven).
   - Recommendation: Use currentValue as the base. Josh's manual estimate is the "truth" and the condition picker creates a small band around it. This also means getConditionValues() should accept a third parameter (baseValue or currentValue).

2. **Photo availability in comp data**
   - What we know: ARMLS CSV exports include address, sold price, sqft, beds, baths, but NOT photo URLs. Getting photos would require Zillow/Redfin APIs or scraping.
   - What's unclear: Whether Josh expects photos in this phase.
   - Recommendation: Skip photos for now. The CONTEXT.md says "maybe one photo (if available)" -- it is not available from CSV data. The comp display works well with just address + sold price. This keeps it minimal per COMP-04.

3. **Should we keep three condition options or adjust?**
   - What we know: Claude's discretion per CONTEXT.md. Three options is established and familiar.
   - Recommendation: Keep three options. The reframe naturally maps to three comparative levels (similar / better / best). Fewer options (two) loses nuance; more (four+) adds decision friction.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.0.18 |
| Config file | `vitest.config.ts` |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| COMP-01 | Condition picker text reframed | manual | Visual inspection of ConditionPicker.tsx and CurrentValue.tsx | N/A (UI text change) |
| COMP-02 | 2-3 comps shown inline with address + soldPrice, no external links | unit | `npx vitest run src/lib/comp-analyzer.test.ts` | Partial (comp-analyzer tests exist, need InlineComps test) |
| COMP-03 | Value range clamped to +/-3-5% | unit | `npx vitest run src/lib/calculations.test.ts` | Partial (getConditionValues tests needed) |
| COMP-04 | Comp display minimal and clean | manual | Visual inspection | N/A (design decision) |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npx vitest run src/lib/calculations.test.ts src/lib/comp-analyzer.test.ts`
- **Per wave merge:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `src/lib/calculations.test.ts` -- add tests for clamped getConditionValues (COMP-03)
- [ ] `src/lib/comp-analyzer.test.ts` -- add tests for soldPrice in representative comps (COMP-02)
- [ ] No new framework install needed -- Vitest already configured and working

## Sources

### Primary (HIGH confidence)
- **Codebase inspection** -- Direct file reads of all relevant source files:
  - `src/components/homeowner/ConditionPicker.tsx` -- current condition picker UI
  - `src/components/homeowner/ConditionCompLinks.tsx` -- current comp link display (to be replaced)
  - `src/components/homeowner/CurrentValue.tsx` -- section where picker lives
  - `src/components/homeowner/HomeownerPage.tsx` -- client orchestrator
  - `src/app/h/[slug]/page.tsx` -- server component, condition variant computation
  - `src/lib/calculations.ts` -- getConditionValues() and all financial math
  - `src/lib/comp-analyzer.ts` -- deriveValueRange(), CompSale type
  - `src/lib/csv-parser.ts` -- parseCompCSV() with ARMLS column mapping
  - `src/lib/types.ts` -- ConditionCompLink, HomeownerRecord, AdminFormState
  - `src/lib/store.ts` -- Zustand admin store
  - `src/components/admin/CSVUpload.tsx` -- comp approval flow
  - `src/lib/supabase/schema.sql` -- database schema
  - `supabase/migrations/20260313000000_add_loan_and_condition_columns.sql` -- condition_comp_links column

### Secondary (MEDIUM confidence)
- **CONTEXT.md** -- User decisions from discussion with Josh (2026-03-13)
- **REQUIREMENTS.md** -- COMP-01 through COMP-04 requirements
- **STATE.md** -- Project decisions and accumulated context

### Tertiary (LOW confidence)
- None -- all findings verified against codebase

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- no new libraries, all changes to existing code
- Architecture: HIGH -- data flow is well-understood from codebase inspection; all touchpoints identified
- Pitfalls: HIGH -- backward compatibility risks identified from actual schema and 200+ existing pages
- Value clamping formula: MEDIUM -- the exact percentages (3% down, 5% up) are a design decision that may need Josh's validation

**Research date:** 2026-03-14
**Valid until:** 2026-04-14 (stable -- no external dependencies or fast-moving APIs)
