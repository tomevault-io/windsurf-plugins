---
trigger: always_on
description: You are working **on** the demo engine, not just using it. The
---

# Working on aidemo — guide for coding agents

You are working **on** the demo engine, not just using it. The
[README](README.md) explains what aidemo is and why it's built this way;
[CONTRIBUTING.md](CONTRIBUTING.md) covers process (DCO sign-off, PR
expectations, CI policy). If this file contradicts them, they win.

**Mental model:** `storyboard.json` (script + per-scene voice/music plan +
browser action-spec) → `voice` (OpenAI TTS) → `record` (deterministic replay in
real Chrome, injected cursor, timeline) → `captions` (Whisper word timing) →
`compose` (ffmpeg: trim idle, sync to narration, zoom, cards, mux) →
`output/final-demo.mp4`.

## Commands

| Task | Command |
|---|---|
| Install | `npm install` — Node 20+, system Chrome, ffmpeg+ffprobe on PATH (no Playwright browser download) |
| Type check | `npm run typecheck` |
| Fixture server | `node examples/local-demo/serve.mjs` (port 8787) |
| E2E smoke test | `node bin/aidemo.mjs render examples/local-demo --headless` |
| Dry-run actions only | `node bin/aidemo.mjs probe examples/local-demo --headless` |
| Resume a take | `node bin/aidemo.mjs record <dir> --from-scene <id>` (also `render`, MCP `fromScene`) — reuses the previous take's earlier scenes (hash-guarded), replays their actions fast, records from `<id>` |
| Draft from a Playwright trace/test | `node bin/aidemo.mjs import-trace <trace.zip\|spec.ts> --name <demo>` — actions + selectors → scenes, no LLM (MCP `import_trace`) |
| Draft from a URL | `node bin/aidemo.mjs init <name> --from-url <url>` — inspect first, headings → scenes, unique selectors → beats (MCP `init_demo {fromUrl}`) |
| Selector discovery | `node bin/aidemo.mjs inspect <url> --dir <demo>` — unique selectors per visible element (MCP `inspect` job); the same scan writes `logs/drift-*.json` suggestions when a take's selector matches nothing |
| Validate a storyboard (no browser) | `node bin/aidemo.mjs validate <dir>` (`--file <path>`, `--json`; non-zero exit on issues) |
| Lint / pacing forecast (no browser) | `node bin/aidemo.mjs lint <dir>` (`--lang`, `--json`, `--strict`) — also auto-runs in probe/record/render; measured counterpart is `output/report.json` from compose |
| Frames for review | `node bin/aidemo.mjs frames <dir> --every 3` (`--source raw` for the take) |
| Walkthrough bundle | `node bin/aidemo.mjs walkthrough <dir>` — output/walkthrough/ (index.html, guide.md, frames, captions) from the final video (also auto in `render` with `output.walkthrough`) |
| One pipeline stage | `node bin/aidemo.mjs voice\|record\|captions\|compose <dir>` |
| Screenshot stills | `node bin/aidemo.mjs stills <dir>` — extract named PNGs from an existing take (also auto-runs in `render` when the storyboard has `still` markers) |
| Golden regression check | `node bin/aidemo.mjs probe <dir> --update-golden` (write baseline) / `--golden` (CI guard, non-zero exit on drift) |
| Multi-language matrix | `node bin/aidemo.mjs render <dir> --langs de,fr` — one take → N locales (also `--lang` on voice/captions/compose) |
| Personalized variants | `node bin/aidemo.mjs render <dir> --variants variants.json` (or `--param k=v`) |
| Always-fresh embed snippets | `node bin/aidemo.mjs embed <dir>` — stable raw-GitHub URLs for READMEs/PRs |
| CI render (consumers) | `uses: tandryukha/aidemo@stable` (composite action, `action.yml`) — see `docs/CI.md` |
| MCP server (agent interface) | `node bin/aidemo.mjs mcp` — stdio; smoke test: `npm run mcp-smoke` (needs Chrome) |
| Print authoring guide | `node bin/aidemo.mjs guide` (`--topic core\|schema\|attention\|polish\|…`, `--list`) |
| Environment check | `node bin/aidemo.mjs doctor` |

`render`, `voice`, and `captions` need `OPENAI_API_KEY` in `.env` (or
`OPENAI_BASE_URL` pointing at a local OpenAI-compatible server, or
`AIDEMO_TTS_PROVIDER=local` + `npm install kokoro-js` for in-process TTS with
offline captions — no key at all); `probe`, `record`, and `compose` re-runs
don't. Every engine change should keep the fixture rendering end-to-end
(that's the smoke test CI can't run for you — it needs Chrome, ffmpeg, and a
TTS/STT endpoint or the local provider).

## Layout

```
bin/aidemo.mjs        CLI entry (launches tsx → src/cli.ts)
src/types.ts          storyboard schema (zod) — the contract everything shares
src/                  pipeline stages: voice, recorder/player/cursor (record),
                      captions/caption-render, compose/zoom/cards/music/ffmpeg
                      (compose writes output/report.json), attention (highlight/
                      spotlight/callout/keystroke/click-ring PNGs + redact blur
                      filter), inspect (page scan → unique selectors; drift
                      ranking for failed selectors), anchors ({{@word}} markers → piecewise
                      retime), frame (produced-look canvas PNG), guide
                      (topic slices of AUTHORING.md), import-trace (Playwright
                      trace.zip / spec → draft storyboard), lint (browser-free
                      pacing forecast + pitfalls), recorder (take lifecycle,
                      resume via per-scene hashes + raw.keep-* footage),
                      stills (screenshot mode), frames (review PNGs),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tandryukha/aidemo](https://github.com/tandryukha/aidemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
