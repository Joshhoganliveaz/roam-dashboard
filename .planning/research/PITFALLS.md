# Domain Pitfalls

**Domain:** v1.1 Enhanced Intelligence -- adding interactivity, data scraping, and multi-timeline projections to existing Homeowner Journey Map
**Researched:** 2026-03-13
**Confidence:** HIGH (based on direct codebase inspection + domain-specific research)

## Critical Pitfalls

Mistakes that cause rewrites, data corruption, or broken existing pages.

### Pitfall 1: Tableau Scraper Breaks Silently -- No Data, No Error, No Fallback

**What goes wrong:** The Cromford appreciation chart feature depends on scraping a Tableau Public dashboard via a reverse-engineered, undocumented API. Tableau changes their internal API structure without notice. The scraper returns empty arrays, malformed data, or stale cached responses. Because there is no official API contract, there are no deprecation warnings. One day it works, the next day it returns nothing -- and if the fallback path was not exercised regularly, the OCR fallback silently fails too, leaving pages with a blank chart section or worse, a runtime error that crashes the page.

**Why it happens:** The `tableau-scraping` library (Python, bertrandmartel/tableau-scraping) and its JS equivalents rely on intercepting Tableau's internal HTTP calls and parsing undocumented response structures. GitHub issues document recurring failures: `getWorksheets()` returns empty arrays (Issue #64), filter operations produce `TypeError` exceptions (Issue #76), and the README example itself has stopped working (Issue #77, Feb 2024). This is not a stable foundation. Tableau actively updates their rendering engine (they moved toward React-based rendering in 2024-2025), and each update can silently break scraper parsing.

**Consequences:**
- Homeowner pages display blank or broken chart sections
- If the error is not caught, the entire server component crashes and the page 404s
- Josh has no visibility into which pages have broken charts until a homeowner reports it
- OCR fallback, if untested, likely fails on its own (see Pitfall 2)

**Prevention:**
- Build the Cromford chart feature as a **separate data-fetch step in the admin flow**, not at page render time. When Josh creates/edits a page, the admin tool attempts to scrape Cromford data and stores the result in Supabase. If scraping fails, Josh sees the failure immediately and can use the screenshot/OCR fallback or manually enter key data points
- Never scrape at page render time (`/h/[slug]`). The public page reads pre-fetched data from Supabase only
- Implement a health check: a scheduled job (or manual admin button) that tests the scraper against a known Cromford dashboard and alerts on failure
- Store scraped data with a `scraped_at` timestamp and `scrape_method` field (`'tableau_api' | 'ocr_fallback' | 'manual'`) so stale/failed scrapes are visible
- Design the page component to gracefully degrade: if no chart data exists, show the narrative section without the chart rather than crashing or showing a blank space
- Pin the scraper library version and test against a snapshot of known Cromford dashboard HTML before upgrading

**Detection:**
- Admin dashboard shows "chart data missing" indicator for pages without Cromford data
- Error logging on every scrape attempt with structured output (success/failure, method, timestamp)
- Periodic automated test against the live Cromford dashboard

**Phase to address:** First phase of v1.1 -- build the scraper as an admin-side data pipeline before touching the public page. The fallback chain (Tableau API -> OCR -> manual entry) must be fully functional before any page template changes.

---

### Pitfall 2: OCR Chart Extraction Produces Confident Garbage

**What goes wrong:** When the Tableau scraper fails, the fallback is OCR on a screenshot of the Cromford chart. Tesseract (or similar) reads the chart axis labels and data points. But chart images are the worst case for OCR: small text, overlapping labels, colored backgrounds, non-standard fonts, and visual elements (gridlines, data points) that OCR interprets as characters. The OCR returns numbers that look plausible but are wrong -- "4.2%" becomes "42%", "$485,000" becomes "$485.000" (European decimal), or axis labels are misaligned with values. Because the numbers are plausible-looking, nobody catches the error until a homeowner questions why their chart says Phoenix appreciated 42% last year.

