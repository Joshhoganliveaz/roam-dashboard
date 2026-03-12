# Phase 3: Narrative Experience - Research

**Researched:** 2026-03-12
**Domain:** Scroll-triggered animations, adaptive narrative by ownership duration, future scenario UI, milestone markers, CTA design
**Confidence:** HIGH

## Summary

Phase 3 layers animation, adaptive narrative, scenario cards, milestone callouts, and a closing CTA on top of the existing branded homeowner page built in Phases 1-2. The existing codebase already has: (1) all calculation functions including three scenario projections (holdAndRent, moveUp, equityPlay), (2) a client-side HomeownerPage orchestrator with 9 section components, (3) SectionWrapper for alternating backgrounds, (4) brand CSS custom properties, and (5) an AppreciationChart using Recharts. No new calculations are needed -- all data is already computed by `computeHomeownerMetrics()`.

The primary technical challenge is adding scroll-triggered animations (section reveals, number counters, chart draw-in) without degrading mobile performance. Motion (formerly Framer Motion) v12.x provides `whileInView` with `viewport={{ once: true }}` which maps directly to the "animate once, stay visible" requirement. For number counters, a lightweight custom hook using `requestAnimationFrame` with easing avoids adding a dependency (react-countup) for a simple feature. The Recharts chart can be triggered via `isAnimationActive` controlled by an Intersection Observer or Motion's `useInView`.

**Primary recommendation:** Use `motion` (v12.x) for all scroll-triggered animations. Add animation logic to SectionWrapper for section-level triggers and create a useCountUp hook for hero number counters. Build three new components (ScenarioCards, MilestoneCallout, CTASection) and add a `getNarrativeVoice()` utility for ownership-duration-based copy selection.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Cinematic & theatrical animation feel with staggered element entrances
- Hybrid stagger: key data sections get element-level stagger (SB7 intro -> hero number -> supporting copy, 200-300ms delays); quieter sections animate as a unit
- Hero number counters use ease-in acceleration (start slow, accelerate, land with impact)
- Animate once only -- sections stay visible after first reveal, no replay on scroll-back
- Three narrative voices: Recent (under 2 years), Mid-term (2-7 years), Long-term (7+ years)
- Copy changes only -- same sections, same order, same structure for all voices
- SB7 intro lines and supporting narrative text adapt per voice
- Card cluster approach for scenarios -- grouped in one section after Net Proceeds
- Three cards: Hold & Rent, Move-Up, Equity Play with tap-to-expand interaction
- Collapsed: hero metric + one-line implication; Expanded: 2-3 supporting details
- Deliberately stops at information cliff -- next question requires Josh
- Cards are NOT full calculators
- Inline milestone callouts within relevant sections (no side rail)
- 2-3 milestones max per page, dynamically selected based on most impressive for each homeowner
- Branded closing section (canyon or olive background) set apart from data sections
- Josh + Jacqui photos side by side in CTA
- Single `sms:` text link to Josh's number
- NOT a sales pitch -- natural conclusion to the narrative

