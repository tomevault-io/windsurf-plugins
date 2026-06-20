---
trigger: always_on
description: >
---


# Mobile App UI/UX Design Skill

This skill produces **pixel-quality, production-grade mobile app UIs** that look like they came from
a top-tier product studio. Reference apps: **Airbnb** (trust + clarity), **Spotify** (personalization
+ emotional peaks), **Duolingo** (delight + feedback loops), **Revolut** (tactile + premium),
**Phantom** (polish = trust in high-stakes domains).

---

## Core Philosophy

Great mobile UI is about **intentionality** — not flashiness. Before any layout or code, answer:

1. **What is the user trying to accomplish?** Reduce friction to that goal.
2. **How should this make the user feel?** Trust, delight, confidence, calm?
3. **What is the ONE thing they should notice first?** Visual hierarchy must answer this.

---

## Step 0 — Load the design playbook first

Before writing any code, read the reference that matches the app type:

| File | Contains |
|---|---|
| `references/visual-system.md` | Colors per industry, typography scale, spacing tokens, elevation, motion |
| `references/components.md` | Production-ready CSS for every component |
| `references/industry-patterns.md` | Screen layouts for finance, AI, health, e-commerce, social |
| `references/industry-conventions.md` | Emotional design, Peak-End rule, Spotify/Duolingo/Revolut lessons |

Read **all four** before starting.

---

## Step 1 — Understand the context

Extract or infer before designing:

1. **App type / industry** — finance, health, AI, e-commerce, social, productivity, crypto
2. **Screen(s) requested** — onboarding, dashboard, detail, auth, settings, chat
3. **User stage** — new (guided), returning (routine), power user (dense info)
4. **Platform feel** — iOS-leaning, Material/Android, or neutral cross-platform
5. **Mood / brand** — dark/light, vibrant/muted, playful/serious, minimal/rich

If ambiguous, **commit to an opinionated direction and state it**. Good designers make decisions.

---

## Step 2 — Choose the right output format

| Request | Output |
|---|---|
| Single screen / component | HTML artifact — single file, 390px mobile viewport |
| Multi-screen flow | HTML with tab/step navigation between screens |
| React component | JSX artifact with Tailwind or styled-components |
| Android / KMP / React Native | Platform-specific code (see Step 10) |

**Default:** Single HTML artifact at 390px (iPhone 15 viewport), wrapped in a phone shell with
status bar and bottom nav for full visual fidelity.

---

## Step 3 — Apply the visual system (non-negotiable)

### 3.1 Color (60/30/10 Rule)

| Share | Role |
|---|---|
| 60% | Neutral base — backgrounds, surfaces |
| 30% | Complementary — text, dark elements |
| 10% | Accent — CTAs, key indicators, icons |

Every color as a **CSS variable**. Use opacity for text hierarchy: 100% headings, 80% body,
60% secondary. Use accent at 5% opacity for secondary buttons. Match shadow color to background
hue — never pure black shadows on colored backgrounds.

**Industry palettes:**

```css
/* Finance / Banking — trust, stability */
:root {
  --bg: #F5F5F5; --surface: #FFFFFF; --accent: #1A56DB; --accent2: #1041A8;
  --accent-soft: #EBF0FF; --text: #111827; --text2: #6B7280; --text3: #9CA3AF;
  --success: #10B981; --error: #EF4444; --divider: rgba(0,0,0,0.07);
  --shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04);
}

/* Health / Fitness — energy, calm */
:root {
  --bg: #F0FAF4; --surface: #FFFFFF; --accent: #16A34A;
  --accent-soft: #DCFCE7; --highlight: #0EA5E9;
}

/* AI / Productivity — dark default */
:root {
  --bg: #0A0A0F; --surface: #16161E; --surface2: #1E1E2A;
  --accent: #7C3AED; --accent-soft: #2D1F5E;
  --text: #F8F8FC; --text2: #9CA3AF; --neon: #A78BFA;
}

/* Social / Lifestyle */
:root { --bg: #FAFAFA; --surface: #FFFFFF; --accent: #EC4899; --accent-soft: #FCE7F3; }

/* E-commerce */
:root {
  --bg: #F9FAFB; --surface: #FFFFFF; --accent: #F97316;
  --accent-soft: #FFF7ED; --badge: #EF4444;
}

/* Crypto / Web3 — futuristic, neon */
:root {
  --bg: #0D0D1A; --surface: #16162A; --accent: #9945FF;
  --accent-soft: #1E1040; --neon: #19FB9B; --text: #F8F8FC;
}
```

**Dark mode mappings:**

| Light token | Dark value |
|---|---|
| #FFFFFF surface | #1C1C1E |
| #F2F2F7 bg | #000000 |
| #111827 text | #F9FAFB |
| #6B7280 secondary | #9CA3AF |
| rgba(0,0,0,0.06) divider | rgba(255,255,255,0.08) |

---

### 3.2 Typography

**Max 4 sizes. Max 2–3 weights.**

| Token | Size | Weight | Use |
|---|---|---|---|
| display | 32–40px | 700 | Hero moments, balance totals, big stats |
| h1 | 24px | 700 | Screen titles |
| title | 20px | 600 | Section headers |
| body | 15px | 400 | Main content |
| label | 13px | 500 | Input labels, button text |
| caption | 12px | 400 | Meta, timestamps, helper text |

- Letter-spacing: **-0.5px** on display/h1, **-0.3px** on title, 0 on body+
- Line-height: **1.3–1.5**
- Use `font-variant-numeric: tabular-nums` for all prices, stats, and large numbers
- System fonts: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, sans-serif`

**Anti-pattern:** Never use more than 2 font families. Never bold everything — create hierarchy
with size, weight, AND opacity together.

---

### 3.3 Spacing (8pt grid — always)

| Token | px | Use |
|---|---|---|
| xs | 4 | Icon gaps, tight chips |
| sm | 8 | Between related elements |
| md | 12 | Input padding, compact cards |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sisovin/mobile-fullstack-skill](https://github.com/sisovin/mobile-fullstack-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
