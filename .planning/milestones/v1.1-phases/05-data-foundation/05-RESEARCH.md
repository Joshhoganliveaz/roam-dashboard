# Phase 5: Data Foundation - Research

**Researched:** 2026-03-13
**Domain:** Condition-aware home values, Cromford OCR extraction, cascading animations
**Confidence:** HIGH

## Summary

Phase 5 transforms the homeowner page from a single-value static display into an interactive, condition-aware experience with real market data. The work spans three domains: (1) admin form and schema changes to capture a value range instead of a single value, (2) Claude Vision-based OCR to extract time-series data from Cromford Report screenshots, and (3) a theatrical cascading counter-roll animation when homeowners change their condition selection.

The existing codebase is well-structured for this extension. The pure calculation functions in `calculations.ts` accept `currentValue` as a parameter, so pre-computing three condition variants (original/updated/remodeled) server-side is straightforward. The HomeownerPage client orchestrator already receives `record` and `metrics` as props, making the pattern of passing three variant sets natural. The dashboard-generator project already has a working Claude Vision + Cromford extraction pipeline that can be adapted.

**Primary recommendation:** Pre-compute all three condition variant metric sets server-side in the `/h/[slug]/page.tsx` server component, pass them to the client, and use React `useState` to swap between them -- no client-side recalculation needed except chart curve positioning.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Condition picker placed inline with the value section ("Today, your home is worth..."), part of the narrative flow
- Each option shows label + one-liner description to help homeowners self-identify
- Values NOT shown upfront -- selecting a condition reveals the value and cascades the update
- Default condition: "Made some updates" (middle)
- Original = low comp value; Made some updates = midpoint; Completely remodeled = high comp value + 3%
- Josh enters two values in admin: low comp value and high comp value
- No backward compatibility handling needed -- no existing pages to worry about
- Try full time-series extraction first (monthly/quarterly price-per-sqft data points)
- Fall back to shaped curve with single appreciation rate when OCR can't extract clean data points
- Cromford screenshot uploaded during page creation as part of the admin form
- After OCR, show Josh extraction preview with data points -- he approves before data goes into chart
- Chart renders real market data as branded Recharts chart (olive-to-gold gradient)
- Condition selector shifts entire appreciation curve proportionally (price-per-sqft x home sqft x condition factor)
- Counter roll animation -- numbers roll/count from old value to new value (odometer style)
- Theatrical cascade timing: 3-4 seconds total, each section's counter completes before the next starts
- Sequential flow down the page section by section
- All sections animate regardless of viewport position
- Appreciation chart curve shifts smoothly (morph, not redraw)

### Claude's Discretion
- Condition picker component style (segmented control vs cards vs other)
- Exact one-liner descriptions for each condition option
- Counter roll animation easing and per-section timing within the 3-4 second budget
- OCR extraction approach (Claude Vision prompt engineering for Cromford screenshots)
- Shaped curve algorithm for appreciation rate fallback
- Admin form layout for the new value range fields and Cromford upload
- How the OCR preview is displayed to Josh (inline, modal, etc.)

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| VALU-01 | Josh can enter low comp value and high comp value in admin form | Admin form extension pattern documented; Supabase schema migration for value_low/value_high columns |
| VALU-02 | Homeowner sees three condition choices that map to positions within comp range | Condition picker UX patterns researched; value mapping formula defined (low/midpoint/high+3%) |
| VALU-03 | Selecting condition recalculates all downstream numbers without page reload | Pre-computed 3 variant sets pattern; existing computeHomeownerMetrics can be called 3x server-side |
| VALU-04 | Condition change triggers cascading ripple animation | Motion library useMotionValue/useSpring/animate primitives available; counter-roll pattern documented |
| CROM-01 | Josh can upload Cromford Report screenshot in admin form | Existing PhotoUpload/FileDropZone pattern; new file upload field in admin form |
| CROM-02 | System extracts price-per-sqft time-series via vision/OCR | Claude Vision API pattern proven in dashboard-generator; custom prompt for time-series extraction |
| CROM-03 | Appreciation chart displays real market data as branded Recharts chart | Existing AppreciationChart.tsx extended with real data points; Recharts Area animation props |
| CROM-04 | Condition selector shifts appreciation curve proportionally | Chart data transformation: multiply price-per-sqft by sqft by condition factor |
</phase_requirements>