### Claude's Discretion
- Animation library choice (Framer Motion vs lighter alternatives)
- Exact SB7 copy for all three narrative voices across all sections
- Chart draw-in animation approach (line trace vs area grow)
- Milestone selection logic and threshold values
- Milestone visual treatment (badge, tag, highlighted callout)
- CTA photo styling and layout details
- Exact scenario card expanded content and copy
- Section transition timing and easing curves
- Mobile animation performance optimization

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| NARR-03 | Data reveals animate on scroll (counters count up, charts draw themselves) | Motion `whileInView` + `viewport={{ once: true }}` for section reveals; custom `useCountUp` hook with ease-in easing for hero numbers; Recharts `isAnimationActive` controlled by `useInView` for chart draw |
| NARR-04 | Narrative adapts by ownership duration | `getNarrativeVoice()` utility returns "recent"/"midterm"/"longterm" from `ownershipMonths`; copy map object with voice-keyed SB7 text for each section |
| NARR-05 | Timeline includes milestone markers | `selectMilestones()` utility picks 2-3 most impressive from candidate list; MilestoneCallout component renders inline within relevant sections |
| NARR-08 | Soft CTA to contact Josh | CTASection component with branded background, team photos, warm narrative copy, single `sms:` link |
| SCEN-01 | Hold & Rent projection | Data already computed in `metrics.holdAndRent`; render as expandable scenario card showing breakeven timeline, rental income, cash flow |
| SCEN-02 | Move-Up projection | Data already computed in `metrics.moveUp`; render as expandable card showing down payment, price range, estimated payment |
| SCEN-03 | Equity Play projection | Data already computed in `metrics.equityPlay`; render as expandable card showing available equity, HELOC amount, investment leverage |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| motion | ^12.35 | Scroll-triggered animations, section reveals, staggered entrances | De facto React animation library; `whileInView` maps directly to requirements; already referenced in project dependencies |
| Recharts | ^3.8 | Appreciation chart (already installed) | Already in project; `isAnimationActive` + `animationBegin` provide chart draw control |
| date-fns | ^4.1 | Ownership duration calculation | Already installed; `differenceInMonths`, `differenceInYears`, `format` |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None new | - | - | All other needs met by existing stack or custom hooks |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| motion | CSS `@scroll-timeline` / Intersection Observer only | CSS-only is lighter but lacks stagger orchestration and spring physics; Motion provides declarative React API that matches the cinematic feel requirement |
| Custom useCountUp hook | react-countup npm package | react-countup adds a dependency for something achievable in ~30 lines; custom hook integrates with Motion's `useInView` for scroll-triggered start |
| motion AnimateNumber | Custom counter | AnimateNumber requires paid Motion+ subscription; custom hook is free and sufficient |

**Installation:**
```bash
npm install motion
```

No other new packages needed.

## Architecture Patterns

### New Components
```
src/
├── components/homeowner/
│   ├── AnimatedSection.tsx    # Wraps SectionWrapper with motion.div for scroll triggers
│   ├── CountUpNumber.tsx      # Animated number counter component
│   ├── ScenarioCards.tsx      # Card cluster with 3 expandable scenario cards
│   ├── ScenarioCard.tsx       # Individual expandable card (collapsed/expanded)
│   ├── MilestoneCallout.tsx   # Inline milestone badge/tag
│   └── CTASection.tsx         # Branded closing section with team photos + sms link
├── lib/
│   ├── narrative.ts           # getNarrativeVoice() + copy maps for 3 voices
│   ├── milestones.ts          # selectMilestones() logic
│   └── hooks/
│       └── useCountUp.ts      # requestAnimationFrame counter with easing
```

### Pattern 1: Animated Section Wrapper
**What:** Wrap existing SectionWrapper with Motion's `motion.div` for scroll-triggered reveals. Key data sections get child-level stagger; quieter sections animate as a unit.
**When to use:** Every section on the homeowner page.
**Example:**
```typescript
"use client";
import { motion } from "motion/react";

interface AnimatedSectionProps {
  children: React.ReactNode;
  stagger?: boolean; // true for key data sections, false for unit animation
}

export function AnimatedSection({ children, stagger = false }: AnimatedSectionProps) {
  if (stagger) {
    return (
      <motion.div
        initial="hidden"
        whileInView="visible"
        viewport={{ once: true, margin: "-100px" }}
        variants={{
          hidden: {},
          visible: { transition: { staggerChildren: 0.25 } },
        }}
      >
        {children}
      </motion.div>
    );
  }

  return (
    <motion.div
      initial={{ opacity: 0, y: 30 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, margin: "-100px" }}
      transition={{ duration: 0.6, ease: "easeOut" }}
    >
      {children}
    </motion.div>
  );
}
```

