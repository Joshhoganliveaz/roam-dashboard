# Feature Landscape: v1.1 Enhanced Intelligence

**Domain:** Personalized homeowner journey pages -- interactive financial projections and real market data
**Researched:** 2026-03-13
**Scope:** NEW features only. v1.0 table stakes, differentiators, and anti-features documented in git history.

## Strategic Shift from v1.0

v1.0 explicitly listed "Interactive calculators / user-adjustable inputs" as an anti-feature: "This is a curated experience, not a tool." v1.1 deliberately reverses this for specific features (condition picker, rate slider). The new framing: **guided interactivity** -- the homeowner can explore within agent-defined bounds, but Josh still controls the narrative. This mirrors Homebot's "Tune Your Value" pattern where the agent sets the range and the homeowner adjusts within it.

This is not a pivot to dashboard mode. The page remains a narrative experience. Interactivity is limited to two focused controls that deepen engagement without breaking the story flow.

---

## Table Stakes (for v1.1 specifically)

Features that the target v1.1 capabilities demand. Without these, the new features feel broken or incomplete.

| Feature | Why Expected | Complexity | Depends On (v1.0) |
|---------|--------------|------------|-------------------|
| **Cascading recalculation on input change** | When the homeowner picks a condition tier or moves a slider, ALL downstream numbers must update instantly -- equity, scenarios, comparisons, everything. Users of any financial tool (mortgage calculators, tax tools) expect this. A partial update would destroy trust. | Medium | Zustand store (exists), calculation engine (`calculations.ts`), narrative engine (`narrative.ts`) |
| **Value range display showing low/mid/high** | Every home value tool (Zillow, Redfin, Homebot) shows a range, not a point estimate. Presenting a single number invites "is this right?" skepticism. A range says "your home is worth between X and Y depending on condition." | Low | Admin form (exists -- needs 2 new fields: `valueLow`, `valueHigh`) |
| **Multi-timeline projection display** | When showing 5/10/15yr projections, users expect to see all three simultaneously for comparison, not toggled one at a time. Every retirement calculator and investment tool shows multi-horizon results side by side. | Medium | Scenario calculations (exist -- need extension from single-point to multi-horizon) |
| **Rate/assumption disclosure** | Any financial projection tool that shows future numbers must disclose the assumptions ("Based on 3.5% annual appreciation"). Without this, the projections feel like promises. Regulatory norm in financial tools. | Low | None -- pure UI addition to scenario sections |

## Differentiators

Features that set v1.1 apart from competitors and from v1.0.

| Feature | Value Proposition | Complexity | Depends On (v1.0) | Notes |
|---------|-------------------|------------|-------------------|-------|
| **Condition-based value picker (original/updated/remodeled)** | Homebot lets homeowners adjust value on a freeform slider within an AVM range. Zillow lets you edit home facts (bedrooms, sqft, remodel year) and the Zestimate recalculates. Neither frames the question as "What condition is your home in?" -- a question every homeowner can answer intuitively. This turns a sterile number into a conversation: "I've updated my kitchen but haven't touched the bathrooms -- where does that put me?" Three tiers (original, updated, remodeled) are simple enough for a homeowner to self-assess and specific enough for Josh to anchor comps. | Medium | Admin form (needs `valueLow`, `valueHigh`), Zustand store, calculation engine |
| **Cromford appreciation chart (real market data)** | v1.0 uses a straight-line appreciation chart. Real appreciation is lumpy -- the 2021-2022 spike and 2023 correction are the story of Phoenix real estate. Showing the actual Cromford data makes the page feel authoritative and local. No other personalized homeowner tool embeds hyper-local market data from the Cromford Report -- it positions Josh as someone with access to professional-grade data. | High | Appreciation chart component (exists -- needs replacement), Tableau scraper infrastructure (NEW) |
| **Merged Investment Comparison section** | v1.0 has separate S&P comparison and Rent vs Own sections. Merging them into one "Your Home as an Investment" section tells a stronger story: leverage advantage (10:1 on your down payment) PLUS opportunity cost victory (you beat the stock market) PLUS rent savings. The combined narrative is more powerful than three separate data points. | Medium | S&P comparison (exists), Rent vs Own (exists), calculation engine |
| **5/10/15yr projection timelines** | v1.0 scenarios show single-point projections. Adding 5/10/15yr horizons lets homeowners see how time amplifies each strategy. "Hold & Rent covers your mortgage in year 3, generates $800/mo surplus by year 10, and builds $340K in equity by year 15." This is how serious financial tools (Vanguard retirement, Personal Capital) present projections. | High | All three scenario calculations (exist -- need major extension), Zustand store |
| **Market rate slider** | An appreciation rate slider (e.g., 2-6% with a default at the local average) lets homeowners see how different market conditions affect their future. This is the most engaging interactive element -- moving the slider and watching all three scenarios cascade-update creates an "aha" moment. No personalized homeowner tool does this. | Medium | 5/10/15yr projections (new -- must exist first), cascading recalculation (new) |
| **4th combo scenario: "Keep, rent, and buy another"** | The most sophisticated scenario -- and the one most likely to generate a phone call. It models: keep current home as rental, use equity for down payment on new primary, show combined cash flow + dual appreciation. This is the dream scenario for many homeowners and the hardest to reason about alone. Needs Josh as guide. | High | Hold & Rent scenario (exists), Move-Up scenario (exists), rental income calculations, dual-mortgage math |
| **Cascading ripple animation on interaction** | When the homeowner changes condition or moves the rate slider, a visual ripple propagates down the page as each section recalculates. This is borrowed from Material Design's ripple feedback pattern but applied at page scale -- the homeowner sees the cause-and-effect of their choice flowing through their story. Creates a visceral "everything is connected" feeling. | Medium | Cascading recalculation (new), scroll animation system (exists) |
| **Rate lock advantage callout** | For homeowners with sub-4% mortgage rates (pre-2022 buyers), a prominent callout showing how much their locked rate saves vs current rates. "Your 2.875% rate saves you $847/month compared to today's 6.5% rates." This is emotionally powerful and unique to the current rate environment. | Low | Mortgage data (exists in admin form), current rate reference (static or admin-entered) |