## Standard Stack

### Core (Already Installed)
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 16.1.6 | App Router, server components, API routes | Already in project |
| React | 19.2.3 | UI framework | Already in project |
| Recharts | 3.8.0 | Chart rendering with animation | Already used for AppreciationChart |
| Motion | 12.36.0 | Animation primitives (useMotionValue, useSpring, animate) | Already used for scroll animations |
| Supabase | 2.99.1 | Database, storage, RLS | Already in project |
| Vitest | 4.0.18 | Unit testing | Already in project |

### New Dependencies
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| @anthropic-ai/sdk | (none needed) | Claude Vision API | Use raw fetch to api.anthropic.com like dashboard-generator does |

### No New Dependencies Required
The project already has everything needed. The Claude Vision API call will use raw `fetch` to `https://api.anthropic.com/v1/messages` matching the pattern in the dashboard-generator project's `claude-api.ts`. No SDK install needed.

**Installation:**
```bash
# No new packages needed. Only env var addition:
# Add ANTHROPIC_API_KEY to .env.local
```

## Architecture Patterns

### Pre-Computed Condition Variants (Server-Side)

**What:** Call `computeHomeownerMetrics()` three times on the server with different `currentValue` inputs, pass all three result sets to the client.

**Why:** Avoids client-side calculation complexity. The existing `computeHomeownerMetrics` is a pure function -- calling it 3x with different values is trivial and deterministic.

**Example:**
```typescript
// In src/app/h/[slug]/page.tsx (server component)
import { computeHomeownerMetrics } from "@/lib/calculations";

// Condition value mapping
function getConditionValues(valueLow: number, valueHigh: number) {
  return {
    original: valueLow,
    updated: Math.round((valueLow + valueHigh) / 2),
    remodeled: Math.round(valueHigh * 1.03),
  };
}

// Pre-compute all three variants
const conditionValues = getConditionValues(record.valueLow, record.valueHigh);
const metricsOriginal = computeHomeownerMetrics({ ...input, currentValue: conditionValues.original });
const metricsUpdated = computeHomeownerMetrics({ ...input, currentValue: conditionValues.updated });
const metricsRemodeled = computeHomeownerMetrics({ ...input, currentValue: conditionValues.remodeled });

// Pass to client
return (
  <HomeownerPage
    record={record}
    conditionVariants={{
      original: metricsOriginal,
      updated: metricsUpdated,
      remodeled: metricsRemodeled,
    }}
    defaultCondition="updated"
    cromfordData={record.cromfordData}
  />
);
```

### Client-Side Condition Switching (useState)

**What:** HomeownerPage uses `useState<'original' | 'updated' | 'remodeled'>` to track selected condition, swaps pre-computed metrics.

**Why:** Decided in STATE.md -- use useState not Zustand for homeowner page state. Instant swap, no calculation delay.

**Example:**
```typescript
// In HomeownerPage.tsx
const [condition, setCondition] = useState<Condition>(defaultCondition);
const metrics = conditionVariants[condition];
// All child components receive the active metrics -- when condition changes, React re-renders with new values
```

### Cascading Counter-Roll Animation Pattern

**What:** When condition changes, numbers roll from old value to new value in sequence, section by section, over 3-4 seconds.

**Why:** Theatrical reveal matching the cinematic feel from Phase 3. Each section gets its moment.