**Why it happens:** Tesseract works best on high-contrast printed text at 300+ DPI. Chart screenshots are typically 72-150 DPI with small axis labels (often 8-10pt equivalent), colored backgrounds that reduce contrast, and mixed content (lines, dots, text, gridlines). Tesseract's page segmentation modes (PSM) default to treating the image as a text block, not a structured chart. Without explicit preprocessing (deskewing, contrast enhancement, region isolation), accuracy on chart labels can drop below 60%.

**Consequences:**
- Appreciation rates shown on the page are wrong, undermining trust in all other data
- Since this is a narrative page (not a dashboard), wrong chart data poisons the story: "Phoenix has seen remarkable 42% annual growth" reads as either laughable or scammy
- Josh has no easy way to verify OCR output against source data at scale

**Prevention:**
- Do NOT use raw OCR as an automated pipeline. Use OCR as an assisted input: the system extracts candidate values, but Josh confirms/corrects them in the admin form before they reach the page
- Pre-process screenshots before OCR: crop to the axis/label region only, convert to grayscale, increase contrast, upscale to 300 DPI, remove gridlines via image processing
- For Cromford charts specifically, the data points are likely annual appreciation rates and median prices. Define expected ranges (appreciation: -5% to +20%, median price: $200K-$800K for Phoenix) and flag any OCR result outside these bounds
- Consider a simpler alternative: instead of OCR, have Josh manually enter 3-5 key data points from the chart (e.g., current appreciation rate, 1yr change, 5yr change). At 20-30 pages/month and shared market data, these numbers change monthly not per-page, so manual entry is 2 minutes of work once a month
- Store OCR confidence scores alongside extracted values. Display a warning in admin when confidence is below threshold

**Detection:**
- Validation rules on extracted values: appreciation rates between -10% and +25%, prices between $100K and $2M
- Admin UI highlights OCR-extracted values in a distinct color with "verify" checkboxes
- Side-by-side display: screenshot on left, extracted values on right, Josh confirms

**Phase to address:** Same phase as the Tableau scraper. The OCR fallback must be built and tested alongside the primary scraper, not as an afterthought. Both should feed into the same admin-verified data pipeline.

---

### Pitfall 3: Adding New Fields to Supabase Breaks Existing Published Pages

**What goes wrong:** v1.1 adds new fields to the homeowners table: `value_low`, `value_high`, `condition` (for the value range selector), Cromford chart data fields, and possibly multi-timeline projection parameters. The migration adds these columns. Existing rows get `NULL` for all new fields. The updated calculation engine or page component now expects these fields to exist. Existing published pages -- the 200+ pages already shared with homeowners via text -- start throwing errors or rendering broken content because `value_low` is null and the new code tries to calculate a range from it.

**Why it happens:** The current `rowToHomeownerRecord()` transform in `src/lib/supabase/transforms.ts` uses direct property access (`Number(row.value_low)`) which returns `NaN` for null values if not handled. The current `computeHomeownerMetrics()` in `src/lib/calculations.ts` takes a `HomeownerInput` with a single `currentValue: number` -- it has no concept of value ranges. Adding `valueLow` and `valueHigh` to `HomeownerInput` as required fields breaks the type contract for all existing records that only have `currentValue`.

**Consequences:**
- Existing homeowner pages 404 or show NaN/$NaN throughout
- ISR-cached versions continue working until cache expires or revalidation triggers, then they break
- Josh discovers the break when a homeowner texts "my page is broken"

**Prevention:**
- All new Supabase columns must have either a DEFAULT value or be nullable. For value range: `value_low numeric DEFAULT NULL`, `value_high numeric DEFAULT NULL`, `condition text DEFAULT NULL`
- The `applyDefaults()` function in `src/lib/defaults.ts` must handle the absence of new fields gracefully. When `valueLow` and `valueHigh` are null, fall back to the existing `currentValue` as both low and high (range of one = single value, backwards compatible)
- The `HomeownerInput` type must keep `currentValue` as the primary field and add `valueLow?: number` and `valueHigh?: number` as optional. The calculation engine uses `currentValue` when range fields are absent
- Write a specific test: `computeHomeownerMetrics()` with a v1.0-shaped input (no range fields) must produce identical output to v1.0
- Run the migration on a copy of production data first. Then load 5 existing pages and verify they render correctly with the new code
- The `rowToHomeownerRecord()` transform must explicitly handle new nullable columns with `?? null` patterns (which it already does for existing optional fields -- extend the same pattern)

