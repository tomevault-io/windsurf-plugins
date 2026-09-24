---
trigger: always_on
description: Client-side video player — play any video file in the browser without a server.
---

# playsvideo

Client-side video player — play any video file in the browser without a server.

Do NOT use auto-memory (`~/.claude/projects/.../memory/`). All project context lives in this file.

## Package Manager

Uses **pnpm workspaces**. The root package is the core library (`playsvideo`). The `app/` directory is a separate workspace for the React media player.

```bash
pnpm install           # install all workspace deps
pnpm -w run <script>   # run a root workspace script
pnpm --filter app dev  # run the media player dev server
```

## Green Gates

Before considering work done, all of these must pass:

```bash
pnpm -w run typecheck    # tsc --noEmit
pnpm -w run test:unit    # vitest run tests/unit (fast, no fixtures needed)
pnpm -w run lint         # biome lint .
pnpm -w run format       # biome format --write . (then verify no unstaged changes)
```

Integration tests (`pnpm -w run test:integration`) require test fixtures in `tests/fixtures/`.

## Project Structure

- `src/pipeline/` — core pipeline modules (demux, mux, segment plan, audio transcode, codec probe)
- `src/adapters/` — platform adapters (node-ffmpeg, node-ffprobe, wasm-ffmpeg)
- `src/engine.ts` — PlaysVideoEngine class (worker, hls.js, subtitles — no UI)
- `src/worker.ts` — browser web worker (demux + segment processing)
- `src/pwa-player.ts` — browser entry point (thin UI wiring to engine)
- `tests/unit/` — fast unit tests (no external dependencies)
- `tests/integration/` — tests requiring ffmpeg/ffprobe and test fixtures
- `tests/e2e/` — playwright browser tests
- `app/` — React media player (separate workspace, see below)

### Media Player App (`app/`)

React app with library management at `/app`. Separate workspace with its own `package.json`, `vite.config.ts`, `tsconfig.json`.

- `app/src/db.ts` — Dexie IndexedDB schema (library, directories, playlists, settings)
- `app/src/scan.ts` — File System Access API directory walker + IDB sync
- `app/src/hooks/useEngine.ts` — React hook wrapping PlaysVideoEngine lifecycle
- `app/src/pages/Library.tsx` — video library grid with folder picker
- `app/src/pages/Player.tsx` — video player page
- Uses `import { PlaysVideoEngine } from 'playsvideo'` via workspace link

## Key Conventions

- TypeScript with ES modules (`.js` extensions in imports)
- Biome for formatting and linting (not ESLint/Prettier)
- vitest for testing
- mediabunny for demux and mux (fMP4)
- hls.js for playback (custom `fLoader` for on-demand segments — no service worker)

## Documentation Workflow

Implementation tactical docs live under `docs/tactical/` and use zero-padded
numeric filenames. The next tactical created under this convention is
`000-{subject}.md`; after that, allocate the next unused number and add it to
`docs/tactical/README.md`. Completed tacticals are execution records, not the
source of continuing guidance.

Focused, living topic docs live under `docs/topics/`. Before working on a
continuing concern, look for and read its topic doc. Update it when work changes
the concern's status, contract, evidence, validation, gaps, or recommended next
direction. Create a focused topic when continuity across sessions or commits
would be valuable, but do not create one for every small standalone change or
broaden a nearby topic into a catch-all.

Adopt these conventions incrementally. Existing architecture, reference,
status, and implementation-plan docs do not need to move solely for
consistency. Architecture and reference docs own durable system shape and
external facts; topic docs own current truth for a focused continuing concern;
tactical docs own bounded implementation slices and execution records.

See `docs/tactical/README.md` and `docs/topics/README.md` for the local indexes
and templates. The root `topics.md` is separate: it registers exact `Topic:`
strings used to thread related commit series.

## Commit Message Guidance

Aim for a subject of at most 65 characters and strictly wrap commit bodies at
72 columns. Use a concise, result-oriented subject that remains scannable in
`git log --oneline`. Never mention Claude, AI, or any AI assistant in commit
messages; write them as human development history.

For non-trivial commits, include a concise synthesis of the originating request
or motivating observation and the key implementation direction. Preserve enough
context for a future maintainer to understand or approximately re-derive the
change without recording digressions, secrets, or a verbatim conversation.
Mechanical or small self-evident changes may use a one-line message.

When commits form a related series, append one or more exact
`Topic: <string>` trailers after the body. The first commit chooses the string;
later commits copy it verbatim so `git log --grep "Topic: ..."` finds the
series. Prefer the matching `docs/topics/` filename slug when the series
implements a documented topic. Use multiple trailers when needed and omit them
for standalone commits with no expected follow-up. Before starting a series,
scan root `topics.md` and register any new topic string there.

## Architecture

### ffmpeg.wasm

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kzahel/playsvideo](https://github.com/kzahel/playsvideo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