**Implementation approach:**
1. Replace `useCountUp` (counts from 0, plays once on scroll) with a new `useCounterRoll` hook that animates between two values
2. Use `motion`'s `animate()` function (imperative API) to tween between old and new values
3. Cascade timing: each section starts after the previous completes. Use a `cascadeDelay` prop based on section index
4. Total budget: 3-4 seconds, divided across ~6 animated sections = ~500-600ms per section

**Example:**
```typescript
// useCounterRoll hook
import { animate } from "motion/react";

export function useCounterRoll(
  target: number,
  trigger: unknown, // condition key -- changes trigger animation
  delay: number = 0,
  duration: number = 500
) {
  const [display, setDisplay] = useState(target);
  const prevTarget = useRef(target);

  useEffect(() => {
    if (prevTarget.current === target) return;
    const from = prevTarget.current;
    prevTarget.current = target;

    const timeout = setTimeout(() => {
      const controls = animate(from, target, {
        duration: duration / 1000,
        ease: "easeInOut",
        onUpdate: (v) => setDisplay(Math.round(v)),
        onComplete: () => setDisplay(target),
      });
      return () => controls.stop();
    }, delay);

    return () => clearTimeout(timeout);
  }, [target, delay, duration]);

  return display;
}
```

### Cromford OCR Pipeline

**What:** API route receives screenshot, sends to Claude Vision with extraction prompt, returns structured time-series data for Josh to preview and approve.

**Architecture:**
```
Admin Form Upload
  -> POST /api/cromford/extract
  -> Convert image to base64
  -> Send to Claude Vision API with time-series extraction prompt
  -> Return { dataPoints: [{date, pricePsf}], status: 'success' | 'fallback' }
  -> Show preview to Josh in admin form
  -> Josh approves -> data saved to cromford_data JSON column
```

**Example prompt for time-series extraction:**
```typescript
const CROMFORD_TIMESERIES_PROMPT = `You are analyzing a Cromford Report screenshot showing price-per-square-foot data over time for a specific city/area in Phoenix metro.

Extract the time-series data points visible in the chart. For each data point, provide:
- date: The month/year (format: "YYYY-MM")
- pricePsf: The price per square foot value at that point

Also extract:
- city: The city or area name shown in the report
- dateRange: The time period covered (e.g., "2019-01 to 2025-12")
- chartType: What the chart shows (e.g., "Monthly Average $/SqFt")

Return a JSON object:
{
  "city": "...",
  "chartType": "...",
  "dateRange": "...",
  "dataPoints": [
    {"date": "2019-01", "pricePsf": 185},
    {"date": "2019-02", "pricePsf": 187},
    ...
  ]
}

Extract as many data points as you can read from the chart. If exact values are unclear, estimate based on the chart's axis scales. Focus on accuracy of the overall shape and trend rather than pixel-perfect values.

Return ONLY the JSON object.`;
```

### Appreciation Chart with Real Data

**What:** Replace the current `generateChartData()` function (which creates a smooth exponential curve from purchase to today) with real Cromford data points when available.

**Key transformation:** Cromford data is in price-per-sqft. To show home value:
```
homeValue = pricePsf * homeSqft * conditionFactor
```

Where conditionFactor:
- Original: `valueLow / (midpointPsf * sqft)` -- derives the factor from the comp range
- Updated: 1.0 (baseline)
- Remodeled: `valueHigh * 1.03 / (midpointPsf * sqft)`

**Chart morphing on condition change:** Use Recharts' `animationDuration` and `animationEasing` props. When data updates (new condition selected), Recharts automatically animates the Area from old data to new data if `isAnimationActive={true}`.

### Shaped Curve Fallback

**What:** When OCR can't extract clean time-series data, generate a realistic-looking curve using the single appreciation rate.

**Why:** Straight lines look fake. Real markets have dips, surges, and plateaus.

