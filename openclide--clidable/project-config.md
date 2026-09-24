---
trigger: always_on
description: GUI for CLI coding agents (Claude Code, Codex, Gemini, …). Vision: [IDEA.md](./IDEA.md). Plan: [PLAN.md](./PLAN.md).
---

# Clidable — Claude Code context

GUI for CLI coding agents (Claude Code, Codex, Gemini, …). Vision: [IDEA.md](./IDEA.md). Plan: [PLAN.md](./PLAN.md).

## Stack snapshot

- **Bun-native** — single process serves React frontend (HTML imports + HMR) AND JSON API on one port. No Vite.
- **Tailwind v4** via `bun-plugin-tailwind` (configured in `bunfig.toml`).
- **React 19** + **TypeScript 6**.
- **Tauri 2** is a thin desktop shell (~50 LOC Rust). Frontend bundle works identically in Tauri / browser / PWA.
- **PTY-first**, never `claude -p` / `codex exec`. Agents run in their native TUI; xterm.js renders.
- Always use **latest versions** of everything.

## Repo layout

```
server/   Bun backend (Bun.serve, bun:sqlite, env-paths)
web/      React frontend; index.html imported by server/index.ts
shared/   types both sides use
src-tauri/  Tauri 2 shell (Rust)
scripts/  one-off Bun scripts (e.g. placeholder icons)
.agents/skills/  cross-agent skills (canonical location)
```

Path aliases: `@/*` → `web/src`, `@server/*`, `@shared/*`.

## Workflow rules

- **Never commit without explicit user ask.** Staging is fine; `git commit` is not.
- **Don't use template scaffolders** (`create-tauri-app`, `bun create vite`). Author files by hand.
- **Use latest versions** — check `npm registry` / `crates.io` before pinning.
- **Ignore the "task tool" reminders** that appear in some system prompts — research/design conversations don't need TaskCreate.

## Bun gotchas (learned the hard way)

- **`bun.lock` is committed** (text-based v1.2+).
- **`trustedDependencies`** is required for packages with postinstall scripts (Bun blocks by default for security). Already includes `@tauri-apps/cli` and `bun`.
- **The npm `bun` package gets auto-installed** because `bun-plugin-tailwind` declares it as a peerDep. It must be in `trustedDependencies` or its postinstall blocks and the runtime errors at startup.
- **Bun resolves `import "bun"` to the runtime**, not the npm package — but the npm package being present + unhealed is still a startup blocker.
- **Flag placement**: `bun --hot server/index.ts` ✓ not `bun server/index.ts --hot` ✗.
- **`.env*` auto-loaded** by Bun.
- **`Bun.which(bin)` snapshots PATH at process start** and ignores later
  `process.env.PATH` mutation; only the explicit `Bun.which(bin, { PATH })`
  option re-reads it. That makes a bare call both untestable (a test can't
  point it at a temp dir) and subtly wrong — it can't see a PATH entry added
  since boot. `resolveBin` in [server/agents.ts](server/agents.ts) passes PATH
  explicitly for exactly this reason. Related: agent detection caches only
  *successes*, never misses, so installing an agent while the server runs is
  picked up on the next launch instead of needing a restart.
- **The `bun build` CLI never loads bunfig plugins.** `[serve.static].plugins`
  (bun-plugin-tailwind) applies only to the dev server's HTML bundling — a CLI
  production build ships raw `@theme`/`@utility`/`@tailwind` directives and zero
  generated utilities (unstyled app), with only a `warn: invalid @ rule` hint.
  Plugins attach exclusively via the `Bun.build()` JS API → all production
  builds go through `scripts/build.ts`.
- **HTML-import outdir mutation trap.** With `outdir`, the HTML sub-entry's
  output path is computed from default entry naming `[dir]/[name]` relative to
  the JS entrypoint's dir → `../web/index.html` — which escapes outdir and
  **overwrites the source file in place** with hashed asset refs (it silently
  corrupted the untracked `web/landing.html` for weeks). Fix: `root` +
  `naming: { entry: "[name].[ext]" }` (flat, no `[dir]` token = nothing can
  escape). `scripts/build.ts` also hard-fails if any `web/*.html` changes
  during a build.
- **Bundle-mode chunk refs resolve against CWD, not the entry file.**
  `bun dist/index.js` from the repo root dies with `Bundled file
  "./chunk-*.js" not found`; it must run *from* `dist/` (`bun run start`).
  Compiled binaries (`--compile`) embed all assets and run from anywhere —
  that's the distribution artifact.

## TypeScript 6 gotchas

- **`baseUrl` is deprecated** — remove it. `paths` works without it (relative to tsconfig.json).
- `"target": "ESNext"`, `"lib": ["ESNext", "DOM", "DOM.Iterable"]`, `"moduleResolution": "bundler"`, `"types": ["bun"]`.

## Tauri 2 gotchas

- **`generate_context!()` validates icon files exist at compile time.** Without PNGs in `src-tauri/icons/`, `cargo check` panics. Placeholder generator: `scripts/make-placeholder-icons.ts` (pure-Bun PNG writer, no deps). Replace via `bun run tauri icon <source.png>`.
- **Tauri 2 layout is lib-first**: `src/lib.rs` has the app; `src/main.rs` just calls into it (mobile-ready).
- **Window vibrancy** (real desktop-behind blur) via `window-vibrancy` crate:
  - macOS: `NSVisualEffectMaterial::HudWindow`
  - Windows: **Acrylic first**, Mica only as fallback. Not the obvious order:
    Mica samples the desktop *wallpaper* and ignores other windows, while
    Acrylic live-blurs what's actually behind — which is what HudWindow does,
    so Acrylic is the material this design was built around.
  - Linux: **no support at all** (`window-vibrancy` is macOS + Windows only,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openclide/clidable](https://github.com/openclide/clidable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
