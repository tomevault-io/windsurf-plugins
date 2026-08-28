---
trigger: always_on
description: Desktop theming app. **Wails 2** (Go backend) + **Svelte 5** frontend (TypeScript, Tailwind v4). Extracts color palettes from wallpapers and applies cohesive themes across a Linux desktop (Omarchy-friendly, works standalone).
---

# Aether — agent notes

Desktop theming app. **Wails 2** (Go backend) + **Svelte 5** frontend (TypeScript, Tailwind v4). Extracts color palettes from wallpapers and applies cohesive themes across a Linux desktop (Omarchy-friendly, works standalone).

## Detailed project docs (outside the repo)

Architecture decisions, design rationale, flow diagrams, ADRs, feature specs, and historical context live at:

```
~/Documents/bjarne/projects/Aether
```

**Read that folder** before making non-trivial changes or when the user asks "how does X work". **Write new detailed docs there** — the repo's `docs/` folder is reserved for user-facing documentation (CLI, installation, custom apps, templates). Content in `~/Documents/bjarne/projects/Aether` is deliberately outside git.

See the `aether-docs` skill (`.claude/skills/aether-docs/SKILL.md`) for the full convention.

## Commands

From repo root (requires Go + `wails` CLI + Node):

- `make dev` — live-reload dev build
- `make build` — production build (`build/bin/aether`)
- `make test` — `go test ./internal/... ./cli/...`
- `make install` — system install (Linux: copies to `/usr/bin/`; macOS: `/Applications/`)

Inside `frontend/`:

- `npm run check` — svelte-check type-checking (run before committing frontend code)
- `npm run dev` — Vite dev server (usually invoked by `wails dev`)

Pre-commit hook runs **prettier** on changed JS/TS/Svelte files and **gofmt** on Go. Don't bypass it.

## Layout

```
app.go                  # Wails app struct; ALL exported methods auto-bound to frontend
main.go                 # entry point
internal/
├── extraction/         # palette extraction (OKLab median-cut)
├── theme/              # theme state, applier, format classifier (IsImageFile)
├── wallpaper/          # file scanning, image thumbnails
├── blueprint/          # saved theme snapshots
├── color/              # color math (RGB/OKLab/HSL/Adjustments)
├── template/           # per-app template engine
├── wallhaven/          # wallhaven.cc API client
├── batch/              # batch palette generation
├── favorites/, omarchy/, platform/
frontend/
├── src/
│   ├── App.svelte              # top-level router
│   ├── app.css                 # Tailwind + theme tokens (light-mode class flips them)
│   ├── lib/
│   │   ├── components/         # UI (editor/, sidebar/, color-picker/, blueprints/, wallhaven/, ...)
│   │   ├── stores/*.svelte.ts  # reactive state (Svelte 5 runes, .svelte.ts extension required)
│   │   ├── utils/              # color.ts, canvas-filters.ts, keyboard.ts, debounce.ts
│   │   ├── constants/colors.ts # ANSI / extended color labels
│   │   └── types/theme.ts      # shared TS types + DEFAULT_PALETTE / DEFAULT_ADJUSTMENTS
└── wailsjs/go/         # generated bindings — DO NOT HAND-EDIT EXCEPT AS LAST RESORT
```

## Svelte 5 conventions

- Stores live in `*.svelte.ts` files and expose `getX()` / `setX()` functions that read/mutate module-scoped `$state`. Never export the state directly — callers use getters so TS can see the boundary.
- Components use runes: `$state`, `$derived`, `$props`, `$effect`. No `writable/derived` from `svelte/store`.
- **`$effect` dependency tracking pattern**: to force an effect to track non-reactive-read values, touch them:
  ```ts
  $effect(() => {
      const _ = JSON.stringify(points);   // deep mutation tracker
      const __ = histogram.length;
      const ___ = getLightMode();
      draw();
  });
  ```
  Throw-away `_` / `__` / `___` names are the established pattern (see `CurvesEditor.svelte`, `WallpaperEditor.svelte`).
- Single-use `$derived` is fine to inline. Multi-use or non-trivial expressions get a named derived.

## Theming & styles

- Theme tokens in `frontend/src/app.css` under `@theme { ... }` (dark) and `:root.light-mode { ... }` (overrides). Tokens: `bg-bg-primary/secondary/surface/elevated/hover`, `text-fg-primary/secondary/dimmed`, `border-border/border-focus`, `text-accent`, `bg-accent-muted`, `text-destructive/success/warning`.
- Light mode toggles via `document.documentElement.classList.add('light-mode')` — tied to the `getLightMode()` store and applied by `ActionBar.svelte`'s `handleApply`.
- **Overlay coloring rule**: UI that sits on top of the wallpaper image uses fixed scrim colors (`bg-black/60..80` + `text-white`) because image content is arbitrary. UI that sits on top of app chrome uses theme tokens (`bg-bg-secondary` + `text-fg-primary` + `border-border`) so it inverts correctly in light mode. Never `text-fg-primary` on a black scrim — it goes invisible in light mode.
- Canvas rendering: canvas can't inherit CSS vars. Read `getLightMode()` in your `draw()` and branch on ink color (see `CurvesEditor.svelte`'s `ink(alpha)` helper).
- Tooltips: use the native `title=` attribute (pattern in `WallpaperHero.svelte`, `HeaderBar.svelte`). No custom tooltip component.
- `border-radius: 0` is enforced globally (see `app.css`). Don't fight it.

## Wails bindings gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omacom/aether](https://github.com/omacom/aether) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