**Algorithm:** Apply a noise function to the exponential growth curve:
```typescript
function generateShapedCurve(
  purchasePrice: number,
  currentValue: number,
  months: number,
  annualRate: number
): ChartPoint[] {
  // Seed deterministic noise from purchasePrice (same home = same curve shape)
  const seed = purchasePrice % 1000;
  const points: ChartPoint[] = [];

  for (let m = 0; m <= months; m++) {
    const baseValue = purchasePrice * Math.pow(1 + annualRate, m / 12);
    // Seasonal variation (spring surge, winter dip)
    const seasonal = Math.sin((m / 12) * 2 * Math.PI) * 0.015;
    // Market cycle noise (longer period)
    const cycle = Math.sin(((m + seed) / 36) * 2 * Math.PI) * 0.02;
    const value = baseValue * (1 + seasonal + cycle);
    points.push({ date: formatDate(m), value: Math.round(value) });
  }

  // Pin last point to actual current value
  points[points.length - 1].value = currentValue;
  return points;
}
```

### Supabase Schema Migration

**New columns needed:**
```sql
ALTER TABLE homeowners
  ADD COLUMN value_low numeric,
  ADD COLUMN value_high numeric,
  ADD COLUMN home_sqft numeric,
  ADD COLUMN cromford_data jsonb;

-- cromford_data structure:
-- {
--   "city": "Scottsdale",
--   "chartType": "Monthly Average $/SqFt",
--   "dataPoints": [{"date": "2019-01", "pricePsf": 185}, ...],
--   "extractedAt": "2026-03-13T..."
-- }
```

**Why nullable:** All new columns are nullable. When `value_low` and `value_high` are both null, fall back to `current_value` (single value mode). This maintains compatibility.

### Project Structure Changes

```
src/
  app/
    api/
      cromford/
        extract/route.ts      # NEW: Claude Vision OCR endpoint
    h/[slug]/page.tsx          # MODIFY: pre-compute 3 condition variants
  components/
    homeowner/
      ConditionPicker.tsx      # NEW: inline condition selector
      HomeownerPage.tsx        # MODIFY: add condition state, cascade animation
      CurrentValue.tsx         # MODIFY: integrate condition picker
      AppreciationChart.tsx    # MODIFY: support real Cromford data + morphing
      CountUpNumber.tsx        # MODIFY: support counter-roll between values
    admin/
      HomeownerForm.tsx        # MODIFY: add value range fields, Cromford upload
      CromfordPreview.tsx      # NEW: OCR extraction preview for Josh
  lib/
    claude-api.ts              # NEW: Claude Vision API wrapper (raw fetch)
    cromford.ts                # NEW: Cromford data processing utilities
    hooks/
      useCountUp.ts            # MODIFY or replace with useCounterRoll
      useCounterRoll.ts        # NEW: animate between old/new values
    types.ts                   # MODIFY: add condition types, cromford data types
    calculations.ts            # NO CHANGES -- already accepts currentValue param
    supabase/
      schema.sql               # MODIFY: add new columns
      transforms.ts            # MODIFY: add new fields to rowToHomeownerRecord
```

### Anti-Patterns to Avoid
- **Client-side recalculation:** Do NOT recalculate metrics on the client. Pre-compute server-side, swap client-side
- **Zustand for homeowner page state:** Use useState as decided in STATE.md
- **Straight-line fallback chart:** Even fallback must have realistic market curve shape
- **Blocking save on OCR failure:** OCR extraction should be optional -- Josh can still create a page without Cromford data
- **Hardcoding condition factors:** Derive condition factors from the actual comp range values, not magic numbers

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Number animation between values | Custom RAF loop from scratch | Motion's `animate()` imperative API | Handles easing, cancelation, cleanup automatically |
| Chart data transition | Manual SVG path interpolation | Recharts built-in animation props | `animationDuration` + `animationEasing` handle Area morphing |
| Image to base64 conversion | Custom file reader | `Buffer.from(await file.arrayBuffer()).toString('base64')` | Standard pattern used in dashboard-generator |
| JSON parsing from Claude | Regex extraction | Parse with try/catch + structured prompt | Claude returns clean JSON when prompted correctly |

