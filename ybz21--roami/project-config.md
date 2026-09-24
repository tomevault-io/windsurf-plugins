---
trigger: always_on
description: **This file is the single source of truth for agent rules in this repository** — Codex, Claude
---

# Agent Instructions

**This file is the single source of truth for agent rules in this repository** — Codex, Claude
Code, and anything else editing this repo. They are requirements, not suggestions.

Codex reads `AGENTS.md` directly; Claude Code reads [`CLAUDE.md`](CLAUDE.md), which is a thin
pointer that imports this file with `@AGENTS.md`. Add or change a rule **here**, never in a
tool-specific copy — two copies drift, and then the two agents work from different rulebooks.

# Design System

All Web UI work — layout, color, typography, spacing, component selection, UX behavior — must
follow [`docs/development/ui-design-system.md`](docs/development/ui-design-system.md). Use the
tokens it defines (`--fs-*` type, `--r-*` radius, `--sp-*` spacing, `--accent*` / `--ok*` accents)
and the existing antd/component patterns in `frontend/src/`. Don't invent a new color value, font
size, radius, or spacing step when a documented one already covers the role. Frontend structure
follows [`docs/design/web/04-frontend.md`](docs/design/web/04-frontend.md) unless a newer pattern
already exists in code.

## Tokens Only: No Raw Values

The ladders are fixed: type `--fs-micro/meta/sm/body/lg/title`, radius `--r-xs/sm/card/pill`,
spacing `--sp-1`…`--sp-5`, accent `--accent` / `--accent-solid` / `--accent-soft` /
`--accent-border`, success `--ok` / `--ok-solid` / `--ok-soft` / `--ok-border`.

NEVER hardcode a design value that has a token. No `#58a6ff` / `#3fb950` in components — there is
exactly one blue and one green site-wide, and antd's `darkAlgorithm` derives its own shade from
the seed, so a copied hex is always a step off what antd draws next to it. No off-ramp font sizes
(10.5 / 11.5 / 12.5), and nothing below 12px outside eyebrow/badge text.

Top-level pages use `.tt-pagehead` (eyebrow + title + one line) for their header. Don't hand-roll
a title row per page.

## Clickable Things Are Buttons

An `<a onClick>` with no `href` is not keyboard-focusable and looks like text — it fails both
discoverability and a11y. Row-end actions use the quiet ghost button `.tt-act` (bare border at
rest, brightens on hover, `.danger` / `.ok` modifiers for destructive and confirming actions).
Reach for `<button type="button">` by default; `<a>` only when it really navigates.

## Breakpoints: One Entry Point

`useLayout()` is the only way to read layout size. NEVER read `window.innerWidth` and never call
`matchMedia` yourself. Pure style differences key off `data-size` / `data-pointer` on `<html>`.
Under a coarse pointer, hit targets are ≥44px — grow the hit area with a pseudo-element, not the
visual size, and overshoot the gap between neighbouring icon buttons by 4px (3px still misses at
dpr 2.75).

## Hover Belongs to the Mouse

Every `:hover` rule is written `:where(html[data-pointer="fine"]) X:hover`. Touch has no
`mouseleave`, so a bare `:hover` sticks to the last thing a finger touched: the previously tapped
button stays lit and a row of buttons flickers as you work down it. `:where()` adds no
specificity, so the gate changes nothing about the cascade. Anything hidden until hover
(`opacity: 0` copy/close/download buttons) needs a coarse rule that keeps it visible, or the phone
can never reach it. `npm run hover:check` (`frontend/scripts/hover-scope-audit.mjs`) enforces both
halves and runs inside `npm run build`.

Tooltips are mouse furniture too: under a coarse pointer `.ant-tooltip` is hidden site-wide,
because a tooltip opened by a long press never gets a `mouseleave` and its overlay then eats the
next tap. An icon-only button therefore carries its name in `aria-label`, not only in a Tooltip.

## The Document Never Scrolls

`html, body { overflow: hidden }` — the app is a fixed-height shell and every page scrolls inside
its own container. One pixel of document overflow summons a 10px document scrollbar, which takes
10px off the workspace and slides the canvas and the terminal dock sideways; on a 150%-scaled
display the fractional rounding flips it back and forth with every burst of terminal output, and
buttons move out from under the cursor between mousedown and mouseup.

## Icons: SVG Only

NEVER add an emoji (🔄 📎 🤖 …), NEVER use a text symbol as an icon
(`✕ × ✓ ▾ ▸ ← → ↑ ↓ ⏎ ■ ● ◆ ⚠ ⎇ ⧉ ＋ …`), and NEVER draw a one-off inline SVG in a component.

- Icons come from `frontend/src/icons.tsx` (24×24 viewBox, `currentColor`, 1.8 stroke, round
  caps). `file-icons.tsx` and `git/parts.tsx` hold the file-type and Git sets in the same style;
  new icons default to `icons.tsx`.
- The same action uses the same icon everywhere. A close button that is `✕` in one place and `×`
  in another is worse than an ugly icon.
- Icons are NEVER baked into i18n strings (`'＋ Add task'`). Copy holds words only; the call site
  passes `icon={<PlusIcon />}`.
- Status dots and swatches are drawn (`border-radius:50%`, `<Swatch />`), not typed (`●`, `■`).
- Brand marks (`AgentLogo`: the Claude and OpenAI/Codex logos) are the one place our icon rules
  step aside. Use the official path data and the official color — `--brand-claude` /
  `--brand-codex` — never redraw a logo and never re-tint one to our accent. A brand mark also
  gets no chrome: render it bare (`.tt-agentmark`), not boxed in an `ant-tag`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ybz21/Roami](https://github.com/ybz21/Roami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
