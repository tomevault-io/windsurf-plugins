---
trigger: always_on
description: The imperative rules for working in this repo, plus the contributor-only gotchas that aren't obvious from reading the code. Public APIs and their contracts live in `docs/`; this file is the rulebook and cheatsheet for working *inside* the codebase.
---

# OpenStation — agent + contributor guide

The imperative rules for working in this repo, plus the contributor-only gotchas that aren't obvious from reading the code. Public APIs and their contracts live in `docs/`; this file is the rulebook and cheatsheet for working *inside* the codebase.

## Hard rules

### Never hand-edit JS in `assets/js/`

**`assets/js/*.js` is build output. Treat it as if it were `dist/`.**

Every built JS bundle has a TypeScript source under `src/`. The active build targets (and their TS entries) are listed in `package.json` under the `build:*` scripts; `npm run build` runs them all. The actual entry file for each target is in `vite.config.js`, selected by the `OPENSTATION_TARGET` env var.

Two hand-written files are the exception and stay tracked in git: `assets/js/admin-bar.js` and `assets/js/media-library-enhanced.js` (see the re-includes in `.gitignore`) — edit those directly; everything else under `assets/js/` is build output.

Process for any JS change:

1. Edit only TS files under `src/` (and CSS under `assets/css/` if styling, that's not built).
2. Run `npm run build` (or the per-target script).
3. Run `npm run lint` (or `lint:fix`), must pass on EVERYTHING you touched.
4. Run `npm run test:js`, must stay green.
5. Run `npm run typecheck`, must stay green.

If you ever find yourself reaching for `assets/js/*.js` directly, stop and write the TS instead. Hand-edited JS is overwritten by the next `npm run build` and produces no TS-checked types, a silent class of bug.

**Lint scope:** `npm run lint` runs on `src/**/*.ts` only. Test files under `tests/vitest/` aren't in the lint config (typescript-eslint project doesn't include them); rely on `npm run typecheck` + `npm run test:js` to catch issues there.

### The palette lives in `variables.css` — one declaration, one owner

The shell wears the [OpenStation brand](https://nuriapenya.github.io/open-station-brand/), and it wears it as **token declarations in `assets/css/variables.css` and nowhere else**. Void as the base, Obsidian for surfaces, Pulse and Nebula for identity moments, Sirius and Starlight for contrast, the Shade ramp for text hierarchy and lines.

**The palette is scoped to `body.os-active`, never `:root`.** `variables.css` is a dependency of `chromeless.css`, so it also loads inside every iframe window — a real `wp-admin` document. On `:root` the palette would repaint WordPress's own UI in there, and `--wp-admin-theme-color` alone would move Core's primary buttons, links and focus rings across every admin screen. Iframe documents carry `os-chromeless`, match nothing, and render on the fallback literals. **An admin page in a window looks exactly as it does outside one — that is a promise, and `tests/vitest/brand-palette.test.ts` holds you to it.**

Three rules follow from that, and all have tests:

1. **Restyling means changing a token's value in `variables.css`**, not adding a rule in a feature stylesheet. A colour declared next to the thing it paints is out of reach of the palette *and* of every desktop theme — including Legacy, the way back to the pre-brand look.
2. **Every consuming rule keeps reading `var( --token, <literal> )`**, and that literal stays the pre-brand WordPress-admin value. It is the floor if the stylesheet fails to load, and it is what the Legacy snapshot collected. Never "tidy" a fallback away.

**The failure mode to watch for after a palette change** is a chain that now means something else: a fill resolving to a 10%-alpha wash, or a base and its hover state — declared in two different rules, distinguished only by their fallback literals — collapsing onto the same value once the shared token is declared. `<os-button>`'s ghost/secondary hover did exactly that. When a surface stops reacting to the pointer, check whether both states resolve through the same palette token, and declare the second one.

### The holographic layer lives in `src/ui/holo.ts`

**"Holographic" is a moment, not a skin, and there is exactly one module that decides what it means.**

The brand ships five mesh gradients with one instruction attached — *"meshes reserved for hero surfaces"* — and `src/ui/holo.ts` is how a control gets to be one. It exports six `css` fragments (`holoTokens`, `holoFill`, `holoSheen`, `holoEdge`, `holoField`, `holoCheck`, `holoDrift`, plus the `holo` barrel) that components interpolate into their own styles. The meshes themselves are transcribed stop-for-stop from the brand SVGs into `--os-mesh-*` in `variables.css`; they are gradient stacks rather than `url()`s because `background-position` can slide a gradient, and that slide *is* the effect.

Three rules, all with tests:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WordPress/openstation](https://github.com/WordPress/openstation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
