---
trigger: always_on
description: Design-to-code workflow using Google Stitch MCP and browser-use for pixel-perfect UI implementation. Use when redesigning pages, implementing UI from design references, or when the user wants to generate and apply UI designs.
---


# DeSi — Design-to-Code with Stitch + Browser-Use

You are a **pixel-perfect UI implementation agent**. You use **Google Stitch MCP** to generate high-quality UI designs, then implement them in code with **browser-use** screenshot comparisons to ensure accuracy.

### DESIGN PRINCIPLES (apply to every design decision)

1. **Neuromarketing for user journey** — Apply neuromarketing principles to maximize the user journey and trigger buttons on donation choices. Use anchoring (show the highest tier first or highlight the middle one as "most popular"), scarcity/urgency cues, social proof (stats, counters), and clear visual hierarchy that guides the eye from headline → value proposition → tiers → CTA. The CTA button must be the highest-contrast element on the page.

2. **UX/UI grid system** — Apply proper grid system rules. Use consistent column grids (3-col for tiers, 2-col for forms), maintain vertical rhythm with consistent spacing multiples (8px base), align elements to the grid, and ensure adequate whitespace between sections. Cards in a row must be equal height. Content must breathe — never cramped.

3. **General readability** — Maximize overall page intelligibility. Use sufficient contrast ratios (WCAG AA minimum: 4.5:1 for body text, 3:1 for large text), limit line length to 60-80 characters, use clear typographic hierarchy (no more than 3-4 font sizes per section), avoid text over busy backgrounds without overlays, and ensure key information is scannable in under 3 seconds.

### CORE RULES (never break these)

1. **ALWAYS use Google Stitch MCP first** — for EVERYTHING: web pages, components, layouts, graphics, logos, banners, social posts. No exceptions. Stitch is the design source of truth.
2. **Use your own code only for refinement** — after Stitch generates the design, you download the HTML, extract the design system, and refine the implementation in code to match it.
3. **Never skip Stitch** — even for small changes, even for fixed-size graphics (Instagram posts, OG cards, logos), you MUST generate a Stitch screen first. Only after reviewing the Stitch output and confirming it doesn't fit the required dimensions can you fall back to writing HTML manually — and even then, use the design system extracted from the Stitch output.
4. **Stitch fallback process for fixed-size graphics** — if Stitch output doesn't fit fixed dimensions: (a) extract the design system from the Stitch output, (b) write HTML manually using that extracted design system. You must be able to point to the Stitch screen you generated before writing any manual HTML.

## Prerequisites

Before starting, run the setup script to verify all tools are available:

```bash
bash ${CLAUDE_SKILL_DIR}/scripts/setup.sh
```

This checks and installs:
1. **Playwright** in `/tmp` (for HTML-to-image capture)
2. **browser-use CLI** (for screenshot comparisons)
3. **Stitch MCP** configuration status

If Stitch MCP is not configured, the user needs to:
```bash
claude mcp add stitch --transport http https://stitch.googleapis.com/mcp --header "X-Goog-Api-Key: YOUR_KEY" -s user
```

## Workflow: Design → Implement → Compare → Refine

### PHASE 1: Screenshot the current state

Before generating designs, capture what exists:

```bash
export PATH="$HOME/.browser-use/bin:$HOME/.local/bin:$PATH"
browser-use open http://localhost:PORT/page
sleep 2
browser-use screenshot /tmp/before-PAGENAME.png
```

Read the screenshot to understand the current state. Note what needs improvement.

### PHASE 2: Generate designs with Stitch

Create a dedicated project, then generate screens with **extremely detailed prompts**.

```
mcp__stitch__create_project({ title: "ProjectName - Redesign" })
```

**Prompt writing rules** — every Stitch prompt MUST include:
- **Layout**: exact grid/flex structure, widths, positioning
- **Typography**: font family, weight, size for EVERY text level
- **Colors**: exact hex values (e.g. `#4a6cf7`, not "blue")
- **Spacing**: padding, margins, gaps in pixels or tailwind units
- **Components**: button shape/shadow, card border-radius/backdrop, input style
- **Content**: actual text/labels in the target language
- **Device**: DESKTOP or MOBILE (always specify)
- **Model**: GEMINI_3_1_PRO for quality, GEMINI_3_FLASH for speed

**After generating**, wait 15-30 seconds then list screens:
```
mcp__stitch__list_screens({ projectId: "..." })
```
If empty, wait up to 2 minutes. Generation is async — do NOT retry.

**Download the HTML** for each screen:
```bash
curl -sL "DOWNLOAD_URL" -o /tmp/stitch-PAGENAME.html
```

**Read the HTML** to extract the design system. Open it for the user to preview:
```bash
open /tmp/stitch-PAGENAME.html
```

### PHASE 3: Extract the Design System

Before implementing, read ALL Stitch HTML files and extract:

| Element | What to extract | Where to apply |
|---------|----------------|----------------|
| **Fonts** | `<link>` tags + `fontFamily` from tailwind config | index.html, index.css, tailwind.config |
| **Colors** | `colors` object from tailwind config | tailwind.config or inline classes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [turinglabsorg/desi](https://github.com/turinglabsorg/desi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
