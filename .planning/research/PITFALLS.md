# Pitfalls Research

**Domain:** Personalized homeowner journey pages (real estate narrative experiences)
**Researched:** 2026-03-11
**Confidence:** HIGH (domain-specific research + existing project patterns + industry evidence)

## Critical Pitfalls

### Pitfall 1: Appreciation Numbers That Don't Match the Homeowner's Reality

**What goes wrong:**
You show a homeowner their home has appreciated 22% since purchase, but they checked Zillow last week and it said 18%. Or worse, you show $85K in equity gain but their neighbor just sold for less than expected. The homeowner immediately distrusts the entire page and, by extension, Josh's expertise. Zillow's own Zestimate has a 7% median error rate on off-market homes -- a $500K home could be off by $35K. Any number you present will be compared against whatever the homeowner already believes.

**Why it happens:**
Developers treat appreciation as a simple math problem (current value - purchase price = gain). But "current value" is an estimate, not a fact. Different sources (Zillow, Redfin, county assessor, recent comps) give different numbers. Presenting a single number as definitive truth is the mistake.

**How to avoid:**
- Never present a single "your home is worth $X" number. Instead, frame appreciation as a range or use purchase-price-anchored metrics: "Based on comparable sales in your area, homes like yours have appreciated approximately 18-24% since [purchase date]."
- Use hedging language that reinforces Josh's expertise: "Based on recent comparable sales in [neighborhood]..." rather than "Your home is worth..."
- Store the data source and date for every value used. Display "as of [date]" on all value-dependent sections.
- When manually entering data, create a simple validation checklist: Does this match within 5% of at least two public sources?

**Warning signs:**
- A homeowner texts back asking "where did you get that number?" -- this means the figure conflicts with their mental model
- Appreciation percentages that seem too good (or bad) relative to the Phoenix metro average
- No source attribution anywhere on the page

**Phase to address:**
Phase 1 (data model and input). Build the data schema with source attribution and date fields from day one. The narrative template must include hedging language as a default, not an afterthought.

---

### Pitfall 2: The "Automated Report" Feel That Kills Differentiation

**What goes wrong:**
The page looks and feels like Homebot, a Zillow email, or a generic CMA -- exactly the thing Josh wants to differentiate from. Homeowners glance at it, think "another automated home value thing," and close the tab. The entire value proposition -- a personal, narrative experience from their agent -- is lost.

**Why it happens:**
Developers default to dashboard patterns: data cards, stat grids, chart-heavy layouts. These are efficient for displaying information but terrible for storytelling. The SB7 framework demands narrative flow where each section builds on the previous one, but most web templates are designed for random-access information consumption.

**How to avoid:**
- Design the page as a vertical story, not a dashboard. No sidebar navigation, no "jump to section" links, no data cards in a grid layout.
- Every section must open with a narrative hook (SB7 requirement), not a number. Wrong: "Equity: $87,000". Right: "Since you moved into [address] in [year], your home has been quietly building wealth."
- Use scroll-triggered reveals so content appears as the homeowner progresses through the story, reinforcing the narrative arc.
- Test the differentiation: show the page to someone unfamiliar with it. If they say "oh, like Homebot?" the design has failed.
- Limit charts to 2-3 maximum. Each chart must be embedded within narrative context, not standing alone.

**Warning signs:**
- The page has more than 3 standalone charts/graphs without narrative framing
- Section headers are data labels ("Equity Summary") rather than story beats ("What your home has been doing while you sleep")
- The page works equally well if you randomize section order -- this means there is no narrative arc

**Phase to address:**
Phase 2 (template and narrative design). This is a design decision, not a data decision. The template structure must enforce narrative flow before any data visualization work begins.

---

### Pitfall 3: Mobile Scroll Performance Death by Animation

**What goes wrong:**
The scrollytelling experience is beautiful on a MacBook Pro but stutters, jitters, or freezes on a 3-year-old iPhone opening the link from a text message. 60%+ of recipients will view this on mobile. A laggy scroll experience feels broken and unprofessional -- the opposite of the premium brand impression Josh needs.

**Why it happens:**
Scroll-triggered animations are CPU/GPU intensive. Developers test on fast machines with large screens. Common offenders: parallax effects on large images, SVG chart animations triggered on every scroll event, CSS transforms without `will-change` hints, and loading all chart data/assets upfront instead of lazily. The Pudding (a leader in scrollytelling) specifically warns that mobile scroll behavior differs fundamentally from desktop.