### Pattern 2: Staggered Child Elements
**What:** Wrap individual elements (SB7 intro, hero number, supporting copy) in `motion.div` with shared variants for staggered entrance within a section.
**When to use:** Key data sections (equity, earnings, net proceeds).
**Example:**
```typescript
const childVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.5, ease: "easeOut" } },
};

// Inside a section component:
<AnimatedSection stagger>
  <motion.p variants={childVariants} className="text-canyon ...">
    That means you have built...
  </motion.p>
  <motion.h2 variants={childVariants} className="text-charcoal ...">
    <CountUpNumber value={equity} format="currency" />
  </motion.h2>
  <motion.p variants={childVariants} className="text-charcoal/70 ...">
    in equity since you first opened the front door.
  </motion.p>
</AnimatedSection>
```

### Pattern 3: Count-Up Hook with Ease-In
**What:** Custom hook using `requestAnimationFrame` to animate from 0 to target value with ease-in acceleration (start slow, speed up, land on target).
**When to use:** Hero numbers (equity, earnings, net proceeds, current value).
**Example:**
```typescript
"use client";
import { useEffect, useState, useRef } from "react";

function easeInQuad(t: number): number {
  return t * t; // ease-in: starts slow, accelerates
}

export function useCountUp(
  target: number,
  isInView: boolean,
  duration = 1500
): number {
  const [value, setValue] = useState(0);
  const hasAnimated = useRef(false);

  useEffect(() => {
    if (!isInView || hasAnimated.current) return;
    hasAnimated.current = true;

    let startTime: number;
    const animate = (timestamp: number) => {
      if (!startTime) startTime = timestamp;
      const elapsed = timestamp - startTime;
      const progress = Math.min(elapsed / duration, 1);
      const easedProgress = easeInQuad(progress);
      setValue(Math.round(target * easedProgress));

      if (progress < 1) {
        requestAnimationFrame(animate);
      } else {
        setValue(target); // ensure exact final value
      }
    };
    requestAnimationFrame(animate);
  }, [isInView, target, duration]);

  return value;
}
```

### Pattern 4: Narrative Voice Selection
**What:** Pure function that returns a voice key based on ownership months, plus a copy map with voice-specific SB7 text.
**When to use:** Every section that has narrative text.
**Example:**
```typescript
// src/lib/narrative.ts
export type NarrativeVoice = "recent" | "midterm" | "longterm";

export function getNarrativeVoice(ownershipMonths: number): NarrativeVoice {
  if (ownershipMonths < 24) return "recent";
  if (ownershipMonths < 84) return "midterm";
  return "longterm";
}

// Copy maps per section
export const equityCopy: Record<NarrativeVoice, { intro: string; supporting: string }> = {
  recent: {
    intro: "Already, in this short time, you have built...",
    supporting: "And this is where your story gets exciting.",
  },
  midterm: {
    intro: "Over these years, you have quietly built...",
    supporting: "in equity -- wealth that is working for you.",
  },
  longterm: {
    intro: "Look at what you have built...",
    supporting: "in equity. That is the result of years of smart decisions.",
  },
};
```

### Pattern 5: Expandable Scenario Card
**What:** Tap-to-expand card showing collapsed hero metric and expanded supporting details. Uses `useState` for toggle, `motion.div` with `AnimatePresence` for smooth expand/collapse.
**When to use:** Each of the three scenario cards.
**Example:**
```typescript
"use client";
import { useState } from "react";
import { motion, AnimatePresence } from "motion/react";

interface ScenarioCardProps {
  title: string;
  heroMetric: string;
  implication: string;
  details: { label: string; value: string }[];
  cliffQuestion: string;
}

export function ScenarioCard({ title, heroMetric, implication, details, cliffQuestion }: ScenarioCardProps) {
  const [expanded, setExpanded] = useState(false);

  return (
    <button
      onClick={() => setExpanded(!expanded)}
      className="w-full text-left bg-white rounded-xl p-6 shadow-sm border border-charcoal/10"
    >
      <h3 className="font-heading text-lg text-charcoal font-semibold">{title}</h3>
      <p className="font-heading text-2xl text-olive font-bold mt-2">{heroMetric}</p>
      <p className="text-charcoal/60 font-body text-sm mt-1">{implication}</p>

      <AnimatePresence>
        {expanded && (
          <motion.div
            initial={{ height: 0, opacity: 0 }}
            animate={{ height: "auto", opacity: 1 }}
            exit={{ height: 0, opacity: 0 }}
            transition={{ duration: 0.3 }}
            className="overflow-hidden"
          >
            <div className="mt-4 pt-4 border-t border-charcoal/10 space-y-2">
              {details.map((d) => (
                <div key={d.label} className="flex justify-between text-sm">
                  <span className="text-charcoal/70">{d.label}</span>
                  <span className="text-charcoal font-medium">{d.value}</span>
                </div>
              ))}
              <p className="text-canyon font-body text-sm italic mt-3">{cliffQuestion}</p>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </button>
  );
}
```