**Detection:**
- TypeScript compilation: if `HomeownerInput` adds required fields, existing call sites fail to compile -- this is actually a good detection mechanism, but only if you run `tsc --noEmit` before deploying
- Integration test: render a page with v1.0-shaped data (no new fields) and assert no NaN/undefined appears in the output
- After migration: load 3 random existing published pages and visually verify

**Phase to address:** The very first plan in v1.1 must be the schema migration + backwards compatibility layer. No feature work should happen until existing pages are proven unbroken with the new schema.

---

### Pitfall 4: Cascading Recalculation Turns the Narrative Page Into a Laggy Calculator

**What goes wrong:** The value range selector lets homeowners pick a condition (original/updated/remodeled), which changes the estimated value, which cascades through every downstream calculation: equity, appreciation %, earnings reframe, S&P comparison, rent vs own, net proceeds, and all three scenario cards. On a 3-year-old iPhone on 4G, each condition change triggers a full recalculation of `computeHomeownerMetrics()` plus re-renders of 10+ section components. The slider/selector feels sluggish (200-400ms input lag), animations stutter during recalculation, and the page jumps as sections resize with new numbers. The homeowner taps "Remodeled," waits, sees numbers flash, and the premium narrative experience feels like a broken spreadsheet.

**Why it happens:** The current architecture runs `computeHomeownerMetrics()` once on the server as a pure function. That is fast because it runs once. But v1.1 moves this to the client side for interactive recalculation. The function itself is not expensive (< 1ms on desktop), but the cascade is: React re-renders every component that receives `metrics` as a prop. `HomeownerPage.tsx` passes `metrics` to 10 child components. Without memoization, a single condition change triggers a full tree re-render. On mobile, this combined cost (recalc + re-render + DOM update + animation) can exceed 100ms, causing visible jank.

The market rate slider in the Future Scenarios section compounds this: continuous slider input fires `onChange` events 30-60 times per second, each triggering a full recalculation cascade.

**Consequences:**
- Slider interactions feel laggy on mobile (the primary device)
- Animations stutter during recalculation, breaking the premium feel
- Users spam-click the condition selector, queuing up multiple recalculations
- CountUpNumber animations restart on every recalc, creating visual chaos

**Prevention:**
- Keep `computeHomeownerMetrics()` as a pure function but wrap the client-side call in `useMemo` with `[currentValue, condition]` as dependencies. Only recalculate when the effective value actually changes
- Debounce the market rate slider: recalculate only when the slider stops moving (150ms debounce), not on every pixel of slider movement. Show the rate label updating in real-time but defer the expensive recalculation
- Memoize child components with `React.memo()`: `SP500Callout`, `RentVsOwn`, `NetProceeds`, and each `ScenarioCard` should only re-render when their specific slice of metrics changes, not when any metric changes
- Split the metrics object: instead of passing the entire `HomeownerMetrics` to every component, derive component-specific props. This way `React.memo` shallow comparison works effectively
- Disable or pause CountUpNumber animations during recalculation. When the condition changes, skip the count-up and snap to the new value, or use a quick 200ms transition instead of a full count-up
- For the ripple animation on condition change: use CSS transitions on the numbers themselves (opacity fade + value swap), not a full component unmount/remount cycle
- Test on a real phone with CPU throttling. Chrome DevTools "4x slowdown" is the minimum bar

**Detection:**
- React DevTools Profiler: check that a condition change does not re-render all 10+ section components
- Performance budget: condition change must complete (recalc + re-render + paint) in under 100ms on mobile
- Input lag test: move the market rate slider rapidly on mobile -- if numbers lag behind the slider thumb by more than 200ms, optimization is needed

**Phase to address:** Build the interactive recalculation infrastructure (condition selector + cascading recalc) as its own plan before wiring it to individual sections. Performance testing happens during this plan, not after all sections are updated.