**How to avoid:**
- Mobile-first development, literally. Build and test on a throttled mobile device (Chrome DevTools device emulation with CPU throttling at 4x slowdown) before touching desktop styles.
- Use Intersection Observer API for scroll triggers, never scroll event listeners. Intersection Observer is battery-efficient and doesn't fire on every pixel of scroll.
- Limit animations to opacity and transform only (these are GPU-composited). No animating width, height, margin, or layout properties.
- Implement `prefers-reduced-motion` media query from the start -- some users have motion sensitivity, and it is an accessibility requirement.
- Lazy-load chart components: only render a chart when its container scrolls into view.
- Set a performance budget: Lighthouse mobile score must stay above 85.

**Warning signs:**
- Lighthouse mobile performance score drops below 80
- Total Blocking Time exceeds 300ms on mobile
- Largest Contentful Paint exceeds 2.5s
- Any scroll event listener in the codebase (should be Intersection Observer instead)

**Phase to address:**
Phase 2-3 (template design and data visualization). Establish the performance budget and mobile-first constraint before building any scroll interactions. Test every visual component on throttled mobile before merging.

---

### Pitfall 4: SB7 Narrative Drift -- The Hero/Guide Roles Get Confused

**What goes wrong:**
The page subtly shifts from "you (homeowner) are the hero" to "Live AZ Co is impressive." Sections start showcasing Josh's market knowledge rather than the homeowner's journey. The word "just" creeps in ("Your home has just appreciated..."). Future scenarios read like sales pitches ("Call us to discuss selling") rather than empowering possibilities ("Here's what you could do with your equity"). The homeowner feels marketed-to rather than empowered.

**Why it happens:**
SB7 discipline is hard to maintain across an entire page, especially when generating narrative programmatically. Template strings naturally drift toward brand-centric language. Developers writing copy placeholders default to marketing-speak. And when future scenarios involve actions that benefit the agent (selling, refinancing), the temptation to pitch is strong.

**How to avoid:**
- Create an SB7 compliance checklist that every page narrative is tested against before publishing:
  - Does every section open by naming a problem or promise relevant to the homeowner?
  - Is the homeowner the subject of the first sentence in every section?
  - Does Live AZ Co appear only in the "guide" position (offering a plan, showing authority)?
  - Is the word "just" absent? (SB7 rule from existing CMA project)
  - Do future scenarios end with possibility language, not sales CTAs?
- Build the narrative templates with locked structural elements: `[Homeowner name], since [date]...` openers that cannot be overridden.
- Keep the CTA to a single, soft guide-position element at the very end: "When you're ready to explore any of these paths, Josh is here."

**Warning signs:**
- More than 2 mentions of "Live AZ Co" or "Josh" in any single section
- Any section where the first sentence subject is the brand, not the homeowner
- Future scenario sections that end with "Contact us" rather than possibility statements
- The word "just" appears anywhere

**Phase to address:**
Phase 2 (narrative template design). The SB7 structure must be baked into the template as constraints, not guidelines. Review the existing CMA project's `SB7_Project_Instructions_Update.md` rules and encode them as template rules.

---

### Pitfall 5: Production Bottleneck -- "Under 5 Minutes" Becomes 20 Minutes

**What goes wrong:**
Creating a new homeowner page requires: looking up the address, finding comparable sales data, calculating appreciation, entering data into a form, reviewing the generated narrative, fixing any data issues, and publishing. What was designed as a 5-minute workflow becomes 15-20 minutes because the input process has too many fields, data validation catches errors that require re-entry, or the preview/publish cycle requires multiple rounds. At 20-30 pages per month, a 20-minute workflow eats 7-10 hours/month instead of the target 2.5-5 hours.

**Why it happens:**
Developers optimize for output quality (more fields = more personalization = better page) without optimizing for input speed. Every "nice to have" field adds 30 seconds to input time. Preview workflows that require a full page reload add minutes per page. And there is no clear boundary between "required for narrative" and "optional enhancement" data.

**How to avoid:**
- Define a minimum viable input set and lock it: address, purchase price, purchase date, client first name. Everything else is derived or optional.
- Auto-calculate everything possible: appreciation from purchase price + estimated current value, ownership duration from purchase date, equity from simple math.
- Build a streamlined admin input form with exactly the required fields visible by default. Hide optional fields behind an "Advanced" toggle.
- One-click publish: preview should be inline (not a separate page load), and publishing should be a single button.
- Time the workflow during development. If creating a test page takes more than 5 minutes, the input process needs simplification.
- Consider a batch creation mode for when Josh wants to create 10+ pages in a session.