## Anti-Features (v1.1)

Features to explicitly NOT build in this milestone.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **Freeform home value slider** | Homebot does this -- homeowner drags to any value in a range. For this product, that creates false precision and removes Josh's comp-anchored authority. Three condition tiers (original/updated/remodeled) are simpler, faster, and maintain the "Josh curated this for you" feeling. | Three-tier condition picker that maps to Josh's low/mid/high comp values |
| **Live Cromford data API** | Cromford Report does not offer a public API. Building a persistent scraper that runs on a schedule creates a maintenance burden and potential ToS issues. | One-time Tableau scrape per property at page creation time. Store the data in Supabase. Screenshot fallback if scrape fails. |
| **Editable mortgage assumptions** | Letting homeowners change their mortgage rate, term, or balance turns this into a mortgage calculator. The admin enters the correct values; the homeowner experiences the story. | Admin-entered mortgage details, displayed but not editable on the homeowner page |
| **Scenario comparison mode** | A side-by-side matrix comparing all 4 scenarios is a dashboard pattern, not a narrative pattern. It breaks the scrollytelling flow. | Present scenarios sequentially in the scroll, each telling its own story. The homeowner absorbs them in order, then calls Josh to discuss which resonates. |
| **Automated Cromford data refresh** | Building a scheduled scraper that refreshes market data monthly adds infrastructure complexity for minimal value -- Josh already manually updates pages when he wants to re-engage. | Manual refresh: Josh re-runs the Tableau scrape when updating a page. One button in admin. |
| **Tax calculation integration** | Cost seg, depreciation, capital gains -- the STR Analyzer handles tax modeling. Adding tax math here conflates the homeowner narrative with investor analysis. | For the combo scenario, show pre-tax cash flow. If the homeowner wants tax analysis, Josh transitions them to the STR Analyzer. |

## Feature Dependencies (v1.1 only)

```
Admin form extensions (valueLow, valueHigh, currentMarketRate)
  |
  ├── Condition picker (maps original/updated/remodeled to low/mid/high)
  |     |
  |     └── Cascading recalculation (Zustand store update triggers)
  |           |
  |           ├── Merged Investment Comparison (recalculates with new value)
  |           ├── Enhanced scenarios (recalculate with new value)
  |           └── Cascading ripple animation (visual feedback of recalc)
  |
  ├── Market rate slider
  |     |
  |     └── 5/10/15yr projection engine (core math)
  |           |
  |           ├── Enhanced Hold & Rent (projected cash flow at 5/10/15yr)
  |           ├── Enhanced Move-Up (equity trajectory at 5/10/15yr)
  |           ├── Enhanced Equity Play (HELOC cost at projected rates)
  |           └── Combo scenario: Keep + Rent + Buy (builds on all three)
  |
  ├── Cromford Tableau scraper (independent track)
  |     |
  |     └── Real appreciation chart (replaces straight-line chart)
  |
  └── Rate lock callout (simple, independent -- just needs mortgage data)

CRITICAL PATH: Admin form extensions → Condition picker → Cascading recalc → Enhanced scenarios → Combo scenario
INDEPENDENT: Cromford scraper, Rate lock callout, Merged comparison
```

## Complexity Assessment

### High Complexity (need phase-specific research)