### Pattern 6: Chart Draw-in via Intersection Observer
**What:** Control Recharts `isAnimationActive` with `useInView` from Motion so the chart only animates when scrolled into view.
**When to use:** AppreciationChart component.
**Example:**
```typescript
"use client";
import { useInView } from "motion/react";
import { useRef } from "react";
import { AreaChart, Area, /* ... */ } from "recharts";

export function AppreciationChart({ data }: { data: ChartDataPoint[] }) {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true, margin: "-100px" });

  return (
    <div ref={ref}>
      <ResponsiveContainer width="100%" height={300}>
        <AreaChart data={data}>
          <Area
            type="monotone"
            dataKey="value"
            isAnimationActive={isInView}
            animationDuration={1500}
            animationEasing="ease-in-out"
            /* ... gradient fill ... */
          />
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
}
```

### Anti-Patterns to Avoid
- **Animating on every scroll pass:** Always set `viewport={{ once: true }}`. Replaying animations on scroll-back feels janky and was explicitly rejected.
- **Heavy spring physics on mobile:** Keep transitions simple (duration + easeOut). Avoid complex spring configs that cause frame drops on mid-range phones.
- **Separate animation state per section:** Use Motion's declarative `whileInView` prop, not manual Intersection Observer state in each component. Motion handles cleanup and optimization.
- **Building calculators in scenario cards:** Cards present pre-computed data with a deliberate information cliff. No inputs, no recalculation, no interactivity beyond expand/collapse.
- **Over-animating:** Not every element needs motion. The quiet sections (personal note, purchase story) animate as a unit. Reserve stagger for high-impact data reveals.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Scroll-triggered reveals | Custom Intersection Observer + state management per section | Motion `whileInView` + `viewport={{ once: true }}` | Motion handles observer lifecycle, cleanup, and provides stagger orchestration |
| Expand/collapse animation | CSS height transition with fixed values | Motion `AnimatePresence` + `height: "auto"` | CSS cannot transition to `height: auto`; Motion measures and animates dynamically |
| Scenario calculations | New calculation functions | Existing `computeHomeownerMetrics()` | holdAndRent, moveUp, equityPlay already computed; see `HomeownerMetrics` interface |
| Number formatting | Custom formatters | Existing `formatCurrency()`, `formatPercent()` | Already built and hardened in Phase 2.1 |
| Ownership duration | Manual date math | `date-fns differenceInMonths()` + existing `metrics.ownershipMonths` | Already computed in metrics |

**Key insight:** Phase 3 adds ZERO new calculations. All data is already in `HomeownerMetrics`. This phase is purely presentation layer: animation, adaptive copy, new UI components (scenario cards, milestones, CTA), and narrative voice logic.

## Common Pitfalls

### Pitfall 1: Mobile Animation Performance
**What goes wrong:** Animations feel janky on iPhones, especially mid-range models opening via text message.
**Why it happens:** Too many simultaneous animations, complex spring physics, or animating layout-triggering properties (width, height, top).
**How to avoid:** Animate only `opacity` and `transform` (these are GPU-composited). Keep stagger groups small (3-4 children max). Use `will-change: transform` sparingly. Test on a real phone, not just Chrome DevTools.
**Warning signs:** Dropped frames, stuttery scroll, visible repaints in browser performance panel.