**Warning signs:**
- The input form has more than 8 visible fields
- Creating a page requires switching between tabs/tools to look up data
- The preview requires a full page navigation away from the input form
- Josh stops creating pages because "it takes too long"

**Phase to address:**
Phase 3 (admin/input workflow). The admin experience is as critical as the homeowner-facing page. Design the input form with a stopwatch running.

---

### Pitfall 6: URL and Page Management Chaos at 200+ Pages

**What goes wrong:**
After 6 months of creating pages, there are 150+ URLs with no consistent naming scheme, no way to find a specific homeowner's page, no way to know which pages have stale data, and no way to unpublish a page for a homeowner who has sold and moved. A former client's page shows "Your home has appreciated 30%!" but they sold it two months ago. Or worse, two homeowners at the same address over time (previous owner and new buyer) have conflicting pages.

**Why it happens:**
URL management feels trivial when you have 5 pages. It becomes a crisis at 200. Developers build the page generation system without building the page management system. There is no concept of page lifecycle (active, stale, archived), no dashboard for Josh to see all pages at a glance, and no search/filter capability.

**How to avoid:**
- Design the URL scheme upfront: `/journey/[slug]` where slug is deterministic (e.g., `firstname-lastname-street-zip` or a short hash).
- Build a simple admin dashboard from Phase 1 that lists all pages with: homeowner name, address, creation date, last updated, status (active/archived).
- Add a "stale data" indicator: if a page hasn't been updated in 6+ months, flag it in the admin view.
- Build archive/unpublish capability before you need it. A page for a homeowner who sold should be archivable, not deleted (they may buy again).
- Prevent duplicate slugs at the database level with a unique constraint.
- Plan for the "same address, different owner" scenario: slugs should include homeowner identity, not just address.

**Warning signs:**
- No admin view exists for listing all created pages
- URLs are auto-generated with random strings instead of meaningful slugs
- No way to search pages by homeowner name or address
- No concept of page status (everything is permanently "live")

**Phase to address:**
Phase 1 (data model) for the schema and URL design. Phase 3 (admin workflow) for the management dashboard. The data model must support lifecycle states from day one even if the UI comes later.

---

### Pitfall 7: Data Staleness Without a Refresh Strategy

**What goes wrong:**
A homeowner receives their journey page in March 2026. They love it and bookmark it. They revisit it in September 2026. The page still shows March 2026 data. The appreciation numbers are 6 months stale, the market context is outdated, and the future scenarios use old projections. The page now undermines trust rather than building it. Or conversely, Josh manually updates some pages but not others, creating an inconsistent experience across his client base.

**Why it happens:**
The project scope says "moment-in-time touchpoint" (not recurring updates like Homebot), which is correct for v1. But developers don't build any mechanism for eventually refreshing data, and they don't clearly communicate to the homeowner that this is a snapshot. The homeowner expects the page to stay current because that is what every other real estate tool does.

**How to avoid:**
- Display the data date prominently: "Your Journey Map -- as of March 2026" near the top of the page.
- In the data model, store a `created_at` and `data_as_of` date. These are different (you might create a page today with data from last week).
- Design the admin workflow so that "refreshing" a page is updating the data fields and clicking save, not creating a new page. The URL stays the same.
- For v1, accept manual refresh and document it. But design the data model so that automated refresh could be added later without schema changes.
- Consider adding a footer note: "This snapshot was prepared for you on [date]. Want an updated view? Text Josh."

**Warning signs:**
- No date displayed anywhere on the homeowner-facing page
- The data model has no timestamp fields for when values were captured
- Updating a page's data requires creating a new URL (breaking the old bookmark)
- No way to identify which pages have the oldest data

