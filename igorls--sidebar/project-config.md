---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Sidebar** — an ambient panel of AI agents for a live meeting: one keeps a rolling
summary, one turns spoken ideas into running HTML prototypes in ~2s, one fact-checks
claims. Built on **Cerebras + Gemma 4** via [`universal-llm-client`](https://github.com/igorls/universal-llm-client), on **Bun**. The hero is the
real-time prototype agent and its fan-out → pick → learn loop.

`sidebar-build-spec.md` is the design source of truth — agent prompts (`prompts.ts`)
and schemas (`schemas.ts`) are copied verbatim from its section 4, and the WS protocol
from section 7. Reconcile changes against it.

## Commands

**Running the app is Docker by default** — `docker-compose.yml` brings up the Bun server
(web + `/ws` + API on one port `:3001`) behind a **Tailscale sidecar** that serves it as an
isolated tailnet node (`https://$TS_HOSTNAME.<tailnet>.ts.net`, tailnet-only `serve`, not
funnel). Needs `TS_AUTHKEY` in `.env`. The working tree is bind-mounted and the server runs
under `bun --watch`, so server/orchestrator edits hot-reload; web edits need a rebuild.

```bash
docker compose up                       # build + run (logs in foreground); the default run path
docker compose up -d                     # detached
docker compose logs -f app               # follow the server log (live agent calls)
docker compose down                      # stop; tailnet node goes offline
docker compose exec app bun run build    # rebuild web bundle after apps/web edits
# After a package.json/lockfile change: docker compose build && docker compose down -v
#   (the anonymous node_modules volumes re-seed from the image)
```

The rest below run **directly on the host** (no Docker) — needs Bun ≥ 1.1, from repo root.

```bash
bun install
cp .env.example .env     # works as-is in mock mode (no API key needed)
bun run dev              # server :3001 + web :5173 together (concurrently)
bun run dev:server       # just the Bun WebSocket server (bun --watch)
bun run dev:web          # just the Vite app
bun run typecheck        # tsc --noEmit across shared + server + web — THE check gate
bun run build            # production web build
bun run asr:gen          # (re)generate fixtures/audio/* from transcripts via ElevenLabs TTS
bun run asr:bench        # score local Gemma ASR against those fixtures (WER) — needs Ollama
bun run meetings:gen     # (re)generate fixtures/meetings/* — naturalistic v3 meetings
bun run meetings:bench   # WER on the realism-set clips (asr-bench --set meetings)
                         #   --backend both -> Gemma local vs ElevenLabs Scribe head-to-head
bun run asr:livesim      # simulate the live on-device path: app's energy-VAD over meeting.wav -> Gemma
```

There is **no test runner and no linter**. `bun run typecheck` is the only automated
check — run it before claiming a change is done. `test-transcripts.json` is *not* a
test suite; it is the fixture stream (see below). To check a single scenario, set
`FIXTURE_SCENARIO` and run the app.

Two committed audio-fixture sets feed ASR work (16 kHz mono WAV, generated once and
committed — the bench never calls the network; needs an ElevenLabs key with
`text_to_speech` to *re*generate):
- **`fixtures/audio/`** — clean TTS of `test-transcripts.json` (canonical), `eleven_multilingual_v2`.
- **`fixtures/meetings/`** — naturalistic, multilingual (EN/PT-BR), overlapping meetings
  authored in `fixtures/meetings/scripts.json` and rendered with **`eleven_v3`** (audio
  tags + emotion). Separate from the agent gold labels on purpose. Per scenario: clean
  per-utterance clips (WER) + a `meeting.wav` mix with true cross-talk (not WER-scored).
  See `fixtures/meetings/README.md`.

## Architecture

Bun-workspace monorepo, three packages. The shared package is the contract both other
packages compile against.

- **`packages/shared`** (`@sidebar/shared`) — the single source of cross-cutting truth:
  the WebSocket event protocol (`events.ts`), Zod schemas + inferred types
  (`schemas.ts`), agent system prompts (`prompts.ts`), and design-language themes +
  the mock prototype HTML builders (`themes.ts`). Imported via the `@sidebar/shared`
  path alias (`tsconfig.base.json`) and as a `workspace:*` dependency. Change a type
  here and both server and web see it.
- **`apps/server`** — `Bun.serve` WebSocket at `/ws` (+ `/health`). One `Session`
  per connection (`session.ts`), each owning one `Orchestrator` (`orchestrator.ts`).
  No DB; all state is in-memory per session.
- **`apps/web`** — Vite + React. A single `useReducer` in `ws.ts` holds the entire
  client state, driven purely by the inbound WS event stream; components just render
  it. Auto-reconnects every 1s.

### The mock/live duality (central pattern)

Every agent has two code paths selected by `config.agents`:
- **`mock`** (default, no API key): the orchestrator replays the gold-label `expect`
  blocks baked into `test-transcripts.json`. The prototype agent streams a pre-built
  themed HTML doc (`buildPrototype` in `themes.ts`) chunked over time to fake token
  streaming.
- **`live`**: real Cerebras inference. Router/summarizer/factcheck use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igorls/sidebar](https://github.com/igorls/sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