### Pitfall 2: CountUp Triggering Before Visible
**What goes wrong:** Number counters start counting before the user can see them, so the animation is already done when they scroll to it.
**Why it happens:** Counter starts on component mount instead of on scroll-into-view.
**How to avoid:** Wire `useCountUp` to `useInView` -- the `isInView` boolean controls when the counter starts. The `hasAnimated` ref prevents replay.
**Warning signs:** Hero numbers show their final value with no visible animation.

### Pitfall 3: Recharts Re-Animation on State Change
**What goes wrong:** The chart re-draws its animation every time a parent component re-renders (e.g., when a scenario card is expanded).
**Why it happens:** Recharts replays animation when `isAnimationActive` toggles or when the component remounts.
**How to avoid:** Use a stable ref-based `isInView` state that only flips once (`once: true`). Memoize the chart component or its data if parent re-renders are frequent.
**Warning signs:** Chart visibly re-draws when interacting with unrelated UI elements.

### Pitfall 4: Narrative Voice Mismatch
**What goes wrong:** A homeowner who purchased 23 months ago gets "long-term owner" language, or thresholds feel arbitrary.
**Why it happens:** Boundary conditions not matching real-world expectations; `ownershipMonths` calculation off by one.
**How to avoid:** Use `differenceInMonths` from date-fns (already used in calculations). Test with dates near boundaries (23 months, 24 months, 83 months, 84 months). Keep thresholds simple: <24 = recent, 24-83 = midterm, 84+ = longterm.
**Warning signs:** Copy tone feels wrong for a specific homeowner's story.

### Pitfall 5: Scenario Card Content Too Sparse or Too Dense
**What goes wrong:** Expanded cards either feel empty (not worth tapping) or overwhelming (defeats the "information cliff" purpose).
**Why it happens:** Not calibrating the sweet spot between "enough to excite" and "not enough to decide."
**How to avoid:** Each expanded card should have exactly 2-3 data rows plus one italic cliff question. The cliff question is the most important element -- it creates the call-Josh moment.
**Warning signs:** Homeowner feels like they have all the info they need (no call), or feels confused (no call either).

### Pitfall 6: CTA Feeling Like a Sales Pitch
**What goes wrong:** The closing section breaks the narrative tone and feels like a generic real estate ad.
**Why it happens:** Switching from warm storytelling voice to "Contact us today!" marketing speak.
**How to avoid:** The CTA copy should read like a natural ending to a conversation, not an ad. Use first-person from Josh. The `sms:` link should feel like texting a friend, not filling out a lead form.
**Warning signs:** Copy uses phrases like "Don't wait!", "Act now!", "Schedule your consultation!"

## Code Examples

### Milestone Selection Logic
```typescript
// src/lib/milestones.ts
import { formatCurrency } from "@/lib/format";

interface Milestone {
  type: string;
  label: string;
  section: string; // which section to render in
}

export function selectMilestones(
  ownershipMonths: number,
  purchaseDate: string,
  equityGained: number,
  appreciationPercent: number
): Milestone[] {
  const candidates: (Milestone & { impressiveness: number })[] = [];

  // Purchase anniversary (whole years only)
  const years = Math.floor(ownershipMonths / 12);
  if (years >= 1) {
    candidates.push({
      type: "anniversary",
      label: `${years}-year homeowner`,
      section: "purchase-story",
      impressiveness: years,
    });
  }

  // Equity thresholds
  const equityThresholds = [50_000, 100_000, 150_000, 200_000, 250_000, 500_000];
  const crossedThreshold = equityThresholds.filter((t) => equityGained >= t).pop();
  if (crossedThreshold) {
    candidates.push({
      type: "equity",
      label: `${formatCurrency(crossedThreshold)}+ in equity`,
      section: "equity-gained",
      impressiveness: crossedThreshold / 50_000,
    });
  }

  // Appreciation percentage
  if (appreciationPercent >= 0.20) {
    candidates.push({
      type: "appreciation",
      label: `${Math.round(appreciationPercent * 100)}% appreciation`,
      section: "current-value",
      impressiveness: appreciationPercent * 10,
    });
  }

  // Sort by impressiveness, take top 2-3
  return candidates
    .sort((a, b) => b.impressiveness - a.impressiveness)
    .slice(0, 3)
    .map(({ impressiveness, ...milestone }) => milestone);
}
```

