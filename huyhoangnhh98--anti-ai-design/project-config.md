---
trigger: always_on
description: Auto-orchestrating UI design skill that prevents generic AI-generated output. Detects project context and target platform, explicitly asks the user which platforms to design for, resolves brand color direction through user choice or auto-selection, presents curated design style options, and generates a CJX-ready output bundle with html/css/js folders plus frozen foundation tokens for visual consistency. Use when asked for: UI design, landing pages, dashboards, app screens, product pages, onboard
---


# anti-ai-design

Auto-orchestrating UI design skill with mandatory anti-AI rules. Produces impossibly beautiful, visually distinct screens — never the generic AI-looked output.

## When to Use

Activate this skill whenever the user asks for:

- A landing page, hero section, or marketing screen
- A dashboard, admin panel, or data-rich interface
- A mobile app screen (iOS, Android) or responsive web UI
- An onboarding flow, signup form, or product walkthrough
- Any component library or design system screen
- "Make it look better", "redesign this", "create a UI for…"

## Non-Negotiable Interaction Contract

Before generating any design output, you MUST resolve these three decisions in order:

1. **Platform scope** — ask the user which platforms to generate for: `mobile`, `tablet`, `desktop`.
   - Multi-select is allowed.
   - If the user chooses only `mobile` and `desktop`, do NOT generate `tablet`.
   - If the platform is already explicitly stated and unambiguous, confirm by restating it and proceed.
2. **Color direction** — resolve brand color direction using this order:
   - If the user supplied exact colors, use them.
   - Else if the user supplied a brand/theme mood, offer 2–3 suitable palette directions and ask them to choose.
   - Else auto-select a palette that fits the product category and state clearly that it was auto-selected.
3. **Style direction** — present 2–3 style options and have the user choose one.

Do not skip these decisions unless the conversation already contains an unambiguous answer.

## Execution Mode

### Interactive (default)
- Ask the user for each missing choice in sequence: platform → color → style.
- Wait for their response before proceeding to the next phase.
- Present option cards for style direction with a RECOMMENDED label.

### Non-Interactive
Activate when the user:
- invokes the skill with `mode: non_interactive` or equivalent flag,
- is running a one-shot CLI command, background Codex job, or scripted pipeline, or
- provides a prompt with no prior conversational context (no chat turns before the request).

**Auto-selection rules (applied in order):**
1. **Platform** — if not stated, default to `desktop`. If mobile-first signals exist (mobile app description, React Native mention, "phone" context), default to `mobile`.
2. **Color direction** — select the best-fit palette for the detected project type without offering alternatives.
3. **Style direction** — from the 2–3 candidate styles for this project context, select the one that would be marked RECOMMENDED. Do not present option cards.

**Required output header before generating (non-interactive only):**

```
Auto-selected:
- Platform: [selected platform(s)]
- Palette: [palette name] — primary [hex value]
- Style: [style name] — [one-sentence vibe]
Proceeding to bundle generation.
```

Do not ask clarifying questions. Do not present option cards. Proceed directly to Phase 3 after the header.

## Output Contract — CJX Bundle, Not Single Loose HTML

Default output is a **bundle directory** for the project, not a single loose HTML file.

```text
<app-slug>-design/
  html/
    landing-page.html
    mobile-<screen>.html
    tablet-<screen>.html
    desktop-<screen>.html
  css/
    foundation.css
    shared.css
    landing-page.css
    mobile-<screen>.css
    tablet-<screen>.css
    desktop-<screen>.css
  js/
    app.js
    landing-page.js
    mobile-<screen>.js
    tablet-<screen>.js
    desktop-<screen>.js
```

### Bundle Rules

- `html/` contains one HTML file per generated screen/platform combination.
- `css/foundation.css` contains the frozen design tokens only.
- `css/shared.css` contains shared primitives, utilities, motion, glass recipes, focus states, and common layout patterns.
- Screen-specific CSS goes into a screen-specific file.
- `js/app.js` contains shared UI helpers, state toggles, mock data handling, and accessibility-safe interactions.
- Screen-specific JS goes into a screen-specific file only if needed.
- If the user asks for a single-file export later, you may additionally provide a self-contained HTML export, but the default contract is the CJX bundle.

### Platform File Naming Rules

- Landing pages stay semantic: `landing-page.html`, `landing-page.css`, `landing-page.js`
- App screens use platform prefixes:
  - `mobile-<screen>.html`
  - `tablet-<screen>.html`
  - `desktop-<screen>.html`
- Generate files only for the selected platforms.
- If the user selected `mobile` and `desktop`, the output bundle must not contain any `tablet-*` files.

### Why Separate Files Instead of One Responsive HTML

Use separate platform files by default because:

- each platform can have materially different IA, navigation, density, and CTA placement,
- review and handoff are easier when each target is isolated,
- CJX verification is clearer when the screen intent is explicit,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huyhoangnhh98/anti-ai-design](https://github.com/huyhoangnhh98/anti-ai-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