---

### Pitfall 5: Server-to-Client Architecture Shift Creates a Hydration Nightmare

**What goes wrong:** v1.0's page renders entirely on the server: the Server Component in `/h/[slug]/page.tsx` calls `computeHomeownerMetrics()`, passes the result as props to the Client Component `HomeownerPage`. This is clean and fast. v1.1 needs client-side interactivity (condition picker, market rate slider), which means the calculation engine must also run on the client. But the initial render still happens on the server (for SEO, performance, and ISR caching). Now you have the same calculation running in two places: server for initial render, client for interactive updates. If the server-computed metrics and client-computed metrics differ by even a rounding error, React throws a hydration mismatch warning and the page flashes/flickers as it hydrates.

**Why it happens:** The server runs `computeHomeownerMetrics(input)` with the default condition (e.g., "as-is" value). The client hydrates and immediately runs the same function. But `new Date()` is called inside `computeHomeownerMetrics` for `ownershipMonths` calculation. If the server renders at 11:59:59 PM and the client hydrates at 12:00:01 AM, `differenceInMonths` returns a different value. Even a 1-month difference cascades through every metric. Alternatively, floating-point arithmetic might differ between Node.js and the browser engine on edge cases.

**Consequences:**
- Console floods with hydration mismatch warnings
- Page content flickers on load (server HTML replaced by client-rendered HTML)
- In worst case, React falls back to full client-side rendering, destroying the performance benefit of ISR
- Homeowner sees numbers briefly flash to different values on page load

**Prevention:**
- Pass the server-computed `now` date as a serialized prop to the client component. The client uses this same date for initial render, ensuring identical calculations
- Do NOT call `new Date()` in the client-side calculation during hydration. Only use `new Date()` when responding to a user interaction (condition change, slider move)
- Structure the page so the initial render uses server-computed metrics (static), and client-side recalculation only activates after the first user interaction. Before any interaction, the page is pure server-rendered HTML with no client calculation
- Use `suppressHydrationWarning` sparingly and only on elements that genuinely differ (e.g., timestamps), never on financial figures
- Test hydration by comparing server-rendered HTML with client-rendered HTML using React's development mode warnings -- they must be zero

**Detection:**
- React dev mode console: zero hydration warnings on page load
- Visual test: no flickering of numbers on initial page load (record a video of page loading on slow 3G)
- Unit test: `computeHomeownerMetrics(input, fixedDate)` on server matches `computeHomeownerMetrics(input, fixedDate)` on client with exact equality

**Phase to address:** The architecture for client-side interactivity must be designed before any interactive features are built. The pattern is: server renders with fixed date -> client hydrates passively -> user interacts -> client recalculates with `new Date()`. This pattern should be established in the first interactive feature (condition selector) and reused for all subsequent features.

---

### Pitfall 6: Merging S&P + Rent vs Own Into Investment Comparison Breaks Existing Page Structure

**What goes wrong:** v1.0 has separate `SP500Callout` and `RentVsOwn` components rendered as distinct sections in `HomeownerPage.tsx` (sections 7-8 in the narrative arc). v1.1 merges these into a single "Investment Comparison" section. The merge seems simple -- combine two components into one. But the narrative arc depends on section ordering and emotional pacing: the appreciation chart builds confidence, SP500 reinforces it with a specific comparison, then Rent vs Own delivers the ownership triumph. Merging changes the pacing. More concretely:

- The `narrative.ts` copy maps (`rentVsOwnCopy`, and any SP500-related copy) are keyed by `NarrativeVoice` and section type. Removing these sections and adding a new one means updating all three voice variants
- The `HomeownerMetrics` interface has separate `sp500Comparison` and `rentVsOwn` fields that downstream components reference. Changing the interface propagates through types, tests, and components
- Existing pages that are ISR-cached still serve the old HTML until revalidated. During the rollout window, some visitors see old layout, others see new layout

