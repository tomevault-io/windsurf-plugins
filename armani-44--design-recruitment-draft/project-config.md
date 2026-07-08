---
trigger: always_on
description: - **Invoke the `frontend-design` skill** before writing any frontend code, every session, no exceptions.
---

# CLAUDE.md — Frontend Website Rules

## Always Do First
- **Invoke the `frontend-design` skill** before writing any frontend code, every session, no exceptions.

## Reference Images
- If a reference image is provided: match layout, spacing, typography, and color exactly. Swap in placeholder content (images via `https://placehold.co/`, generic copy). Do not improve or add to the design.
- If no reference image: design from scratch with high craft (see guardrails below).
- Screenshot your output, compare against reference, fix mismatches, re-screenshot. Do at least 2 comparison rounds. Stop only when no visible differences remain or user says so.

## Local Server
- **Always serve on localhost** — never screenshot a `file:///` URL.
- Start the dev server: `node serve.mjs` (serves the project root at `http://localhost:3000`)
- `serve.mjs` lives in the project root. Start it in the background before taking any screenshots.
- If the server is already running, do not start a second instance.

## Screenshot Workflow
- Puppeteer is installed locally via the project's `node_modules` (macOS).
- **Always screenshot from localhost.** The simple path is `node screenshot.mjs http://localhost:3000`; for viewport-specific shots (mobile/tablet) or scrolled-section captures, use an inline `node --input-type=module` script that drives Puppeteer directly.
- Screenshots save to `./temporary screenshots/screenshot-N.png` (auto-incremented, never overwritten). Optional label suffix: `screenshot-N-label.png`.
- After screenshotting, read the PNG with the Read tool — Claude can see and analyze the image directly.
- When comparing, be specific: "heading is 32px but reference shows ~24px", "card gap is 16px but should be 24px".
- Check: spacing/padding, font size/weight/line-height, colors (exact hex), alignment, border-radius, shadows, image sizing.

## Stack & Output Defaults
- Plain hand-written HTML, CSS, and JavaScript. No build step, no framework, no Tailwind.
- Shared design tokens live in `tokens.css` (root level). Both the homepage and `about/index.html` link to it. Edit tokens once and the rest of the site follows.
- Per-page styles live in a `<style>` block in the page's `<head>`. Reuse class names across pages where the design is shared (e.g. `.section-tag`, `.fade-up`, `.serif-i`, `.btn-primary`, `.btn-ghost`, `.btn-dark-ghost`).
- Pages: `index.html` (homepage), `about/index.html` (About), plus `privacy.html` / `cookies.html` / `terms.html`. New top-level pages live in their own subfolder (`/<name>/index.html`) so URLs stay clean.
- Fonts in `/Fonts/` (Canela woff/woff2). Brand assets in `/Brand_assets/`. Reference both with absolute paths so they resolve from any subfolder.
- Mobile-first responsive. Breakpoints used elsewhere on the site: 1024px (tablet), 768px (mobile), 640px (small mobile). Stick to those.
- Placeholder images: `https://placehold.co/WIDTHxHEIGHT` only when no real asset is available.

## Brand Assets
- Always check the `brand_assets/` folder before designing. It may contain logos, color guides, style guides, or images.
- If assets exist there, use them. Do not use placeholders where real assets are available.
- If a logo is present, use it. If a color palette is defined, use those exact values — do not invent brand colors.

## Anti-Generic Guardrails
- **Colors:** Use the brand palette declared in `tokens.css` (`--color-midnight`, `--color-navy-deep`, `--color-blue`, `--color-gold`, `--color-cream`). Derive variants from those — don't invent new brand colours.
- **Shadows:** Use layered, dark-tinted shadows with low opacity (e.g. `0 2px 4px rgba(7,7,26,0.04), 0 12px 28px rgba(7,7,26,0.08)`). No flat single-layer shadows.
- **Typography:** Canela Deck (serif, italics enabled) for display/headings via `--font-display`; Montserrat for body via `--font-body`. Tight letter-spacing (`-0.02em`) on large headings, line-height ~1.7 on body.
- **Gradients:** Layer multiple gradients (linear + radial) for depth. The site uses a fixed grain overlay; reuse it on new pages where appropriate.
- **Animations:** Only animate `transform` and `opacity`. Never `transition-all`. Use the eased curves declared in `tokens.css` (`--ease-out-expo`, `cubic-bezier(0.25,0.46,0.45,0.94)`). Sequence reveals with `transition-delay` rather than firing everything at once.
- **Interactive states:** Every clickable element needs hover, focus-visible, and active states. No exceptions.
- **Images:** Pair full-bleed photos with a vignette gradient overlay so foreground text is always legible. Use the existing `data-bg="light"` attribute on cream sections so the floating nav pill auto-inverts.
- **Spacing:** Use the spacing scale and section padding tokens declared in `tokens.css` (`--space-*`, `--pad-section-y`, `--pad-section-x`). No random pixel values.
- **Depth:** Surfaces follow a layering system (`--bg-base` → `--bg-surface` → `--bg-elevated`). Don't park everything on the same z-plane.

## Hard Rules
- Do not add sections, features, or content not in the reference.
- Do not "improve" a reference design — match it.
- Do not stop after one screenshot pass.
- Do not use `transition-all`.
- Do not invent brand colours — use the palette in `tokens.css`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [armani-44/Design-Recruitment-Draft](https://github.com/armani-44/Design-Recruitment-Draft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
