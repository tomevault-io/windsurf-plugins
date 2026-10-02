---
trigger: always_on
description: OpenCode v2 TUI plugin: a prompt-footer status line (context, cache, streaming
---

# opencode-status-line

OpenCode v2 TUI plugin: a prompt-footer status line (context, cache, streaming
speed, cost, elapsed time, uncommitted changes). `README.md` is the friendly
introduction; `MANUAL.md` is the exhaustive user reference.

## What is unusual here

- **No build step.** `package.json` publishes the source (`./tui` →
  `src/tui.tsx`) and OpenCode transpiles it on load; don't add a bundler, and
  keep `src/` the only shipped surface — `bun.lock` exists solely so the
  typecheck job resolves the host and Solid types reproducibly. `bun test` (or
  `bun test test/rate.test.ts` for one module) needs no install;
  `bun install && bun run typecheck` checks types, including `src/tui.tsx`,
  which no test imports; `npm run check:pack` checks the package surface. CI is
  `.github/workflows/ci.yml`; releases are `.github/workflows/publish.yml`, not
  a laptop. `main` takes pull requests only: green CI plus one approving
  review (the maintainer bypasses for their own work).
- **`src/tui.tsx` is the plugin entry**, loaded straight from this checkout —
  the live host's `~/.config/opencode/cli.json` lists the directory. OpenCode
  transpiles the TSX on load and hot-reloads on save, so a broken save shows up
  in the running TUI immediately. The plugin cannot run standalone; verify
  runtime changes by hand in a session (`/opencode-status-line` opens the stats
  dialog).
- **The root `tui.tsx` is a load-bearing shim** re-exporting `src/tui.tsx`. The
  running 2.0.16 loader resolves a directory plugin through `<dir>/tui` before
  checking `package.json` exports; deleting the shim drops the plugin from the
  live TUI. npm consumers resolve `@rashidrazak/opencode-status-line/tui` through exports to
  `src/tui.tsx` instead.
- Internal imports carry `.ts`/`.tsx` extensions (`./rate.ts`); the host
  resolves them verbatim, so keep that style.

## Layout

| Path | Role |
| --- | --- |
| `src/tui.tsx` | Entry: event wiring, slot render, command. The only file importing `@opencode/plugin`, `solid-js`, or host APIs. |
| `src/rate.ts` | Speed maths (sliding window, turn fold, calibration, history) and the `USAGE_LABELS` icon/word sets. Pure. |
| `src/render.ts` | Gauge and context-bar geometry, run cutting and wrapping for narrow widths. Pure. |
| `src/format.ts` | Token / money / duration formatting. Pure. |
| `src/diff.ts` | Uncommitted-change totals from the host's VCS status, and the diff segment's cache policy. Pure. |
| `src/palette.ts` | The bundled colour palettes and palette/override resolution. Pure. |
| `src/config.ts` | JSON config loader; pure except an injectable `read`. |
| `test/*.test.ts` | One per pure module. |
| `tui.tsx` | Root shim re-exporting `src/tui.tsx`; see above. |

Keep new logic in the pure modules so it can be tested without a terminal.

## Host-API traps (each cost a TUI restart to learn)

### Entry rules — every edit to `src/tui.tsx`

- Register the keymap layer inside the `app` slot's `render`, never directly in
  `setup`: v2 keeps the keymap provider in the component tree, so a `setup`
  registration throws `Keymap.Provider is missing` and kills the plugin. Give
  the layer `mode: "global"`: a layer that names no mode is pinned to `base`,
  and v2 pushes `autocomplete` while the slash list is open and `modal` while a
  dialog is, so a mode-less command is unreachable in the two places it would
  be found.
- Build rendered parts inside a `createMemo`. `Show` calls its children
  untracked, so a plain array is evaluated once and the line never repaints.
- Route every event handler through `safely`; an uncaught throw inside one can
  kill the plugin generation, and a half-saved file has done exactly that.
- Saving any `src/` file the entry imports hot-reloads the plugin: the module is
  re-imported and module scope comes back empty, which used to blank the meter
  segment mid-turn on every save. State that must outlive a generation lives on
  `globalThis` (`sharedMeters` in `src/tui.tsx`). Touching `README.md`,
  `MANUAL.md` or `test/` does not reload; the `src/` imports do.
- A 250 ms ticker repaints only while a stream is active; a 1 s heartbeat keeps
  the elapsed timer and held figures repainting when nothing streams. Stop
  both in the cleanup function.

### Speed maths and the session record

- The meters are process-scoped: a session met without one — a resume, or a
  reload before any delta — seeds its last figure from the stored messages
  (`seedMeter` in `src/tui.tsx`; `recordedSteps` / `restoreFinal` in
  `src/rate.ts` hold the logic and its rationale). The seed is lazy — the meter
  branch in `usageRows` kicks it when the map misses — and retries on later
  paints, because the host hydrates messages page by page and a partial page
  must not fold as a turn; one `message.sync` per session forces the full fetch.
  It sets `final` plus a resting zero `sliding` (empty gauge, `↯ 0.0`) — never
  `turn`, which a later step would absorb — and the rebuilt figure is close to,
  not bit-identical with, the live one: accept the ~1% tolerance rather than
  chase it with tool-time heuristics. `time.streamed` is a stream-finalisation
  stamp, not a first token (see `firstTokenAt`). Window samples and the
  statistics are memory-only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rashidrazak/opencode-status-line](https://github.com/rashidrazak/opencode-status-line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