**Key insight:** The motion library already installed provides the `animate()` imperative API for tweening numeric values -- no need for a custom requestAnimationFrame implementation for the counter-roll.

## Common Pitfalls

### Pitfall 1: Recharts Animation on Data Update
**What goes wrong:** Recharts may not animate when data array reference changes but keys remain the same
**Why it happens:** React reconciliation + Recharts animation triggers can be tricky
**How to avoid:** Use a `key` prop on the AreaChart that changes with condition selection, forcing a fresh mount with animation. Or ensure `isAnimationActive={true}` is always set.
**Warning signs:** Chart snaps to new position without smooth transition

### Pitfall 2: OCR Accuracy on Cromford Screenshots
**What goes wrong:** Claude Vision estimates values from chart visualizations; exact numbers may be off by 5-10%
**Why it happens:** Chart axis scales, resolution, and overlapping labels make pixel-perfect extraction unreliable
**How to avoid:** The extraction preview step (Josh approves) catches errors. Frame the chart as "narrative, not legal" -- directional accuracy is sufficient per Out of Scope decisions. Let Josh manually edit individual data points if needed.
**Warning signs:** Extracted values don't match the visual curve shape

### Pitfall 3: Cascade Animation Conflicts with Scroll Animation
**What goes wrong:** Phase 3 added scroll-triggered count-up animations (play once). Condition-change counter-roll must work differently.
**Why it happens:** `useCountUp` uses `hasAnimated.current = true` to prevent replay
**How to avoid:** Two distinct animation modes: (1) initial scroll reveal (count from 0, play once) and (2) condition-change roll (animate from old to new, repeatable). The `useCounterRoll` hook handles the second case and should only activate after the initial scroll animation has completed.
**Warning signs:** Numbers don't animate on condition change, or initial scroll animation replays

### Pitfall 4: Condition Factor for Appreciation Curve
**What goes wrong:** The appreciation curve shift needs to be proportional across all time periods, not just the current value
**Why it happens:** Naively multiplying the current value by a factor doesn't shift historical data correctly
**How to avoid:** The condition factor should be applied to price-per-sqft (the Cromford raw data), then multiplied by home sqft. This ensures the entire curve shifts proportionally. For shaped fallback curves, apply the factor to the base growth curve.
**Warning signs:** Historical values are unrealistically high/low for a given condition

### Pitfall 5: Large Base64 Images in API Route
**What goes wrong:** Cromford screenshots can be large (2-5MB), and base64 encoding inflates size by 33%
**Why it happens:** High-resolution Cromford Report screenshots
**How to avoid:** Resize/compress the image on the client before uploading to the API route. Or use Supabase Storage as an intermediary. Claude Vision supports images up to 20MB base64.
**Warning signs:** API route timeouts, slow extraction responses

## Code Examples

### Admin Form Value Range Fields
```typescript
// In HomeownerForm.tsx - Financials fieldset
// Replace single "Current Value" field with Low/High comp fields
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2">
  <div>
    <label className="block text-sm font-medium text-charcoal mb-1">
      Low Comp Value *
    </label>
    <input
      type="number"
      value={store.valueLow || ""}
      onChange={(e) => store.setField("valueLow", Number(e.target.value) || 0)}
      placeholder="440000"
      className="w-full rounded-md border border-canyon/30 ..."
    />
  </div>
  <div>
    <label className="block text-sm font-medium text-charcoal mb-1">
      High Comp Value *
    </label>
    <input
      type="number"
      value={store.valueHigh || ""}
      onChange={(e) => store.setField("valueHigh", Number(e.target.value) || 0)}
      placeholder="520000"
      className="w-full rounded-md border border-canyon/30 ..."
    />
  </div>
</div>
```