**Phase to address:**
Phase 1 (data model) for timestamp fields. Phase 2 (template) for displaying the date. Phase 3 (admin) for the refresh workflow.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoding Phoenix market data (avg appreciation rates, rental yields) | Faster to build, no API needed | Cannot expand to other markets; numbers go stale silently | v1 only, with a clear data constants file that can be swapped later |
| Single-file page component (all sections in one component) | Faster initial development | Cannot A/B test sections, cannot reorder narrative, hard to maintain at 1000+ lines | Never -- section components should be separate from day one |
| No input validation on admin form | Faster form development | Bad data produces embarrassing pages (negative appreciation, future purchase dates, $0 home values) | Never -- basic validation is cheap and prevents visible errors |
| Storing all page data in a single JSON blob | Simple data model, flexible schema | Cannot query across pages ("show me all pages with >$50K equity"), cannot update individual fields | v1 only, but migrate to structured fields before 50 pages |
| Skip OpenGraph/social meta tags | Faster page development | When homeowner shares the link on social media, it shows a generic preview instead of a branded card with their address | Never -- this is a 30-minute task that dramatically improves shareability |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Supabase (page storage) | Exposing all page data via public API without Row Level Security | Use RLS policies: public can read individual pages by slug, only authenticated admin can create/update/list all pages |
| Google Fonts (Source Serif 4, DM Sans) | Loading all font weights upfront, blocking page render | Use `font-display: swap`, load only weights 400 and 700, preconnect to fonts.googleapis.com |
| Image CDN (hero photos, maps) | Serving full-resolution images on mobile | Use responsive images with `srcset`, serve WebP with JPEG fallback, max 200KB per image on mobile |
| Chart library (Recharts/Chart.js) | Importing the entire library for 2-3 simple charts | Tree-shake or use lightweight alternatives (e.g., custom SVG for simple line charts, CSS for bar charts) |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Loading all chart data inline in the HTML | Slow initial page load, large HTML document size | Lazy-load chart data as JSON fetched when chart scrolls into view | At 5+ charts per page with real data |
| CSS animations on scroll without GPU compositing | Janky scrolling on mid-range phones | Use only `transform` and `opacity` for animations; add `will-change` sparingly | Immediately on phones older than 2 years |
| No image optimization pipeline | Hero images are 2-5MB each, page loads in 8+ seconds on mobile | Resize to max 1200px width, compress to WebP, lazy-load below-fold images | At first page view on a 4G connection |
| Server-side rendering every page on request | Cold starts on Cloudflare/Vercel, slow response for first visitor | Use static generation (SSG) for all pages -- content changes only when Josh updates data | At 50+ concurrent visitors across all pages |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Guessable URL slugs (sequential IDs, `/journey/1`, `/journey/2`) | Anyone can enumerate and view all homeowner pages, exposing names, addresses, and financial data | Use slugs with enough entropy to prevent enumeration but still be readable: `firstname-lastname-[4-char-hash]` |
| No robots.txt or noindex on journey pages | Google indexes personalized pages, homeowner data appears in search results | Add `<meta name="robots" content="noindex, nofollow">` to all journey pages; add to robots.txt |
| Admin panel accessible without authentication | Anyone who finds the admin URL can create/edit/delete pages | Protect admin routes with authentication (at minimum, password-based like the Idea Pipeline project) |
| Storing sensitive financial data in page source | Exact purchase prices, equity amounts visible in page source/API responses | This data is inherently visible on the page -- the protection is preventing enumeration and search indexing, not hiding data from the page viewer |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Overwhelming the homeowner with too many numbers on first scroll | Cognitive overload; they close the tab before reaching the narrative payoff | Lead with the emotional hook (story of their home), introduce numbers gradually within narrative context |
| Charts without plain-language interpretation | Homeowner sees a line going up but doesn't know if that's good, average, or exceptional | Every chart must have a one-sentence takeaway below it: "Your home has outpaced the Phoenix metro average by 4%." |
| Future scenarios presented as predictions | Homeowner makes financial decisions based on projections shown as facts, then blames Josh when reality differs | Label all projections: "If current trends continue..." and "This is an illustration, not a guarantee." Include a disclaimer. |
| Generic narrative that doesn't feel personal | Homeowner can tell this is a template with their name inserted | Reference specific details: neighborhood name, purchase season ("the spring you bought"), ownership milestone ("approaching your 3rd year") |
| No clear "what to do next" after reading | Homeowner finishes the page, feels good, but doesn't take any action | End with a single, soft CTA in guide position: "When you're ready to explore any of these paths, Josh is here to help." Include phone number. |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Appreciation section:** Often missing source attribution and date -- verify every value has an "as of [date]" label
- [ ] **Mobile layout:** Often tested only in Chrome DevTools -- verify on a real iPhone via text message link (the actual delivery method)
- [ ] **Social sharing:** Often missing OpenGraph tags -- verify that sharing the URL on iMessage/Facebook shows a branded preview card, not a blank link
- [ ] **Future scenarios:** Often missing disclaimer language -- verify projections include "illustration, not guarantee" language
- [ ] **Page load on 4G:** Often only tested on WiFi -- verify page loads under 3 seconds on throttled 4G connection
- [ ] **Print/screenshot appearance:** Homeowners will screenshot sections to share -- verify key sections look good as static screenshots (no half-rendered animations)
- [ ] **Long names/addresses:** Often breaks layout -- verify with "Christopher & Alexandra Wellington-Bartholomew" and "12345 East Camelback Mountain Estates Boulevard, Unit 2401"
- [ ] **Page at 12+ months old:** Often no visual indicator of data age -- verify stale pages clearly show their snapshot date
- [ ] **SB7 compliance:** Often drifts during development -- verify word "just" is absent, homeowner is hero in every section, brand appears only as guide

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Wrong appreciation data sent to homeowner | MEDIUM | Immediately update the page data (URL stays the same), text the homeowner: "Updated your page with the latest numbers -- take a fresh look." Frame the update as attentiveness, not error correction. |
| Page feels like Homebot/generic report | HIGH | Requires template redesign. Cannot be fixed with content changes alone -- the structural layout must change from dashboard to narrative flow. Do this before creating more pages. |
| Mobile scroll performance is broken | MEDIUM | Disable animations first (ship content-only), then reintroduce animations one at a time with performance testing after each. |
| SB7 drift across many pages | LOW | Run the SB7 compliance checklist against all existing pages. Fix template strings centrally -- all pages update because they share the template. |
| URL/page management is chaotic | HIGH | Retroactively cataloging 100+ pages is painful. Export all pages to a spreadsheet, establish the naming convention, and migrate URLs (set up redirects from old URLs). Much cheaper to prevent than to fix. |
| Data staleness across many pages | MEDIUM | Add "data as of" date to the template (all pages update). Create an admin view sorted by oldest data. Batch-refresh the stalest pages first. |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Inaccurate appreciation data | Phase 1 (Data Model) | Data schema includes source, date, and range fields; input form has validation |
| Automated report feel | Phase 2 (Template Design) | Show page to 3 non-technical people; none say "like Homebot" |
| Mobile scroll performance | Phase 2-3 (Template + Visualization) | Lighthouse mobile score > 85; tested on real phone via text link |
| SB7 narrative drift | Phase 2 (Template Design) | SB7 checklist passes on every section; "just" grep returns 0 results |
| Production bottleneck (>5 min) | Phase 3 (Admin Workflow) | Timed test: create a page from scratch in under 5 minutes |
| URL/page management chaos | Phase 1 + Phase 3 (Schema + Admin) | Admin dashboard lists all pages with search, filter, status |
| Data staleness | Phase 1 + Phase 2 + Phase 3 (Schema + Display + Refresh) | Every page shows data date; admin view shows pages sorted by staleness |