| Feature | Why High | Risk | Research Needed |
|---------|----------|------|-----------------|
| **Cromford Tableau scraper** | Tableau Public scraping is fragile -- server-side rendering blocks data extraction, URL structures change, and the Cromford Report may reorganize their Tableau embeds at any time. The `tableau-scraping` Python library returns pandas DataFrames but fails silently on server-rendered charts. A screenshot+OCR fallback adds a second failure mode. | Scraper breaks with no warning. Must design for graceful degradation. | Verify Cromford's specific Tableau viz renders client-side. Test `TableauScraper` against the actual URL. Define fallback chain. |
| **5/10/15yr projection engine** | Not just multiplying by an appreciation rate. Each scenario compounds differently: Hold & Rent has rent growth + appreciation + mortgage paydown. Move-Up has sale costs + new purchase + new appreciation. Equity Play has HELOC drawdown + investment returns + rate changes. The market rate slider means every scenario recalculates on every slider move. | Performance on mobile -- recalculating 4 scenarios x 3 horizons x variable rates on every slider change. | Profile Zustand recalculation cost. May need debouncing on slider. Review STR Analyzer's projection patterns. |
| **Combo scenario** | Models TWO properties simultaneously: current home (now rental) + new primary. Inputs include: rental income on current, new mortgage on second property, dual appreciation, combined cash flow, qualification assumptions (lenders use 75% of rental income). This is functionally a mini version of the STR Analyzer. | Scope creep -- could easily become its own product. Must stay narrative, not analytical. | Define exact inputs/outputs. What does Josh actually want to show? What level of detail triggers a "call me" vs satisfying curiosity? |

### Medium Complexity (standard patterns, well-understood)

| Feature | Notes |
|---------|-------|
| Condition picker UI | Three-option selector (radio/card), maps to stored values, triggers Zustand update. Standard pattern. |
| Cascading recalculation | Zustand store already does this in v1.0. Extension to handle condition + rate slider inputs. |
| Merged Investment Comparison | Combining existing sections. Copy/narrative work > engineering work. |
| Market rate slider | Range input with debounced Zustand update. Standard pattern. Main work is in the projection engine it feeds. |
| Cascading ripple animation | CSS animation triggered by state change. Intersection Observer + CSS transitions. No heavy library needed. |

### Low Complexity (straightforward additions)

| Feature | Notes |
|---------|-------|
| Rate lock callout | Conditional display: if `mortgageRate < currentMarketRate`, show savings callout. Simple math, simple UI. |
| Rate/assumption disclosure | Static text beneath projections. No engineering beyond copy. |
| Admin form field additions | Two new number inputs (`valueLow`, `valueHigh`) + one market rate field. Supabase column additions. |

## How Industry Tools Handle These Features

### Home Value Condition Adjustment

**Zillow:** Homeowners claim their home, then edit "home facts" (bedrooms, bathrooms, sqft, remodel year). The Zestimate recalculates based on these structured data points. A "structural remodel" must increase sqft by 10%+ to materially affect the estimate. The approach is granular and data-driven -- homeowners often don't know what to enter.

**Homebot "Tune Your Value":** The agent sets a value range (min/max). The homeowner sees a slider within that range and can adjust freely. Changes take immediate effect on all downstream modules. Homeowners can also note improvements (new windows, remodel) and request a professional CMA. The agent gets notified.

**v1.1 approach (recommended):** Three condition tiers (original / updated / remodeled) mapped to Josh's low/mid/high comp values. Simpler than Zillow's fact editing, more guided than Homebot's freeform slider. The homeowner self-assesses condition; Josh pre-anchors the values to real comps. This creates a conversation opportunity ("I picked 'updated' but I've also done X -- should that change my number?").

### Tableau Public Data Extraction

**How it works:** The `TableauScraper` Python library (PyPI: `TableauScraper`, GitHub: bertrandmartel/tableau-scraping) loads a Tableau Public viz URL, intercepts the client-side data payload, and returns pandas DataFrames per worksheet. It supports filter selection, parameter setting, and tooltip extraction.

**Data format:** Returns `pandas.DataFrame` objects per worksheet. Columns match the Tableau field names. Data types are inferred (dates, numbers, strings).

**Reliability concerns:**
- Server-side rendered vizzes return no data (images only). Must verify Cromford's viz renders client-side.
- Tableau Public URL structure changes break hardcoded URLs.
- Library has 169 commits on GitHub with ongoing maintenance, but no SLA.
- Rate limiting or IP blocking is possible on repeated automated requests.

**Recommended fallback chain:**
1. Primary: `TableauScraper` extracts data, stored in Supabase, rendered as custom chart
2. Fallback: Puppeteer screenshot of the Tableau viz, embedded as image
3. Last resort: Admin manually enters key data points (median price, YoY change), rendered as custom chart from manual data

