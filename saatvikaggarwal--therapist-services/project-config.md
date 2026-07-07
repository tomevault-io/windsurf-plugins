---
trigger: always_on
description: **Stillwater** is a single-page therapy services website exported from Figma using the Figma Make tool. It is a static, client-side React app with no backend. The site presents a fictional therapy practice with a service catalogue, therapist profile, testimonials, and a contact form.
---

# Therapy Services Website — CLAUDE.md

## Project Overview

**Stillwater** is a single-page therapy services website exported from Figma using the Figma Make tool. It is a static, client-side React app with no backend. The site presents a fictional therapy practice with a service catalogue, therapist profile, testimonials, and a contact form.

- **Stack:** React 18 · Vite 6 · TypeScript · Tailwind CSS v4 · shadcn/ui (Radix UI primitives)
- **Package manager:** npm (pnpm-workspace.yaml exists but README and runtime use npm)
- **Entry point:** `src/main.tsx` → mounts `<App />` into `#root`
- **Dev server:** `npm run dev` → `http://localhost:5173`
- **Build:** `npm run build`

---

## Directory Structure

```
therapy-figma-export/
├── index.html                      # HTML shell — sets title, meta description
├── package.json                    # Dependencies and scripts
├── vite.config.ts                  # Vite config — React plugin, Tailwind, path alias @/, figma asset resolver
├── postcss.config.mjs
├── pnpm-workspace.yaml
├── default_shadcn_theme.css        # Reference theme (not imported)
├── ATTRIBUTIONS.md
│
└── src/
    ├── main.tsx                    # App entry — createRoot → <App />
    │
    ├── app/
    │   ├── App.tsx                 # Entire page — all sections live here (single file, no routing)
    │   └── components/
    │       ├── figma/
    │       │   └── ImageWithFallback.tsx   # <img> wrapper with error fallback SVG
    │       └── ui/                         # shadcn/ui component library (42 files)
    │           ├── accordion.tsx
    │           ├── alert-dialog.tsx
    │           ├── alert.tsx
    │           ├── aspect-ratio.tsx
    │           ├── avatar.tsx
    │           ├── badge.tsx
    │           ├── breadcrumb.tsx
    │           ├── button.tsx
    │           ├── calendar.tsx
    │           ├── card.tsx
    │           ├── carousel.tsx
    │           ├── chart.tsx
    │           ├── checkbox.tsx
    │           ├── collapsible.tsx
    │           ├── command.tsx
    │           ├── context-menu.tsx
    │           ├── dialog.tsx
    │           ├── drawer.tsx
    │           ├── dropdown-menu.tsx
    │           ├── form.tsx
    │           ├── hover-card.tsx
    │           ├── input-otp.tsx
    │           ├── input.tsx
    │           ├── label.tsx
    │           ├── menubar.tsx
    │           ├── navigation-menu.tsx
    │           ├── pagination.tsx
    │           ├── popover.tsx
    │           ├── progress.tsx
    │           ├── radio-group.tsx
    │           ├── resizable.tsx
    │           ├── scroll-area.tsx
    │           ├── select.tsx
    │           ├── separator.tsx
    │           ├── sheet.tsx
    │           ├── sidebar.tsx
    │           ├── skeleton.tsx
    │           ├── slider.tsx
    │           ├── sonner.tsx
    │           ├── switch.tsx
    │           ├── table.tsx
    │           ├── tabs.tsx
    │           ├── textarea.tsx
    │           ├── toggle-group.tsx
    │           ├── toggle.tsx
    │           ├── tooltip.tsx
    │           ├── use-mobile.ts   # Hook: returns boolean via window.matchMedia (768px breakpoint)
    │           └── utils.ts        # cn() helper — merges clsx + tailwind-merge
    │
    └── styles/
        ├── index.css               # Imports fonts → tailwind → theme (in that order)
        ├── fonts.css               # Google Fonts import: Fraunces (serif) + Nunito (sans-serif)
        ├── tailwind.css            # @import 'tailwindcss'
        ├── theme.css               # CSS custom properties (:root + .dark) + @theme inline block
        └── globals.css             # Additional global styles (if any)
```

---

## App.tsx — Page Sections

The entire page is one component (`App.tsx`). No react-router; all navigation uses anchor links (`#services`, `#approach`, etc.).

### State

| Variable | Type | Purpose |
|---|---|---|
| `activeCategory` | `string` | Active filter tab in the Services section ("All" by default) |
| `menuOpen` | `boolean` | Controls mobile hamburger menu open/closed |
| `expandedService` | `number \| null` | ID of currently expanded service card (accordion toggle) |

### Data Constants (top of file)

| Constant | Type | Description |
|---|---|---|
| `services` | `array[12]` | Service objects: `id, category, title, description, duration, format, icon, color, accent, image` |
| `categories` | `string[]` | `["All", "Individual", "Couples & Family", "Group", "Specialized"]` — drives filter buttons |
| `testimonials` | `array[3]` | `{ quote, name, service }` — client quotes |

### Page Sections (in render order)

| Section | ID / Element | Description |
|---|---|---|
| **Navbar** | `<nav>` fixed top | Logo + nav links + CTA button. Mobile: hamburger toggles `menuOpen` |
| **Hero** | `<section>` | Headline, subtext, two CTA buttons, hero image |
| **Approach** | `id="approach"` | 3-pillar cards: "Whole-person care", "Evidence and intuition", "Unhurried pace" |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Saatvikaggarwal/therapist_services](https://github.com/Saatvikaggarwal/therapist_services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
