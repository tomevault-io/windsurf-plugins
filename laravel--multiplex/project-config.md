---
trigger: always_on
description: Tabbed TUI for running multiple CLI commands simultaneously. Built with Ink (React for terminals), TypeScript, Commander.
---

# @laravel/multiplex

Tabbed TUI for running multiple CLI commands simultaneously. Built with Ink (React for terminals), TypeScript, Commander.

## Scope

This is for the commands you run **while you are actively developing** — a dev server, a queue listener, a watcher, a build in watch mode — for the length of a working session, with you sitting in front of it.

It is not a CI runner and not a process supervisor. It works fine in CI (inline mode exits with a real code, `--json` is parseable) and nothing should gratuitously break there, but CI is not what the defaults are tuned for. Neither is unattended operation: there is no daemonising, no log rotation, no restart policy worth the name, and buffers are capped for memory rather than kept for later.

Weigh proposals against the working session. "What if this runs for three days" and "what if nobody is watching" are usually the wrong questions here; "what happens when I save a file and something crashes" is the right one.

## Commands

- `pnpm run build` — TypeScript compile to `dist/`
- `pnpm test` — run tests via `tsx --test`
- `pnpm run dev` — run from source via tsx
- `pnpm run check` — biome lint + format (auto-fix)
- `pnpm run lint` — biome lint only

Always run build + test after changes.

## Architecture

- `cli.tsx` — CLI entry point. Parses argv with Commander, then hands off to `multiplex()` and exits with its code. Validation errors from `multiplex()` are reported through `program.error`.
- `multiplex.tsx` — the package's programmatic entry point (`main`/`exports`), exporting `multiplex(options)`. Validates options, then runs either `runTui()` or `runInlineMode()`. Each has its own `shutdown()`, wired to normal exit, failure, and SIGINT/SIGTERM/SIGHUP/SIGQUIT via the shared `installTeardown`.
- `app.tsx` — Main React/Ink component. Sidebar + content panes, keyboard input handling, search overlay, stream/tab mode toggle.
- `supervisor.ts` — The process layer, with no React and no rendering in it: spawn, line splitting, restart accounting, auto-restart with a 5-attempt limit, and settle detection. Emits events; both front ends drive it.
- `use-processes.ts` — Thin React wrapper over the supervisor. Turns its events into the TUI's per-command buffers, stream lines, `failedProcs` state and desktop notifications.
- `inline.ts` — The non-TUI front end. Same supervisor, but each line is written to stdout/stderr as it arrives, or serialised as NDJSON when `json` is set. Resolves with an exit code when everything has settled.
- `use-scroll.ts` — Hook for scroll state (offset tracking, page up/down, new-output indicator).
- `search.ts` — ANSI-aware search in two phases: `indexMatches` counts and locates matches across the whole buffer, `highlightLine` highlights a single line. Strips escape codes for matching, preserves them in output, highlights across ANSI boundaries.
- `args.ts` — The two input paths onto `CommandDef[]`: `parseCommandDef` for CLI positionals (incremental, one value at a time) and `normalizeCommands` for programmatic input (validates and fills in colors for the whole list at once).
- `color.ts` — Everything that knows what a color is: the palette, the accepted names (ansi-styles') and their approximate RGB (ours), `normalizeColor` as the one validation gate, `colorOpen` for the raw SGR sequence and `contrastText` for legible text on a filled background.
- `util.ts` — Shared constants and helpers (hex-to-RGB, timestamp formatting, dynamic sidebar width).
- `types.ts` — Shared type definitions.

## Design decisions

- **A missing TTY is a fallback, not an error.** Piping, redirecting or running from CI drops to inline mode instead of refusing to start, because those are the cases where people most want a process runner. There is deliberately no flag to restore the old hard failure: it would only ever fire where nobody wanted a TUI anyway.
- **The supervisor emits events, not formatted text.** It would have been shorter to keep building the `Process exited with code 1` strings where the exit is handled, but then JSON output would be a scrape of the human output and every renderer would inherit the TUI's wording. Each front end decides how an event reads.
- **`onData` exists alongside `onLine` for one reason.** The tabbed view shows raw chunks so a partial line — a progress bar, a prompt — is visible before its newline lands. Everything line-oriented (the stream buffer, inline mode, JSON) uses `onLine` and waits for the newline. Dropping `onData` and rebuilding the tab buffer from lines would make unterminated output invisible until the process exits.
- **Inline mode's defaults are not the TUI's**, in two places only: it exits when the last command does, and it exits with the first permanent failure's code. Everything else, auto-restart included, is identical — a mode-dependent default means the same command line behaves differently depending on whether someone piped it, which is worse than either choice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laravel/multiplex](https://github.com/laravel/multiplex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