**Consequences:**
- If the merge is incomplete, some narrative copy references deleted sections and throws runtime errors
- Tests that assert on `SP500Callout` or `RentVsOwn` components break
- The narrative pacing is disrupted if the merged section does not maintain the emotional arc
- ISR cache inconsistency during rollout (minor, self-resolving)

**Prevention:**
- Keep `sp500Comparison` and `rentVsOwn` as separate fields in `HomeownerMetrics`. The calculation engine should not change. Only the presentation layer merges them into one visual section
- Create a new `InvestmentComparison` component that receives both `sp500Comparison` and `rentVsOwn` as props. Do not refactor the calculation output
- Add new narrative copy for the merged section to `narrative.ts` alongside the old copy. Remove the old copy only after the new component is fully tested
- Update `HomeownerPage.tsx` to render `InvestmentComparison` in place of the two separate sections. Verify the section count remains correct for animation stagger timing
- After deploy, trigger revalidation for all published pages (a simple script that calls `revalidatePath` for each slug) to clear stale ISR cache
- Write the new section with the same emotional arc: S&P comparison builds leverage story, rent vs own delivers the triumph. The merge should feel like a tighter version of the same story, not a data dump

**Detection:**
- Narrative arc review: read the merged section aloud. Does it still build from comparison to triumph?
- Component test: `InvestmentComparison` renders correctly with all three `NarrativeVoice` variants
- Integration test: full page render produces the expected number of sections and no missing copy
- Visual regression: screenshot comparison of v1.0 and v1.1 page layouts

**Phase to address:** The Investment Comparison section should be built as its own plan after the interactive infrastructure is in place but before the future scenarios rework (since scenarios also change section structure).

---

## Moderate Pitfalls

### Pitfall 7: Financial Projection Accuracy Drifts With Multi-Timeline Compounding

**What goes wrong:** v1.1 adds 5, 10, and 15-year projections for all scenarios. The existing `calcHoldAndRent` already projects forward up to 360 months with a simple monthly compounding model. Extending to 15 years with a user-adjustable market rate slider means compounding small floating-point errors over 180 months. At 4% annual appreciation, a $500K home projects to $900K at 15 years. A 0.1% rounding error per month compounds to a $15K-$25K discrepancy at the 15-year mark. This is within "illustration" tolerance but looks sloppy if two different sections show slightly different 15-year values because they calculated independently.

**Prevention:**
- Use a single projection function that computes all timelines (5/10/15yr) in one pass, returning values at each checkpoint. Do not call three separate functions with the same inputs -- floating-point path differences will produce slightly different results
- Round display values to the nearest $1,000 for projections beyond 5 years. Nobody needs to see "$912,347 projected value in 15 years" -- "$912,000" is more honest about the precision level
- Store intermediate results and reuse them: the 5-year value is a waypoint on the path to 10 years, not an independent calculation
- Add a disclaimer calibrated to timeline length: "5-year estimate based on current trends" vs "15-year illustration -- many factors will change over this period"
- Use `Math.round()` at display time, not during intermediate calculations. Accumulate full precision, round only for presentation

**Detection:**
- Unit test: project the same property at 5yr, 10yr, and 15yr. The 10yr result must exactly equal the 10-year checkpoint from the 15yr projection
- Cross-section consistency check: if the Hold & Rent scenario and the Move-Up scenario both show a 10-year projected value, they must match

**Phase to address:** Part of the future scenarios rework plan. Build the unified projection function before updating any scenario components.

---

### Pitfall 8: Condition Selector UX Creates Decision Paralysis on a Narrative Page

**What goes wrong:** The homeowner opens their journey page -- a curated, personal narrative experience -- and immediately encounters a dropdown: "What condition is your home in? Original / Updated / Remodeled." This transforms them from a story reader into a quiz taker. They do not know what "updated" means in appraisal terms. They worry about picking wrong. They wonder why Josh did not already know this. The interactive element at the top of the page breaks the narrative contract established in v1.0: "this is your story, we prepared it for you."

**Why it happens:** Developers think of the condition picker as a simple UI element. But in the context of a narrative page, it is a cognitive mode shift. The homeowner was passive (reading their story) and is now active (making a decision that changes their numbers). This is the core tension of v1.1: adding interactivity to what was a guided experience.

