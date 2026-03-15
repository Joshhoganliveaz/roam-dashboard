---
type: design-spec
phase: 07-future-scenarios-rework
status: approved
created: 2026-03-14
author: josh + claude (brainstorm session)
affects:
  - src/components/homeowner/HeroSection.tsx
  - src/components/homeowner/PaycheckHook.tsx
  - src/components/homeowner/HomeownerPage.tsx
  - src/app/globals.css
  - src/app/layout.tsx
  - public/logo-white.png
---

# Hero Section Redesign Spec

## Summary

Replace the current HeroSection + PaycheckHook two-section layout with a single premium hero section. Full-bleed home photo with warm gradient overlay and frosted glass panels containing the homeowner identity and weekly paycheck stat. The stat is the visual centerpiece — oversized Playfair Display in gold at ~112px. The goal: screenshot-worthy, "show a neighbor" quality.

## Design Decisions (Approved)

All decisions below were validated through visual mockups in a brainstorming session with Josh.

### Layout

**Desktop (md+):**
- Full-bleed home photo background with warm bottom-heavy gradient overlay
- Two frosted glass panels positioned asymmetrically:
  - **Top-left panel:** Live AZ Co logo (white PNG) + homeowner name (Source Serif 4) + address (DM Sans)
  - **Bottom-right panel:** "Your home earned you" label + $597 stat (Playfair Display, gold) + "per week" label
- Creates a diagonal reading path (upper-left → lower-right) across the photo
- 3px cream strip at bottom edge for seamless transition to page content

**Mobile (<md):**
- Full-bleed home photo with stronger gradient (more opacity for readability)
- Logo floats top-left corner — small text, no frosted panel (saves space)
- Single frosted panel at bottom, full-width, containing:
  - Homeowner name + address (centered)
  - Gold horizontal divider
  - "Your home earned you" + $597 + "per week" (centered)

### Typography

| Element | Font | Weight | Size (desktop) | Size (mobile) | Color |
|---------|------|--------|----------------|---------------|-------|
| $597 stat | Playfair Display | 700 | 112px | 76px | #C9953E (gold) |
| "per week" | DM Sans | 400 | 18px | 14-15px | rgba(201,149,62,0.55) |
| "Your home earned you" | DM Sans | 400 | 13px | 11px | rgba(250,247,242,0.6), uppercase, letter-spacing: 2.5px |
| Homeowner name | Source Serif 4 | 600 | 22px | 17px | rgba(250,247,242,0.95) |
| Address | DM Sans | 400 | 13px | 11-12px | rgba(250,247,242,0.55) |
| Logo text (mobile fallback) | DM Sans | 700 | — | 13px | rgba(250,247,242,0.5), uppercase, letter-spacing: 2px |

**New font dependency:** Playfair Display 700 — used ONLY for the hero stat. All other page typography remains Source Serif 4 + DM Sans.

### Gradient Overlay

```
Desktop: linear-gradient(180deg,
  rgba(44,44,44,0.1) 0%,
  rgba(44,44,44,0.2) 30%,
  rgba(44,44,44,0.45) 55%,
  rgba(44,44,44,0.7) 80%,
  rgba(44,44,44,0.85) 100%
)

Mobile: stronger — increase mid and bottom stops:
  rgba(44,44,44,0.05) 0%,
  rgba(44,44,44,0.1) 20%,
  rgba(44,44,44,0.3) 45%,
  rgba(44,44,44,0.65) 65%,
  rgba(44,44,44,0.88) 85%,
  rgba(44,44,44,0.95) 100%
```

Photo visible through the top ~35-40%. Bottom where text lives is near-solid.

### Frosted Glass Panels

```css
background: rgba(44, 44, 44, 0.5);
backdrop-filter: blur(16px);
-webkit-backdrop-filter: blur(16px);
border-radius: 12px; /* 10px on mobile */
border: 1px solid rgba(250, 247, 242, 0.08);
```

Desktop padding: 20-28px (name panel), 28-40px (stat panel)
Mobile padding: 24px

### Stat Styling

```css
color: #C9953E;
font-family: 'Playfair Display', Georgia, serif;
font-size: 112px; /* 76px mobile */
font-weight: 700;
line-height: 1.05;
letter-spacing: -3px; /* -2px mobile */
text-shadow: 0 2px 20px rgba(201, 149, 62, 0.2);
```

### Gold Divider (Mobile Only)

Between name/address and stat inside the single panel:
```css
height: 1px;
background: linear-gradient(to right, transparent, rgba(201, 149, 62, 0.3), transparent);
margin: 0 20px 16px;
```

### Logo

- **Source file:** `/Users/joshuahogan/Library/CloudStorage/GoogleDrive-josh@liveazco.com/Shared drives/live AZ co/Final Brand Logos/Final Logo White/Live Az Co White Logo - Bold Text - PNG.png`
- **Copy to:** `public/logo-white.png` (resize to ~200px wide for web)
- **Desktop:** Rendered inside top-left frosted panel at ~52px height, opacity 0.85
- **Mobile:** Logo floats top-left without a panel, small text fallback ("LIVE AZ CO") or small image

### Negative Equity Handling

When `earningsPerWeek <= 0`:
- **Skip the stat panel entirely** — do not show the bottom-right (desktop) or bottom (mobile) frosted panel
- Hero shows only: home photo + gradient + logo/name panel (desktop) or logo + photo (mobile)
- The messaging about shifting values comes in the sections below the hero
- This avoids drawing attention to a negative situation at the screenshot-worthy moment

### Section Height

- Desktop: `h-[28rem]` (448px) — current height, keep it
- Mobile: `h-[35rem]` (560px) — taller to accommodate single bottom panel

## Components Affected

### Delete
- `src/components/homeowner/PaycheckHook.tsx` — absorbed into the new HeroSection

### Modify
- `src/components/homeowner/HeroSection.tsx` — full rewrite with new design
- `src/components/homeowner/HomeownerPage.tsx` — remove PaycheckHook import/render, pass earningsPerWeek back to HeroSection
- `src/app/layout.tsx` — add Playfair Display font import (700 weight only)
- `src/app/globals.css` — add `--font-playfair` CSS variable, add to `@theme inline`

### Create
- `public/logo-white.png` — resized logo for web use

## Props (New HeroSection Interface)

```typescript
interface HeroSectionProps {
  clientName: string;
  address: string;
  city: string;
  state: string;
  homePhotoUrl: string | null;
  earningsPerWeek: number;
}
```

`isNegative` is derived internally: `earningsPerWeek <= 0`.

## Tailwind Considerations

- `backdrop-blur-md` or `backdrop-blur-[16px]` for frosted glass
- Custom gradient stops via arbitrary values
- Responsive breakpoints: default = mobile, `md:` = desktop
- Playfair Display added as `font-display` in theme config

## Test Expectations

- Hero renders with photo when `homePhotoUrl` is provided
- Hero renders gradient fallback when `homePhotoUrl` is null
- Stat panel shows when `earningsPerWeek > 0`
- Stat panel hidden when `earningsPerWeek <= 0`
- Logo image renders
- Correct font family applied to stat (Playfair Display)
- Mobile layout shows single panel (not two)

## Visual Reference

Mockup files preserved in `.superpowers/brainstorm/89091-1773498139/`:
- `hero-legibility.html` — frosted panel approach (option B selected)
- `hero-fonts-clean.html` — Playfair Display selected
- `mobile-layout.html` — single panel selected
