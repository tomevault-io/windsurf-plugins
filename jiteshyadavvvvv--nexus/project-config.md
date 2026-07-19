---
trigger: always_on
description: > **MANDATORY READ FOR ALL AI AGENTS**: Before writing any frontend code, read this
---

# NEXUS — Agent Design Brief

> **MANDATORY READ FOR ALL AI AGENTS**: Before writing any frontend code, read this
> entire file. The design system defined here is the single source of truth.
> Do not introduce new colors, fonts, layouts, or component patterns that
> contradict anything in this document.
> **CRITICAL**: For accessing backend code refer to .\sync-backend. You have full permission to make changes in backend. but be carefull to not break existing code.

---

## 🔒 Project Identity

**NEXUS** is the central club management platform for Army Institute of Technology (AIT), Pune.
It is built for students and club executives. The visual identity is deliberately:
- **Dark-first** — pure black backgrounds, not dark grey
- **Minimal but precise** — sharp typography, intentional spacing, no decoration for decoration's sake
- **Tech-adjacent** — monospace accents, electric borders, technical vocabulary

**Do not** introduce light backgrounds, warm palettes, serif fonts, or rounded/bubbly component styles.

---

## 🎨 Color System

All custom tokens are defined in `frontend/src/index.css` under `:root` and `.dark`.
**Always use CSS variables — never hardcode colors directly in JSX.**

### CSS Variables (Light Mode — rarely used)
| Token | Value | Usage |
|---|---|---|
| `--bg` | `#f6f7f9` | Page background |
| `--panel` | `#ffffff` | Surface / card |
| `--text` | `#0b1220` | Primary text |
| `--muted` | `#6b7280` | Secondary/dimmed text |
| `--accent` | `#0f62fe` | Primary CTA, active states |
| `--accent-2` | `#0066d6` | Hover variant of accent |
| `--border` | `#e6e9ef` | Dividers, card outlines |
| `--glass` | `rgba(255,255,255,0.6)` | Glassmorphism surfaces |

### CSS Variables (Dark Mode — the primary theme)
| Token | Value | Usage |
|---|---|---|
| `--bg` | `#020617` | Page background |
| `--panel` | `#0f172a` | Surface / card |
| `--text` | `#f8fafc` | Primary text |
| `--muted` | `#94a3b8` | Secondary/dimmed text |
| `--accent` | `#3b82f6` | Primary CTA, active states |
| `--accent-2` | `#60a5fa` | Hover variant of accent |
| `--border` | `#1e293b` | Dividers, card outlines |
| `--glass` | `rgba(15,23,42,0.6)` | Glassmorphism surfaces |

### Page-level Background
The `body` background is hardcoded to `#000000` (pure black), not `--bg`.
This is intentional — pages use absolute/fixed black as the canvas.

### Tailwind Semantic Colors Used in Components
When using Tailwind utility classes directly in JSX, adhere to this palette:
| Purpose | Tailwind class(es) |
|---|---|
| Primary text | `text-white` |
| Secondary text | `text-gray-400`, `text-gray-500` |
| Background surfaces | `bg-white/5`, `bg-white/10` |
| Borders | `border-white/10`, `border-slate-700/70` |
| Accent / CTA | `bg-indigo-500`, `hover:bg-indigo-400` |
| Focus rings | `focus:ring-indigo-500` |
| Error states | `text-red-500/50` |

### Section Title Colors (via CSS variables)
| Variable | Light | Dark |
|---|---|---|
| `--title-h1` | `#ff6b6b` | `#f87171` |
| `--title-h2` | `#3b82f6` | `#60a5fa` |
| `--title-h3` | `#1e3a8a` | `#93c5fd` |
| `--title-h4` | `#00c29a` | `#34d399` |

---

## 🔤 Typography

### Font Stack (Priority Order)
```css
body {
  font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial;
}
```

### Decorative / Display Fonts (use sparingly, only where already established)
| Font | Class / Usage |
|---|---|
| `Science Gothic` | Hero headings (`.home-hero .section-title`) — 100px, tracked |
| `Jersey 20` | Themed accent text (`.oi-regular`, Tailwind token `font-jersey-20`) |
| `Black Ops One` | Specific branded headings (`.black-ops-one-regular`) |
| `Foldit` | Experimental display (`.foldit-regular`) |

**Do not** introduce new Google Fonts. Use only fonts already imported in `index.css`.

### Type Scale Rules
- `font-mono` is used throughout page-level layouts (sections, main containers)
- `font-sans` is used for body text and UI labels
- Hero section titles: `font-size: 100px`, `font-weight: 900`, `text-transform: uppercase`
- Section subtitles: `text-3xl md:text-4xl lg:text-5xl font-bold tracking-tight`
- Body / lead: `text-base md:text-lg`, `leading-relaxed`, `text-gray-500`

---

## 📐 Layout & Spacing

### Container
```css
.site-container {
  max-width: 1200px;  /* --container-max */
  margin: 0 auto;
  gap: 22px;          /* --gap */
  padding-top: 140px;
}
```
Standard section inner width: `max-w-[1200px]`, padded `px-6 sm:px-8`.

### Spacing Tokens
| Token | Value |
|---|---|
| `--radius` | `12px` — default border-radius |
| `--gap` | `22px` — grid/flex gap |
| `--navbar-height` | `56px` |
| `--shadow` | `0 10px 30px rgba(16,24,40,0.06)` (light) / `rgba(0,0,0,0.5)` (dark) |

### Grid
- 3-column event/card grid: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 md:gap-10`
- 2-column card fallback at ≤1000px, 1-column at ≤720px

---

## 🧩 Component Patterns

### Buttons
There are two established button styles. Do not invent new ones.

**Ghost/Secondary (navigation, back buttons):**
```jsx
className="flex items-center gap-2 px-5 py-2.5 text-sm font-semibold
           bg-white/5 hover:bg-white/10 text-gray-300 border border-white/10
           transition-all hover:-translate-y-0.5 active:scale-95 duration-300 rounded-lg"
```

**Primary CTA (forms, OTP, submit):**
```jsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiteshYadavvvvv/NEXUS](https://github.com/JiteshYadavvvvv/NEXUS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