**Prevention:**
- Do NOT place the condition selector at the top of the page or before the narrative begins. Place it after the current value reveal (section 4), framed as an exploration: "Your home's estimated value is $X. Want to see how improvements could change that?"
- Default to a pre-selected condition (the one Josh chose when creating the page, stored in the admin data). The homeowner sees their numbers immediately without making a choice. The selector is optional exploration, not a required input
- Use clear, homeowner-friendly labels with brief explanations: "As-is (no major updates)" / "Updated (new kitchen, bathrooms, or similar)" / "Fully remodeled (extensive renovation)" -- not technical appraisal terms
- Show the value range visually (a simple bar from low to high) with the current condition highlighted. This makes the interaction feel like "see where you are on the spectrum" rather than "pick the right answer"
- The cascading recalculation should feel delightful (smooth animation, numbers gracefully transitioning) not jarring (flash of new numbers). See Pitfall 4 for the performance side of this

**Detection:**
- User test: have someone unfamiliar with the page open it on their phone. Do they understand the condition picker without explanation? Do they hesitate?
- Narrative flow test: read the page from top to bottom. Does the selector feel like a natural part of the story or an interruption?

**Phase to address:** Design the UX of the condition selector as part of the interactive infrastructure plan. Wire it up in the value/equity sections before cascading to scenarios.

---

### Pitfall 9: Slider Fatigue -- Too Many Interactive Controls Dilute the Narrative

**What goes wrong:** v1.1 adds a condition picker (3 options), a market rate slider (continuous), and 5/10/15yr timeline toggles. Combined with the existing scroll-based narrative, the page now has 3+ interactive controls scattered across sections. Each control demands attention, creates a decision point, and competes with the narrative for cognitive bandwidth. The homeowner stops reading the story and starts playing with sliders. They lose the emotional arc (calm context -> building confidence -> triumphant climax) and instead experience a fragmented calculator with nice fonts.

Hick's Law: decision time increases logarithmically with the number of choices. Adding 3 interactive controls to a previously zero-interaction page is a significant cognitive load increase.

**Prevention:**
- Limit visible controls to 2 maximum on any single screen viewport. The condition picker and market rate slider should never be visible simultaneously
- Make the 5/10/15yr timeline toggle lightweight: segmented control (tab-like), not a dropdown or slider. It should feel like reading different chapters, not adjusting a parameter
- Progressive disclosure: the market rate slider appears only within the Future Scenarios section, not globally. It affects only scenario projections, not the backward-looking story
- Default everything to reasonable values. The page must tell a complete, compelling story with zero interaction. Interactive controls are "explore further" options, not required inputs
- Consider a "what-if" mode toggle: the narrative page is the default experience, and a "what-if explorer" mode reveals all interactive controls for homeowners who want to dig deeper

**Detection:**
- Count the controls: if more than 2 interactive elements are visible at once, the page has crossed into calculator territory
- Read-through test: can someone scroll through the entire page without interacting with any control and still have a complete experience? If not, the interactivity has become a dependency rather than an enhancement

**Phase to address:** UX design for all interactive elements should be planned holistically before individual features are built. Each interactive control should have a documented justification for why it exists and where in the narrative arc it appears.

---

### Pitfall 10: The 4th Combo Scenario Is Exponentially More Complex

**What goes wrong:** The existing three scenarios (Hold & Rent, Move Up, Equity Play) are independent calculations. The 4th combo scenario ("Keep, rent, and buy another") combines elements of all three: keep current home (Hold & Rent cash flow), use equity (Equity Play HELOC), buy a second property (Move Up purchasing power). The calculation requires assumptions that compound: rental income from property 1, HELOC payment on property 1, mortgage on property 2, rental income or personal use of property 2. Each assumption has its own timeline. The projection space is vast, and the narrative framing is harder because there are multiple interacting outcomes.

