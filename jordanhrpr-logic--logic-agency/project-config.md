---
trigger: always_on
description: Marketing website for Logic Agency Inc., a 2-person packaging and supply chain operations agency. The site has one job: convert brands searching for packaging help into monthly retainer clients.
---

# CLAUDE.md — Logic Agency Inc.

## What This Project Is

Marketing website for Logic Agency Inc., a 2-person packaging and supply chain operations agency. The site has one job: convert brands searching for packaging help into monthly retainer clients.

## Company Context

Logic Agency is an outsourced packaging and supply chain ops team on a monthly retainer. Founded by Jordan Harper, 20+ years in China/Asia supply chain operations. Based in Orange County, CA and Salt Lake City, UT.

**What they do:** Packaging sourcing, supply chain management, retail execution, inventory management, vendor coordination, global sourcing across 15+ countries.

**Who they serve:** Brands in beauty/wellness, CPG, and tech wearables doing $5-20M+ revenue, shipping 50K-500K+ units. Clients include Adidas, Vans, Target, Disney, Puma, Paramount+, Spotify, Epicutis, A24.

**How they charge:** Monthly retainers, not projects.
- Starter: $2,500-3,000/month — Advisory and sourcing
- Growth: $5,000-7,000/month — Packaging program management (most popular)
- Enterprise: $10,000+/month — Embedded operations

**Contact:**
- jordan@logicagencyinc.com
- 385.368.6837
- Domain: logicagencyinc.com

## Site Architecture

```
/                                    → Homepage (conversion page with tiers)
/guides/retail-ready-packaging       → Guide: Getting Your Packaging Retail-Ready
/guides/packaging-cost-reduction     → Guide: Packaging Cost Reduction Without Sacrificing Brand
/guides/packaging-system-that-scales → Guide: Building a Packaging System That Scales
/guides/packaging-sourcing           → Guide: How to Source Packaging Without Getting Burned
```

Future pages (not yet built):
- `/guides/us-market-entry` — US Market Entry for International Brands (build when Haldirams case study has results)

## Design System

### Colors
```css
--o: #FF600A;    /* Tiger Orange — primary brand color, CTAs, accents */
--od: #E55509;   /* Orange dark — hover states */
--bl: #0055FF;   /* Accent Blue — secondary accent, rarely used */
--bk: #212121;   /* Near black — body text */
--gr: #3E3E3E;   /* Gray — secondary text */
--dk: #2A2A2A;   /* Dark — dark section backgrounds */
--lt: #DFDFDF;   /* Light gray — borders, dividers */
--ow: #F3F3F3;   /* Off-white — light section backgrounds */
--w: #FFFFFF;    /* White — cards, content areas */
```

### Typography
- Font: **Poppins** (Google Fonts) — weights 400, 500, 600, 700, 800, 900
- Headlines: 800 weight, tight letter-spacing (-1.5px to -2.5px), tight line-height (1.05-1.15)
- Body: 400 weight, 16.5px, line-height 1.8, color `var(--gr)`
- Labels: 11-13px, 700 weight, uppercase, letter-spacing 1.5-2.5px

### Grid Texture
Light and dark sections use a subtle grid background:
```css
/* Light grid */
.gl { background-color: var(--ow); background-image: linear-gradient(rgba(0,0,0,.03) 1px, transparent 1px), linear-gradient(90deg, rgba(0,0,0,.03) 1px, transparent 1px); background-size: 28px 28px; }

/* Dark grid */
.gd { background-color: var(--dk); background-image: linear-gradient(rgba(255,255,255,.04) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,.04) 1px, transparent 1px); background-size: 28px 28px; }
```

**CRITICAL:** Every section must have an explicit `background-color`, not just `background-image`. Without it, mobile browsers show transparent sections and text becomes unreadable against inherited dark backgrounds.

### Logo
SVG — two overlapping circles in Tiger Orange:
```html
<svg viewBox="0 0 40 40" fill="none">
  <circle cx="15" cy="20" r="12" stroke="#FF600A" stroke-width="2.5" fill="none"/>
  <circle cx="25" cy="20" r="12" stroke="#FF600A" stroke-width="2.5" fill="none"/>
</svg>
```

### Buttons
- Primary: `.bo` — Tiger Orange background, white text, 50px border-radius
- Ghost light: `.bg` — transparent, dark text, light border
- Ghost dark: `.bw` — transparent, white text, white border at 40% opacity
- All buttons: 16px 36px padding, 700 weight, 15px font size, hover lifts with translateY(-2px)

### Section Pattern
Sections alternate between light (`.gl`) and dark (`.gd .dks`) backgrounds. Dark sections use white text with opacity for hierarchy (100% headlines, 85% body, 75% secondary, 60% tertiary, 50% labels).

### Navigation
Fixed dark nav bar, 72px height, backdrop blur. Logo left, links right. "Let's Talk" CTA button in orange. Nav links hide on mobile (max-width: 640px).

### Cards
White background, 16-20px border-radius, 1px border at 4% black opacity, hover lifts with translateY(-4px to -6px) and box-shadow.

## Page Templates

### Homepage
Full marketing page with sections: Hero → Situations (problem cards) → How It Works → Pricing Tiers → Metrics → Case Studies → Industries → FAQ (accordion) → CTA → Footer.

### Guide Pages
Article layout: Nav → Hero (breadcrumb, h1, lede paragraph, meta bar) → Article body (800px max-width, prose with embedded components) → CTA Band → Related Guides → Footer.

Guide-specific components (vary by page):
- Numbered section cards
- Comparison tables (dark header, alternating rows)
- Callout boxes (white, orange left border)
- Timeline layouts
- Checklist grids

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhrpr-logic/logic-agency](https://github.com/jordanhrpr-logic/logic-agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