### Scenario Card Data Mapping
```typescript
// Map HomeownerMetrics scenario data to ScenarioCard props
function buildScenarioCards(metrics: HomeownerMetrics) {
  return [
    {
      title: "Hold & Rent",
      heroMetric: formatCurrency(metrics.holdAndRent.monthlyRent) + "/mo",
      implication: metrics.holdAndRent.cashFlow > 0
        ? "Your rental income would already cover the mortgage."
        : `${formatCurrency(Math.abs(metrics.holdAndRent.cashFlow))}/mo gap to cover.`,
      details: [
        { label: "Estimated rental income", value: formatCurrency(metrics.holdAndRent.monthlyRent) + "/mo" },
        { label: "Current mortgage", value: formatCurrency(metrics.holdAndRent.monthlyMortgage) + "/mo" },
        { label: "Monthly cash flow", value: formatCurrency(metrics.holdAndRent.cashFlow) },
      ],
      cliffQuestion: "What would my actual numbers look like?",
    },
    {
      title: "Move Up",
      heroMetric: formatCurrency(metrics.moveUp.purchasingPower),
      implication: `Your equity unlocks homes up to ${formatCurrency(metrics.moveUp.purchasingPower)}.`,
      details: [
        { label: "Your net proceeds", value: formatCurrency(metrics.moveUp.netProceeds) },
        { label: "As a 20% down payment", value: formatCurrency(metrics.moveUp.downPaymentAt20Pct) },
        { label: "Purchasing power", value: formatCurrency(metrics.moveUp.purchasingPower) },
      ],
      cliffQuestion: "What could I actually get in the areas I want?",
    },
    {
      title: "Equity Play",
      heroMetric: formatCurrency(metrics.equityPlay.helocAt80Pct),
      implication: "Potential HELOC to leverage into an investment property.",
      details: [
        { label: "Available equity", value: formatCurrency(metrics.equityPlay.availableEquity) },
        { label: "HELOC at 80% LTV", value: formatCurrency(metrics.equityPlay.helocAt80Pct) },
        { label: "Investment leverage", value: formatCurrency(metrics.equityPlay.investmentLeverage) },
      ],
      cliffQuestion: "Is this actually a good move for my situation?",
    },
  ];
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| framer-motion package | motion package (import from "motion/react") | Late 2024 | Same API, new package name; install `motion` not `framer-motion` |
| Manual Intersection Observer + useState | Motion `whileInView` / `useInView` | Framer Motion v7+ | Declarative, handles cleanup, integrates with animation orchestration |
| react-countup library | Custom hook or Motion AnimateNumber | 2025+ | Motion's AnimateNumber is paywalled (Motion+); custom hook in ~30 lines is sufficient |
| CSS transitions for expand/collapse | Motion AnimatePresence + layout animations | Ongoing | CSS cannot animate to `height: auto`; Motion measures and animates dynamically |

**Deprecated/outdated:**
- `framer-motion` package name: Still works but `motion` is the current package. Import from `"motion/react"` not `"framer-motion"`.
- Intersection Observer polyfill: No longer needed; all modern browsers support it natively.

## Open Questions

1. **Team photos for CTA**
   - What we know: Josh + Jacqui photos side by side for team feel
   - What's unclear: Whether photos are already uploaded to Supabase Storage or need to be added as static assets
   - Recommendation: Use static images in `/public/team/` directory. These are site-wide assets, not per-homeowner data. Optimize with Next.js `<Image>`.

2. **Chart draw-in approach**
   - What we know: Chart should "draw itself" on scroll trigger
   - What's unclear: Whether line-trace (draws left to right) or area-grow (fills from bottom) looks better
   - Recommendation: Use Recharts' built-in animation with `animationEasing="ease-in-out"` and `animationDuration={1500}`. Recharts' default animation draws the area from left to right, which naturally looks like "drawing itself." Test both `isAnimationActive` toggle approaches to find the one that feels most cinematic.

3. **Milestone placement accuracy**
   - What we know: Milestones render inline within relevant sections
   - What's unclear: Exact positioning -- before the section, after the hero number, or as a floating badge
   - Recommendation: Render as a small tag/badge below the hero number in the relevant section. This pairs the milestone with its context without disrupting the narrative flow.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Vitest 4.x with jsdom |
| Config file | `vitest.config.ts` (exists from Phase 1) |
| Quick run command | `npm test` |
| Full suite command | `npm test` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| NARR-03 | Sections animate on scroll (counters, chart draw) | unit (hook) + manual | `npx vitest run src/lib/hooks/useCountUp.test.ts` | No -- Wave 0 |
| NARR-04 | Narrative adapts by ownership duration | unit | `npx vitest run src/lib/narrative.test.ts` | No -- Wave 0 |
| NARR-05 | Milestone markers appear inline | unit | `npx vitest run src/lib/milestones.test.ts` | No -- Wave 0 |
| NARR-08 | Soft CTA with sms link | smoke (render) | `npx vitest run src/components/homeowner/CTASection.test.tsx` | No -- Wave 0 |
| SCEN-01 | Hold & Rent card displays data | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "hold"` | No -- Wave 0 |
| SCEN-02 | Move-Up card displays data | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "move"` | No -- Wave 0 |
| SCEN-03 | Equity Play card displays data | smoke (render) | `npx vitest run src/components/homeowner/ScenarioCards.test.tsx -t "equity"` | No -- Wave 0 |

### Sampling Rate
- **Per task commit:** `npm test`
- **Per wave merge:** `npm test && npm run build`
- **Phase gate:** Full suite green + visual review of animations on mobile viewport before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `src/lib/narrative.test.ts` -- voice selection boundary tests (23mo, 24mo, 83mo, 84mo) covers NARR-04
- [ ] `src/lib/milestones.test.ts` -- milestone selection and ranking logic covers NARR-05
- [ ] `src/lib/hooks/useCountUp.test.ts` -- counter easing and final value accuracy covers NARR-03
- [ ] `src/components/homeowner/ScenarioCards.test.tsx` -- render tests for all 3 cards covers SCEN-01/02/03
- [ ] `src/components/homeowner/CTASection.test.tsx` -- render test for sms link covers NARR-08
- [ ] Motion install: `npm install motion` -- not yet in dependencies

## Sources

### Primary (HIGH confidence)
- Project source code summaries: Phase 1 and Phase 2 plan summaries -- all types, calculations, component structure
- Phase 1 Research: `HomeownerMetrics` interface with holdAndRent, moveUp, equityPlay scenario data
- Phase 2 Research: Component architecture, SB7 patterns, SectionWrapper, Recharts integration
- 03-CONTEXT.md: Locked decisions on animation feel, narrative voices, scenario presentation, CTA design

### Secondary (MEDIUM confidence)
- [Motion npm](https://www.npmjs.com/package/motion) -- v12.35.2 current, `whileInView` + `viewport={{ once: true }}` API confirmed
- [Motion scroll animations docs](https://www.framer.com/motion/scroll-animations/) -- whileInView, useInView, viewport options
- [Motion useInView docs](https://motion.dev/docs/react-use-in-view) -- ref-based scroll detection
- [Recharts animation docs](https://github.com/recharts/recharts/issues/375) -- isAnimationActive, animationBegin, animationDuration props

### Tertiary (LOW confidence)
- None -- all findings verified with official sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- Motion is the clear choice for React scroll animations; all other libraries already installed
- Architecture: HIGH -- builds directly on established project patterns; all calculation data already available
- Pitfalls: HIGH -- mobile animation performance is well-understood; countUp timing issues are common and preventable
- Narrative copy: MEDIUM -- SB7 voice adaptation logic is straightforward but exact copy quality requires iteration

**Research date:** 2026-03-12
**Valid until:** 2026-04-12 (stable stack, no fast-moving dependencies)