**Prevention:**
- Keep the combo scenario simpler than the individual scenarios, not more complex. Show the concept and headline numbers, not a full analysis. "You could keep your home as a rental, tap your equity for a down payment, and buy a second property -- here is what that looks like at a high level"
- Use a fixed set of assumptions (the same defaults from `DEFAULTS` in `defaults.ts`) rather than exposing all parameters. The combo scenario is an illustration of possibility, not an underwriting tool -- Josh's STR Analyzer already does detailed investment analysis
- Calculate in stages: (1) Hold & Rent cash flow, (2) HELOC available after maintaining mortgage, (3) purchasing power with HELOC as down payment. These are already-built functions composed together, not a new calculation engine
- Display as a timeline narrative, not a data table: "Year 1: rent your home for $X/mo while your new home builds equity. Year 5: both properties have appreciated, your rental covers its mortgage, and you've built $Y in combined equity."
- Mark this scenario explicitly as an illustration with the strongest disclaimer of all four scenarios

**Detection:**
- Code review: the combo scenario function should call existing calculation functions, not duplicate their logic
- Narrative review: the combo scenario card should be shorter than the individual scenario cards, not longer

**Phase to address:** Build the combo scenario last among the four scenarios. The other three must be updated to v1.1 format (5/10/15yr timelines) first, establishing the pattern that the combo scenario follows.

---

## Minor Pitfalls

### Pitfall 11: ISR Cache Stale During v1.1 Rollout

**What goes wrong:** After deploying v1.1 code, existing ISR-cached pages still serve the v1.0 layout until revalidation. Some homeowners see old layout, some see new.

**Prevention:** After deploying v1.1, run a bulk revalidation script that calls `revalidatePath('/h/${slug}')` for every published page. This is a one-time operation. The existing admin `actions.ts` already has revalidation logic -- extend it to support bulk revalidation.

---

### Pitfall 12: Market Rate Slider Allows Unrealistic Values

**What goes wrong:** The slider has no bounds, and a homeowner drags it to 15% annual appreciation. The 15-year projection shows their $500K home worth $4M. This makes the entire page look like a scam.

**Prevention:** Set slider bounds based on historical Phoenix data: -2% to +8% annual appreciation. Display the current Cromford/market rate as the default and clearly mark it. Add soft labels at bounds: "Conservative" at the low end, "Aggressive" at the high end.

---

### Pitfall 13: Cascading Animation Conflicts With Recalculation

**What goes wrong:** The ripple animation on condition change fires at the same time as CountUpNumber animations, AnimatedSection entrance animations, and chart draw animations. Multiple competing animations create visual chaos on mobile.

**Prevention:** Establish an animation priority system: (1) recalculation transitions take priority, (2) entrance animations only fire once on first view, (3) count-up animations are suppressed during recalculation (snap to value instead). Use a shared animation state flag (`isRecalculating`) that child components check before starting their animations.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Schema migration + backwards compat | Breaking existing published pages (Pitfall 3) | All new columns nullable with defaults; `applyDefaults()` handles absence; integration test with v1.0 data |
| Tableau scraper + OCR pipeline | Silent failure, confident wrong data (Pitfalls 1, 2) | Admin-side pipeline, not render-time; health checks; manual verification step |
| Condition selector + cascading recalc | Mobile performance, hydration mismatch (Pitfalls 4, 5) | Debounce sliders, memoize components, server-date passthrough, passive hydration |
| Merged Investment Comparison | Broken narrative arc, orphaned copy/tests (Pitfall 6) | Keep calc engine unchanged, merge only at presentation layer, update all voice variants |
| Future scenarios rework (5/10/15yr) | Compounding errors, projection inconsistency (Pitfall 7) | Single-pass projection function, round at display only, cross-section consistency test |
| 4th combo scenario | Exponential complexity, overengineered calc (Pitfall 10) | Compose existing functions, keep simpler than individual scenarios, strongest disclaimer |
| UX: interactive controls | Decision paralysis, slider fatigue, calculator feel (Pitfalls 8, 9) | Defaults pre-selected, max 2 controls per viewport, page works with zero interaction |
| Cascading animations | Visual chaos during recalculation (Pitfall 13) | Animation priority system, suppress count-ups during recalc, shared `isRecalculating` flag |
| Deployment/rollout | Stale ISR cache (Pitfall 11) | Bulk revalidation script post-deploy |

