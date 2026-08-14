---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this

Grimorio Engine is a pure CSS UI framework by ESC Labs with a PS1/CRT retro aesthetic. Zero external JS dependencies. Fonts are loaded from Google Fonts. The entire framework lives in one file: `packages/css/grimorio.css`.

`apps/showcase/index.html` is the **canonical component showcase** — it demonstrates every component and is the source of truth for all design patterns and usage. All JavaScript lives in `packages/core/grimorio.js` (the editable source; `js/grimorio.js` at repo root is a generated mirror for distribution — see "Two distribution channels").

**System requirements:** Node ≥18.0.0, npm ≥9.0.0

### Repository documentation map

Prose docs are in Spanish; this file is the developer entry point. When a task touches identity, tokens, or AI-generation, read the relevant doc rather than guessing:

| Doc | Read it when |
|---|---|
| `COMPONENTES.md` | You need the full utility/component catalog with examples — the AI manifest and de-facto component reference. |
| `ESC-LABS-PS1-FRAMEWORK.md` | The task touches brand identity, voice, typography, or the non-negotiable visual rules (source for "Identity constraints" below). |
| `USO-CON-IA.md` | Explaining or wiring up how an external AI generator consumes the framework (system-prompt + verification loop that pairs with `COMPONENTES.md`). |
| `PROYECTO.md` | You need the component/roadmap status board (what's done vs. partial). |
| `CHANGELOG.md` | Cutting a release or checking what changed between versions. |
| `README.md` / `packages/css/README.md` | Consumer-facing install/usage (GitHub-install + npm package page). |
| `llms.txt` | Root discovery index (llmstxt.org) linking the above. |

## Running / Developing

**Zero build step during development.** Edit `packages/css/grimorio.css` directly and see changes instantly with any static server. Choose one:

### Quick start (no npm):
```bash
python -m http.server 8080
# then open http://localhost:8080/apps/showcase/
```

### With npm:
```bash
npm install        # installs clean-css-cli devDependency (optional)
npm run serve      # npx serve . (any port) — then navigate to /apps/showcase/
```

### Before publishing:
```bash
npm run build      # minifies packages/css/grimorio.css -> packages/css/grimorio.min.css,
                    # mirrors both CSS files to css/ at repo root, and mirrors
                    # packages/core/grimorio.js -> js/grimorio.js (see "Two distribution channels")
npm run validate   # 9 invariants (see "Testing" below) — run before every commit
```

**Development workflow:** Open showcase in browser, edit CSS directly in `packages/css/grimorio.css`, refresh to see changes. No compilation or transpilation needed.

**Validation:** `.hintrc` configures HTMLHint validation (development only). The `no-inline-styles` rule is disabled by design — this policy is maintained manually in code review (see below).

## Common development tasks

**Add a new component:**
1. Create a new CSS section in `packages/css/grimorio.css` with the standard header pattern (see Architecture)
2. Use BEM naming in Spanish
3. Use CSS variables for all colors (never hardcode hex in components)
4. Add demo markup + example in `apps/showcase/index.html` under a `.separador` label
5. Test all three palettes (no class, `.cosmos`, `.crimson`)

**Update the showcase:**
- **Zero `style=""` anywhere** — `index.html` and all secondary pages are inline-style-free (verify with a grep for `style="` returning nothing). Solve every layout need with a utility or BEM class; if a value is missing, add the utility to `grimorio.css` first.
- Changes to `packages/css/grimorio.css` auto-reload in the browser; no build step needed
- Always add theme-switcher button to secondary pages (`data-tema` attribute + `#tema-flash` div required)

**Create a new secondary page (like login.html, contacto.html):**
- Copy boilerplate from existing secondary pages in `apps/showcase/html/`
- Update `menu-principal__item--active` on current nav item
- Secondary pages live at `apps/showcase/html/` (3 levels below repo root), so CSS and JS use a **`../../../` prefix** (`../../../packages/css/grimorio.css`, `../../../packages/core/grimorio.js`) — both link the editable source directly (zero-build dev), not the root mirrors. Images live at `apps/showcase/images/`, so they use `../images/`. Sibling pages use plain relative links (`contacto.html`).
- Paths to images use `../images/`

**Publish a release:**
1. Run `npm run build` — minifies `packages/css/grimorio.css` and mirrors both files into `css/` at repo root
2. Ensure `packages/css/grimorio.min.css` AND the root `css/grimorio.css` + `css/grimorio.min.css` mirror exist and are committed (the root mirror is what GitHub-install/CDN consumers get — see "Two distribution channels" below)
3. Tag the commit (e.g., `v2.0.1`)
4. Distribution via GitHub: `npm install github:ClaysterChief/GrimorioEngine#v2.0.1`

## Architecture

### Single-file CSS framework

`packages/css/grimorio.css` is organized into sections marked with this header pattern:

```css
/* ================================================

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClaysterChief/GrimorioEngine](https://github.com/ClaysterChief/GrimorioEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