### Claude Vision API Call (Raw Fetch)
```typescript
// src/lib/claude-api.ts
export async function extractCromfordData(
  imageBase64: string,
  mediaType: string
): Promise<CromfordExtraction> {
  const apiKey = process.env.ANTHROPIC_API_KEY;
  if (!apiKey) throw new Error("ANTHROPIC_API_KEY not set");

  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": apiKey,
      "anthropic-version": "2023-06-01",
    },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: 4096,
      messages: [{
        role: "user",
        content: [
          { type: "image", source: { type: "base64", media_type: mediaType, data: imageBase64 } },
          { type: "text", text: CROMFORD_TIMESERIES_PROMPT },
        ],
      }],
    }),
  });

  const data = await response.json();
  const text = data.content?.find((c: { type: string }) => c.type === "text")?.text || "";
  return JSON.parse(text) as CromfordExtraction;
}
```

### Condition Picker Component (Segmented Control Style)
```typescript
// src/components/homeowner/ConditionPicker.tsx
type Condition = "original" | "updated" | "remodeled";

const CONDITIONS = [
  { key: "original", label: "Mostly original", desc: "Original fixtures, no major renovations" },
  { key: "updated", label: "Made some updates", desc: "Kitchen or bath refresh, newer flooring" },
  { key: "remodeled", label: "Completely remodeled", desc: "Full renovation, modern finishes throughout" },
] as const;

export function ConditionPicker({
  selected,
  onChange,
}: {
  selected: Condition;
  onChange: (c: Condition) => void;
}) {
  return (
    <div className="flex flex-col sm:flex-row gap-2 sm:gap-3 mt-4">
      {CONDITIONS.map(({ key, label, desc }) => (
        <button
          key={key}
          onClick={() => onChange(key)}
          className={`flex-1 rounded-xl px-4 py-3 text-left transition-all border-2 ${
            selected === key
              ? "border-olive bg-olive/10 shadow-sm"
              : "border-canyon/20 hover:border-canyon/40"
          }`}
        >
          <span className="block text-sm font-semibold text-charcoal">{label}</span>
          <span className="block text-xs text-charcoal/60 mt-0.5">{desc}</span>
        </button>
      ))}
    </div>
  );
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| framer-motion package | motion package (v12+) | 2024 | Same API, new package name; already migrated in this project |
| Recharts v2 | Recharts v3 | 2024-2025 | Better animation support, React 19 compatibility; already on v3.8.0 |
| Anthropic SDK | Raw fetch to API | N/A | Dashboard-generator uses raw fetch; keeps bundle small, no extra dependency |
| Single value input | Value range (low/high) | Phase 5 | Enables condition-aware calculations |

**Deprecated/outdated:**
- `framer-motion` package name: Use `motion` instead (already done in this project)
- `useCountUp` for condition changes: Needs to be supplemented with `useCounterRoll` for between-value animation

## Open Questions

1. **Home square footage source**
   - What we know: Cromford data is in price-per-sqft; we need home sqft to compute absolute values
   - What's unclear: The current schema doesn't have a `home_sqft` column. Josh would need to enter it, or we derive it from `currentValue / pricePsf`
   - Recommendation: Add `home_sqft` to admin form (optional, with smart default derivable from value and market data). If Cromford data is present, we can derive sqft from `currentValue / currentPricePsf`

2. **OCR extraction quality at different resolutions**
   - What we know: Claude Vision works best with high-resolution, cropped images. Cromford Report screenshots vary in quality.
   - What's unclear: Exact extraction accuracy for monthly data points from chart visualizations
   - Recommendation: The approval preview step handles this. Build it, test with real screenshots, iterate on the prompt. The "fallback to shaped curve" provides a safety net.

3. **Recharts Area chart morphing behavior**
   - What we know: Recharts has `animationDuration` and `animationEasing` on Area components. Setting a `key` prop forces remount with animation.
   - What's unclear: Whether updating data in-place (same component instance) produces a smooth morph vs a redraw
   - Recommendation: Test both approaches (key-based remount vs data update). If in-place data update doesn't morph, use `key={condition}` to force a clean animated entrance.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.0.18 + jsdom |
| Config file | `vitest.config.ts` (exists) |
| Quick run command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |
| Full suite command | `cd ~/Projects/homeowner-journey-map && npx vitest run` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| VALU-01 | Admin form accepts low/high comp values and saves to DB | manual-only | N/A (admin form UI test) | N/A |
| VALU-02 | getConditionValues maps original/updated/remodeled to correct positions | unit | `npx vitest run src/lib/calculations.test.ts` | Extend existing |
| VALU-03 | computeHomeownerMetrics produces correct results for each condition value | unit | `npx vitest run src/lib/calculations.test.ts` | Extend existing |
| VALU-04 | Cascading animation triggers on condition change | manual-only | N/A (visual animation) | N/A |
| CROM-01 | Cromford upload accepted in admin form | manual-only | N/A (form UI) | N/A |
| CROM-02 | Claude Vision extraction returns structured time-series | unit | `npx vitest run src/lib/cromford.test.ts` | Wave 0 |
| CROM-03 | Chart renders with Cromford data points | manual-only | N/A (Recharts visual) | N/A |
| CROM-04 | Condition factor shifts Cromford curve proportionally | unit | `npx vitest run src/lib/cromford.test.ts` | Wave 0 |

### Sampling Rate
- **Per task commit:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Per wave merge:** `cd ~/Projects/homeowner-journey-map && npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `src/lib/cromford.test.ts` -- covers CROM-02 (data processing/transformation) and CROM-04 (curve shifting)
- [ ] Extend `src/lib/calculations.test.ts` -- covers VALU-02 (condition value mapping) and VALU-03 (pre-computed variants correctness)
- [ ] `src/lib/hooks/useCounterRoll.test.ts` -- covers counter-roll animation logic (unit testable portion)