### Multi-Timeline Financial Projections

**Industry standard:** Vanguard, Fidelity, and Personal Capital show projection fans -- a central "expected" line with optimistic/pessimistic bands. Real estate tools (BiggerPockets calculators, DealCheck) typically show a single timeline with adjustable assumptions.

**Best practice for this product:**
- Show 5/10/15yr as three distinct columns or cards, not a continuous chart
- Each column shows the key outcome number for that horizon
- The market rate slider changes all three simultaneously
- Default rate = local average appreciation (Cromford data if available, otherwise Phoenix metro average)
- Show a range band (optimistic/pessimistic) using +/- 1% from the selected rate

### Combo Scenario: "Keep, Rent, and Buy Another"

**How lenders evaluate this:** 75% of projected rental income offsets the existing mortgage payment for qualification. The borrower must show reserves for both mortgages. Minimum 25% equity in the retained property is typical.

**Key outputs to show:**
1. Monthly cash flow: rental income - old mortgage - management fees - vacancy reserve
2. New home buying power: remaining income qualifies for $X new mortgage
3. Combined wealth at 5/10/15yr: appreciation on both properties + equity paydown on both + rental income accumulation
4. Break-even point: when combined position exceeds selling-and-buying-one scenario

**Narrative framing (SB7):** This is the "hero's boldest path" -- the most complex scenario that requires a guide. The narrative should make it feel possible but complex enough that the homeowner thinks "I need Josh for this."

## Phase Prioritization Recommendation

**Phase A -- Interactive Foundation:**
1. Admin form extensions (valueLow, valueHigh)
2. Condition picker (three tiers)
3. Cascading recalculation on condition change
4. Cascading ripple animation
5. Rate lock callout

Rationale: Smallest increment that adds interactivity. The condition picker alone creates a new engagement moment on every page. Independent of Cromford and projection work.

**Phase B -- Real Market Data:**
1. Cromford Tableau scraper (with fallback chain)
2. Real appreciation chart replacing straight-line
3. Market context in narrative sections

Rationale: High-risk feature isolated in its own phase. If the scraper fails, the fallback chain activates without blocking other work.

**Phase C -- Deep Projections:**
1. 5/10/15yr projection engine
2. Market rate slider
3. Enhanced Hold & Rent, Move-Up, Equity Play scenarios
4. Merged Investment Comparison section

Rationale: Heaviest calculation work. Depends on condition picker (Phase A) being stable since projections build on the selected home value.

**Phase D -- Combo Scenario:**
1. 4th scenario: Keep + Rent + Buy Another
2. Dual-property modeling
3. Qualification assumptions display

Rationale: Most complex feature, highest conversation-generation potential, depends on all three enhanced scenarios being correct first.

## Sources

- [Homebot "Tune Your Value" Module](https://help.homebotapp.com/en/articles/3631274-module-overview-tune-your-value) -- How Homebot handles homeowner value adjustment (MEDIUM confidence)
- [Zillow Zestimate Owner Updates](https://www.zillow.com/premier-agent/make-instant-updates-to-zestimate/) -- How Zillow handles condition/remodel adjustments (HIGH confidence, official Zillow docs)
- [TableauScraper Python Library](https://github.com/bertrandmartel/tableau-scraping) -- Tableau Public data extraction tool, returns DataFrames (HIGH confidence, direct code review)
- [Tableau Embedding API v3](https://help.tableau.com/current/api/embedding_api/en-us/docs/embedding_api_data.html) -- Official Tableau data access from embeds (HIGH confidence, official docs)
- [Buy vs Rent + Stock Market Comparison Tools](https://buyvsrent.vercel.app/) -- Example merged investment comparison calculator (LOW confidence, third-party tool)
- [ROE App - Rental vs Stock Market IRR](https://roeapp.com/) -- Investment comparison methodology using IRR (LOW confidence)
- [Real Estate Financial Modeling Guide](https://madrasaccountancy.com/blog-posts/real-estate-financial-modeling-a-complete-guide-for-investors-cpas/) -- Standard 5-10yr projection methodology (MEDIUM confidence)
- [VP Management - Rent Current Home Guide](https://www.vpmanagement.net/rent-out-your-house-and-buy-another-expert-guide/) -- Keep + rent + buy scenario inputs/requirements (MEDIUM confidence)
- [Redfin vs Zillow Estimates (2026)](https://www.realestateskills.com/blog/redfin-vs-zillow) -- Accuracy comparison of AVMs (MEDIUM confidence)
- [Cromford Report](https://cromfordreport.com/) -- Phoenix market data source (HIGH confidence, direct site)
