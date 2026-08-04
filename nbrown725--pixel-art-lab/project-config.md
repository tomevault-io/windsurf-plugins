---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A local web app for benchmarking **any** OpenRouter model at agentic pixel art creation. It drives
[`pixel-mcp`](https://github.com/willibrandon/pixel-mcp) — an MCP server scripting a real Aseprite
install — through a tool-calling loop so the model draws, renders a preview, *looks at it*, and fixes
what's wrong.

Because the point is to *compare* models, anything that makes a model's job harder for reasons
unrelated to drawing (huge tool menus, undocumented API quirks, a two-step canvas creation dance) is
smoothed over by the server rather than left for each model to rediscover. That principle is why the
synthetic tools and the curated toolset exist; keep it in mind before exposing more raw surface.

## Commands

npm workspaces (`server`, `web`) at the root:

```sh
npm run dev          # server on :8787 + web on :5273 (vite proxies /api → 8787)
npm test             # vitest run in both workspaces
npm run typecheck    # tsc --noEmit in both
npm run build        # tsc -b (server) then tsc -b && vite build (web)
```

Single workspace / single test:

```sh
npm run test --workspace server
npx vitest run src/mcp/paths.test.ts --workspace server   # or cd server && npx vitest run paths
npx vitest run -t "rejects traversal" --workspace server  # by test name
```

Read back what a model actually did on a run (see "Run transcripts" below):

```sh
npm run transcript                    # the most recent run
npm run transcript -- <runId> --full  # full tool args/results; --reasoning adds reasoning text
npm run transcript -- --list          # runs, newest first
```

End-to-end smoke test against a real model (needs the server already running, costs money):

```sh
node server/scripts/smoke.mjs [model] [prompt]
# env knobs: PAL_BASE W H TURNS ITERS MAX_COST EFFORT
```

Check the MCP dependency is healthy before debugging a failing run:

```sh
./pixel-mcp/bin/pixel-mcp --health   # or wherever PIXEL_MCP_BIN points
```

## Requirements

Node 20+, Aseprite plus a built `pixel-mcp` binary with `~/.config/pixel-mcp/config.json` pointing at
the Aseprite executable, and `OPENROUTER_API_KEY` set. The key is read server-side only
(`server/src/config.ts`) and never reaches the browser — models are proxied through `/api/models` for
that reason. All tunables (budget ceilings, preview sizing, context-pressure thresholds, timeouts)
live in `config.ts`; prefer adding one there over hardcoding.

The server binds `127.0.0.1` explicitly, and that is not incidental: there is no authentication
anywhere, and the app proxies a key that spends money, starts runs on request and serves files out of
the run workspaces. On the default `0.0.0.0` all three belong to whoever shares the network. Making
it reachable from another machine means adding auth first, not widening the bind. Relatedly,
`/api/health` reports `ok` from whether the pixel-mcp binary is actually executable rather than from
the process being up — the first thing a fresh clone gets wrong is that path, and a spawn `ENOENT`
surfaced as run-end error text reads like the model broke rather than like setup is incomplete.

Environment comes from `.env` at the repo root (`cp .env.example .env`), loaded by the server's own
npm scripts via `--env-file-if-exists` — *if exists*, so a shell-exported key still works and a
missing file fails as `assertConfigured`'s clear message rather than an ENOENT. Node's parser does no
tilde expansion, so `~/...` in `.env` is a literal path; `PIXEL_MCP_BIN`, `RUNS_DIR` and `GALLERY_DIR`
are each resolved against the project root, so a relative value there means what it looks like
regardless of where the server was started from.

## Architecture

Request flow: browser posts a multipart form to `POST /api/runs` → `createRun` makes an isolated
workspace → the response returns `{runId}` **immediately** and `runAgent` runs detached → the browser
subscribes to `GET /api/runs/:id/events` (SSE). Events are buffered on the `Run` and replayed on
connect, so a late or reconnecting browser still sees the whole run.

### Run transcripts

`emitTo` appends every `RunEvent` to `runs/<runId>/events.jsonl` (preceded by a `run.request` header
line with the model and form settings), so a run outlives the browser tab and the server process —
the in-memory `Run.events` buffer only serves SSE replay. Each line carries `atMs`, elapsed run time;
`tool.result`'s own `ms` is the call duration, which is why the two keys differ.
`server/scripts/transcript.mjs` folds that file into a readable transcript. Alongside it,
`runs/<runId>/mcp.log` is pixel-mcp's own stderr — the place to look when a tool fails for reasons
the tool result does not explain.

### The benchmark gallery (`server/src/gallery.ts`, `web/src/components/Gallery.tsx`)

Saving an image **copies** it — plus the sprite, the whole `RunRequest`, and the run's tallies — into
`gallery/<entryId>/{image.png, sprite.aseprite, meta.json}`. It has to be a copy: `pruneOldRuns`
deletes the workspace it came from, and the point of a benchmark record is that it outlives the run.
`config.galleryDir` is deliberately a sibling of `runsDir`, not a child.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nbrown725/pixel-art-lab](https://github.com/nbrown725/pixel-art-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