## Pitfall-to-Code Mapping

Specific files in the existing codebase that each pitfall affects:

| Pitfall | Files Impacted | Change Required |
|---------|---------------|-----------------|
| Pitfall 3 (schema) | `src/lib/types.ts`, `src/lib/supabase/transforms.ts`, `src/lib/defaults.ts` | Add optional fields to types, handle nulls in transform, extend `applyDefaults()` |
| Pitfall 4 (recalc) | `src/lib/calculations.ts`, `src/components/homeowner/HomeownerPage.tsx` | Wrap in `useMemo`, split metrics prop passing, memoize child components |
| Pitfall 5 (hydration) | `src/app/h/[slug]/page.tsx`, `src/components/homeowner/HomeownerPage.tsx` | Pass `now` date as serialized prop, defer client recalc to first interaction |
| Pitfall 6 (merge) | `src/components/homeowner/SP500Callout.tsx`, `src/components/homeowner/RentVsOwn.tsx`, `src/lib/narrative.ts` | New `InvestmentComparison` component, new narrative copy, remove old sections from orchestrator |
| Pitfall 7 (projections) | `src/lib/calculations.ts`, `src/lib/defaults.ts` | New unified projection function, extended DEFAULTS for timeline params |
| Pitfall 8 (condition UX) | `src/components/homeowner/CurrentValue.tsx` (or new component) | Condition selector placed after value reveal, default pre-selected |
| Pitfall 10 (combo) | `src/lib/calculations.ts`, `src/components/homeowner/ScenarioCards.tsx` | New `calcComboScenario()` composing existing functions, new ScenarioCard |

## Sources

- [tableau-scraping GitHub Issues](https://github.com/bertrandmartel/tableau-scraping/issues) -- documented failures including empty worksheets (Issue #64), TypeError on filters (Issue #76), broken README example (Issue #77) -- HIGH confidence
- [Tesseract OCR Accuracy](https://tesseract-ocr.github.io/tessdoc/ImproveQuality.html) -- official documentation on quality requirements: 300 DPI minimum, preprocessing requirements -- HIGH confidence
- [Tesseract Page Segmentation Modes](https://pyimagesearch.com/2021/11/15/tesseract-page-segmentation-modes-psms-explained-how-to-improve-your-ocr-accuracy/) -- PSM selection for structured content -- MEDIUM confidence
- [Supabase Database Migrations](https://supabase.com/docs/guides/deployment/database-migrations) -- official migration patterns for adding columns -- HIGH confidence
- [React useMemo](https://react.dev/reference/react/useMemo) -- official docs on memoization and when to use it -- HIGH confidence
- [React Performance: Input Lag](https://dev.to/arindam1997007/improving-react-performance-dealing-with-input-lag-397) -- cascading re-render patterns and debounce strategies -- MEDIUM confidence
- [Next.js Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components) -- official docs on hydration and component boundaries -- HIGH confidence
- [Cognitive Overload in UX (Smashing Magazine)](https://www.smashingmagazine.com/2016/09/reducing-cognitive-overload-for-a-better-user-experience/) -- Hick's Law and progressive disclosure -- HIGH confidence
- [Minimize Cognitive Load (NN/g)](https://www.nngroup.com/articles/minimize-cognitive-load/) -- interaction design principles -- HIGH confidence
- [JavaScript Financial Precision](https://dev.to/benjamin_renoux/financial-precision-in-javascript-handle-money-without-losing-a-cent-1chc) -- floating-point rounding in financial calculations -- MEDIUM confidence
- [JavaScript Rounding Errors](https://www.robinwieruch.de/javascript-rounding-errors/) -- IEEE 754 precision issues in compound calculations -- MEDIUM confidence
- Direct codebase inspection of `/Users/joshuahogan/Projects/homeowner-journey-map/src/` -- all file-level claims verified against actual code -- HIGH confidence

---
*Pitfalls research for: Homeowner Journey Map v1.1 Enhanced Intelligence*
*Researched: 2026-03-13*
