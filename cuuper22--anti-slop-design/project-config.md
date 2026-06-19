---
trigger: always_on
description: >
---


# Anti-Slop Design System

Every AI model in existence has converged on the same aesthetic: purple gradient,
Inter font, 8px border radius, centered hero, three perfectly equal columns, a
sprinkle of Heroicons, and a dark mode that's just `#000000` with white text.
It's the design equivalent of elevator music. This skill exists because I got
tired of every AI-generated UI looking like it was built by the same sleep-deprived
intern at a Series A startup in 2023. Anti-slop covers 8 domain profiles (SaaS,
e-commerce, editorial, fintech, healthcare, creative, dev-tools, and general),
ships a 15-rule anti-slop checklist that catches generic output before it reaches
the user, and follows progressive disclosure: you're reading the hub doc now,
which points to `domain-map.json` for domain profiles, `references/` for
platform-specific guidance, `assets/` for tokens and textures, and `templates/`
for starting points. Read this file first. Everything else flows from here.


---

## Design Thinking Protocol

Before generating ANY design output — a React component, a landing page, an
artifact, a CLI app, whatever — run through these five steps. Every time. No
shortcuts. The whole point is that generic output is the default failure mode,
and the only way to avoid it is a deliberate process.

### Step 1: Identify Domain

Read the user's prompt. What are they building? A fintech dashboard? An
e-commerce checkout? A developer tool? Match keywords to one of the 8 domains
defined in `domain-map.json`:

- **saas** — B2B dashboards, admin panels, analytics platforms
- **ecommerce** — Product pages, carts, checkout flows, catalogs
- **editorial** — Blogs, magazines, news sites, content-heavy layouts
- **fintech** — Banking, trading, crypto, financial dashboards
- **healthcare** — Patient portals, EHR interfaces, wellness apps
- **creative** — Portfolio sites, agency pages, design studios
- **devtools** — IDEs, documentation, API explorers, terminal apps
- **general** — The fallback. Use this only if the prompt genuinely doesn't
  fit any other category. "Make me a website" with zero context? Fine, general.
  But try harder before defaulting here.

If the prompt is ambiguous — "build me a dashboard" could be SaaS, fintech, or
healthcare — ask the user. Don't guess. A fintech dashboard and a healthcare
dashboard should look nothing alike.

### Step 2: Load Domain Profile

Open `domain-map.json`. Find the matching domain key. Extract the full profile:

- **Color palette** — OKLCH values. Not hex. Not HSL. OKLCH, because it's
  perceptually uniform and you can derive tints/shades without things going
  muddy. Each domain has primary, secondary, accent, neutral, success, warning,
  error, and info colors with 10 lightness steps each.
- **Typography stack** — Primary and secondary font families, weights, and
  the fluid type scale to use. No, you cannot substitute Inter.
- **Border radius** — The `shape.borderRadius` value. Could be 2px for fintech,
  12px for creative, 6px for SaaS. It is NOT 8px for everything.
- **Motion level** — `none`, `minimal`, `moderate`, or `expressive`. Fintech
  gets `minimal`. Creative gets `expressive`. This controls whether you use
  transitions, spring animations, or nothing at all.
- **Density** — `compact`, `normal`, or `spacious`. Affects spacing multipliers.
- **Shadows** — Shadow style and intensity. Some domains use sharp shadows,
  some use diffuse, some use almost none.

### Step 3: Select Platform Reference

Based on what the user is building (not what domain they're in), load the
correct reference file from `references/`:

- Building a React dashboard? → `references/web-react.md`
- Building a landing page? → `references/web-landing.md`
- Building a Claude artifact? → `references/web-artifacts.md`
- Building an iOS app? → `references/mobile-native.md`
- Building a CLI tool? → `references/cli-terminal.md`
- And so on. See the Platform Routing Table below for the full mapping.

The reference file contains platform-specific constraints, patterns, and
anti-patterns. Read it. It's there for a reason.

### Step 4: Apply Anti-Slop Checklist

This is the core of the system. Before you show anything to the user, run
every output decision through the 15-rule checklist below. Every. Single. One.
If any rule is violated, fix it. Don't ship slop with a "you might want to
change the colors" disclaimer. Just fix it.

The checklist is not optional. It's not a suggestion. It's the whole point.

### Step 5: Assemble from Assets

Now build the thing:

1. Start with the appropriate template from `templates/`
2. Copy CSS tokens from `assets/css/` (reset, fluid scales, motion tokens)
3. Inject domain colors from `assets/tokens/domain-tokens/{domain}.json`
4. Apply domain typography, spacing, and shape values
5. Select SVG textures if the design calls for them
6. Customize the template with domain-specific values
7. Run the checklist one more time. Yes, again.


---

## Anti-Slop Checklist

Fifteen rules. Memorize them. Tattoo them on your forearm. Every single one
exists because AI models consistently get it wrong. These are not edge cases —
they are the most common failures in AI-generated design output.

### Rule 1: Typography

- **Check:** Is the primary font Inter, Roboto, or Open Sans?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cuuper22/anti-slop-design](https://github.com/Cuuper22/anti-slop-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