## Sources

### Primary (HIGH confidence)
- Project codebase: `src/lib/calculations.ts`, `src/components/homeowner/*.tsx`, `src/lib/types.ts` -- direct code inspection
- Dashboard-generator codebase: `src/lib/claude-api.ts`, `src/lib/claude-prompts.ts` -- proven Claude Vision + Cromford extraction pattern
- Package versions: Recharts 3.8.0, Motion 12.36.0, Next.js 16.1.6 -- from `package.json` and `node_modules`
- Motion library exports: `useMotionValue`, `useSpring`, `useTransform`, `animate` -- verified via `framer-motion/dist/es/index.mjs` export scan

### Secondary (MEDIUM confidence)
- [Recharts animation configuration](https://github.com/recharts/recharts/issues/375) -- animationDuration, animationBegin, animationEasing props
- [Motion AnimateNumber docs](https://motion.dev/docs/react-animate-number) -- not available in v12.36.0, but `animate()` imperative API confirmed
- [Claude Vision API docs](https://platform.claude.com/docs/en/build-with-claude/vision) -- image content type in messages, up to 20 images per request

### Tertiary (LOW confidence)
- [Recharts data transition behavior](https://app.studyraid.com/en/read/11352/354993/animation-configuration-options) -- unclear if in-place data updates produce smooth morphs vs redraws in v3
- Claude Vision numeric extraction accuracy on chart images -- known limitation, mitigated by Josh's approval step

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all libraries already installed and in use
- Architecture: HIGH -- pre-compute pattern validated in STATE.md decisions, calculations.ts already pure and parameterized
- Cromford OCR: MEDIUM -- proven pattern from dashboard-generator but time-series extraction is more demanding than metric extraction
- Animation: MEDIUM -- Motion's `animate()` is confirmed available, but Recharts chart morphing needs testing
- Pitfalls: HIGH -- identified from codebase analysis of existing animation system and Claude Vision limitations

**Research date:** 2026-03-13
**Valid until:** 2026-04-13 (stable stack, no fast-moving dependencies)