## Sources

- [Zillow Zestimate Accuracy](https://www.zillow.com/learn/influencing-your-zestimate/) -- 7% median error on off-market homes
- [Zestimate Accuracy Analysis](https://www.eatonrealty.com/blog/selling/how-accurate-zillows-zestimate) -- notable errors including Zillow founder's own home
- [Homebot Analytics Approach](https://amplitude.com/blog/homebot-analytics-experiment) -- engagement patterns and A/B testing lessons
- [Homebot Client Engagement](https://homebot.ai/blog/unlocking-homeownership-engagement-how-to-create-repeat-clients) -- 80% retention loss without consistent engagement
- [Homebot Review](https://theclose.com/homebot-reviews/) -- pricing, pros and cons for agents
- [Scrollytelling Best Practices (Pudding)](https://pudding.cool/process/responsive-scrollytelling/) -- mobile-specific scrollytelling patterns
- [Complete Scrollytelling Guide 2025](https://ui-deploy.com/blog/complete-scrollytelling-guide-how-to-create-interactive-web-narratives-2025) -- performance budgets and mobile pitfalls
- [StoryBrand SB7 Implementation Tips](https://hughesintegrated.com/tips-storybrand-agency/) -- 105 tips including common mistakes
- [StoryBrand Framework Guide](https://www.gravityglobal.com/blog/complete-guide-storybrand-framework) -- hero/guide role confusion
- [Next.js SSG Best Practices](https://nextjs.org/docs/pages/building-your-application/rendering/static-site-generation) -- static generation for page-heavy sites
- [ISR with Next.js (Smashing Magazine)](https://www.smashingmagazine.com/2021/04/incremental-static-regeneration-nextjs/) -- incremental regeneration patterns
- [Mobile Data Visualization (Smashing Magazine)](https://www.smashingmagazine.com/2020/05/data-visualization-mobile-web-experience/) -- chart design for small screens

---
*Pitfalls research for: Personalized Homeowner Journey Pages (Live AZ Co)*
*Researched: 2026-03-11*
