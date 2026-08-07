---
trigger: always_on
description: - Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
---

# Obsidian community plugin

## Project overview

- Target: Obsidian Community Plugin (TypeScript → bundled JavaScript).
- Entry point: `src/main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, and optional `styles.css`.

## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required for this sample - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required for this sample - `esbuild.config.mjs` and build scripts depend on it). Alternative bundlers like Rollup or webpack are acceptable for other projects if they bundle all external dependencies into `main.js`.
- Types: `obsidian` type definitions.
- **codemirror-vim fork**: The plugin uses a fork of `@replit/codemirror-vim` at `~/Repos/codemirror-vim`. All core vim behavior changes go in the fork's `src/vim.js`. The fork has its own test suite (1882 browser tests) and Neovim golden comparison infrastructure (756 golden cases, 476 pass, 280 known deviations). The fork includes an operator-prefix shadow resolver (`operatorshadowtimeout` option, default 1000ms) that disambiguates operator-pending motions vs multi-key actions (e.g., flash `s` motion vs surround `s<character>` action) by deferring to partial matches with a configurable timeout fallback. The fork exposes `setLivePreviewField(field: StateField<boolean>)` so the host plugin can provide Obsidian's `editorLivePreviewField` — the fork's frontmatter properties navigation (`focusBefore` in `findPosV`) is gated on this field to avoid intercepting cursor movement in source mode. The fork exposes `setPropertiesSource(fn: () => boolean)` so the host plugin can indicate when frontmatter is rendered as source text in Live Preview (Obsidian's "Properties in document" = "Source") — the frontmatter interception is also skipped when this callback returns `true`, preventing the cursor from getting stuck on a hidden `.metadata-container`. The fork exposes `setCursorSuppressed(suppressed: boolean)` and per-view overrides (`setCursorSuppressedForView`, `clearCursorSuppressedForView`, `isCursorSuppressedForView`) so the host plugin can suppress all cursor DOM rendering when using its own canvas-based animated cursor. Insert-mode surround (`<C-G>s`/`<C-G>S`) inserts both delimiters up front (matching vim-surround) and supports full dot-repeat — the fork stores `_surroundInsertChar`/`_surroundInsertNewline` on `lastInsertModeChanges` and replays them via `replaySurroundAwareInsert` inside `repeatLastEdit`, exceeding both vim-surround and nvim-surround where insert-mode surround dot-repeat is broken. The fork exposes `feedKeys(cm, keys, { noremap })` for programmatic key injection with correct noremap semantics — delegates to `doKeyToKey` with the internal `noremap` flag and `keyToKeyStack` recursion protection. Used by expr mapping result feeding. The fork exposes `undefineEx(name)` to remove ex commands registered via `defineEx` — cleans both `exCommands` and `commandMap_` prefix entries. Used by the plugin's vimrc soft-reload to clean up stale `exmap` handlers. The fork uses a CM6 `eventObservers.keydown` (DOM event observer) instead of `eventHandlers.keydown` for vim's keydown processing — in CM6's dispatch order, observers run before handlers, guaranteeing vim fires first regardless of `Prec` ordering or plugin load order. The fork exposes `setKeyInterceptActive(active: boolean)` so the host plugin can suppress the observer during modal key-interception states (flash labels, EasyMotion labels, hint mode).
    - **IMPORTANT: dependency URL in `package.json`**: The `@replit/codemirror-vim` dependency MUST point to `https://github.com/saberzero1/codemirror-vim.git` (the remote URL) before committing. During local development, use `npm install ~/Repos/codemirror-vim` for fast iteration, but **always switch back to the HTTPS URL before committing** — `file:../codemirror-vim` breaks CI, the community scanner, and anyone cloning the repo. Check `git diff package.json package-lock.json` before every commit to verify no local path leaked.
- **fengari fork**: The plugin uses a [browser-only fork of fengari](https://github.com/saberzero1/fengari) at `~/Repos/fengari` for the Lua 5.3 runtime. The fork strips all Node.js dependencies (`fs`, `child_process`, `os` module, `readline-sync`, `tmp`) and ships with zero runtime dependencies (`sprintf-js` replaced with a custom `luaSprintf` formatter). Integers are widened from 32-bit to 53-bit (`math.maxinteger = 9007199254740991`); bitwise operations remain 32-bit (JS platform limitation). `string.packsize("j")` returns 8. `__gc` metamethods on userdata are supported via `FinalizationRegistry` (finalization order unspecified; tables not finalized; drain at `luaD_pcall` return, `collectgarbage("collect")`, and `lua_close`). `collectgarbage()` returns safe no-op values for all modes (was `luaL_error`). The plugin installs a `lua_atnativeerror` handler so native JS errors (TypeError, etc.) produce extractable Lua error strings instead of being lost. See the fork's [`DIFFERENCES.md`](https://github.com/saberzero1/fengari/blob/master/DIFFERENCES.md) for the full list of changes from upstream.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saberzero1/motions](https://github.com/saberzero1/motions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
