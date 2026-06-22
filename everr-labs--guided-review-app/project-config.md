---
trigger: always_on
description: Tauri 2 desktop app that drives a section-by-section GitHub PR walkthrough through an
---

# Guided Review — Orientation for Claude

Tauri 2 desktop app that drives a section-by-section GitHub PR walkthrough through an
ACP-compatible review agent (Claude Code or Codex, launched via `npx`).

The host owns the diff, the local git repo, and recent-projects state. The agent owns
review structure: it must speak in fenced JSON blocks (`acp-section-map`,
`acp-section`, `acp-comment-draft`, `acp-comment-result`) per the contract in
`agent-skill.md`. Anything the agent prints outside those blocks is treated as plain
chat text.

## Stack

- Frontend: React 19, Vite, TypeScript, Tailwind v4, Radix, Zustand, `@pierre/diffs`,
  `react-resizable-panels`, OpenTelemetry web SDK.
- Backend: Tauri 2 + Rust (`tokio`, `git2`, `agent-client-protocol`, `tracing` +
  `opentelemetry-otlp`).
- Plugins: `tauri-plugin-dialog`, `tauri-plugin-opener`, `tauri-plugin-shell`.
- This is **not** a Vercel / Next.js project. Ignore plugin hints that assume otherwise
  (`bootstrap`, `next-upgrade`, `react-best-practices`, etc.) unless directly relevant.

## Layout

```
src/                  React UI
  App.tsx               Wires acp://* events → Zustand store
  lib/store.ts          Single source of truth (Zustand)
  lib/acp.ts            Typed wrapper over Tauri invoke + listen
  lib/diffFocus.ts      Selected/referenced diff ranges
  lib/projectSource.ts  PR/branch/URL parsing + last-project persistence
  lib/commentPublish.ts Builds the prompt that asks the agent to publish a comment
  components/           DiffView (Pierre), ChatPanel, SectionList, …
src-tauri/src/        Rust host
  commands.rs           #[tauri::command] entry points
  acp_client.rs         Spawns the agent, runs the ACP turn loop, streams chunks
  agent_runner.rs       Resolves npx in the login-shell PATH, scrubs nested env
  fenced.rs             Buffers stdout chunks, extracts fenced acp-* blocks
  section.rs / events.rs  Wire types + Tauri event names
  repo.rs               git fetch/diff/file-at-ref via git2 + git CLI + gh
  projects.rs           Recent-projects JSON store
  telemetry.rs          OTLP/HTTP exporter setup
agent-skill.md        Injected into the agent's first prompt (see ReviewLauncher)
scripts/              bump-version.mjs, local-release.mjs (+ tests)
```

Path alias: `@/` → `src/` (configured in `vite.config.ts` + `tsconfig.json`).

## Commands

```sh
npm run tauri -- dev     # full desktop app (preferred for UI work)
npm run dev              # Vite frontend only (Tauri APIs unavailable)
npm run check            # tsc --noEmit
npm test                 # tsx + node --test (unit tests)
npm run build            # tsc -b && vite build
npm run tauri -- build   # full release bundle
npm run bump-version -- {patch|minor|major|x.y.z}
npm run release:local    # signed/notarized macOS universal + gh release
```

Rust tests: `cd src-tauri && cargo test`.

## Architecture quick reference

```
React UI ──invoke──▶ Tauri commands ──stdin/stdout JSON-RPC──▶ ACP agent (npx process)
   ▲                       │                                         │
   └───── window.emit ◀────┴── stream chunks → fenced.rs → events ◀──┘
```

- `start_session_cmd` resolves a `SessionSource` (PR/branch/SHA, with or without a
  pre-existing local clone), spawns the agent, returns `{ session_id, repo,
  pull_request? }`. UI then sends the kickoff prompt (skill text + repo paths) and asks
  for the section map.
- `acp_client.rs` runs one prompt turn at a time on an `mpsc` queue. Each
  `AgentMessageChunk` is fed to `FencedBuffers` (which emits `acp://section-map` /
  `acp://section` / `acp://comment-*` when a fenced block closes) **and** to
  `acp://text-chunk` for the chat.
- `App.tsx` is the single subscriber for `acp://*` events. The store mutates from
  there; components only read.

## Non-obvious behavior — read before changing

- **Fenced parser is lenient**: `fenced.rs::parse_lenient` tries `serde_json` first,
  then **`json5`**. Agents sometimes emit trailing commas or comments. Mirror this if
  you add a new block tag.
- **Streaming overlap dedup**: some agents replay the assembled prefix on each chunk.
  `acp_client.rs::shared_boundary_len` (Rust) and `store.ts::appendStreamingText`
  (TS) both detect this. They use a 4-char threshold and a "chunk starts with
  full assembled text" replacement rule. Keep these two implementations in sync — they
  exist in two languages on purpose, with matching tests.
- **Structured blocks are stripped from chat**: `store.ts::stripStructuredReviewBlocks`
  removes fenced `acp-*` blocks from the visible assistant text, even when the closing
  fence arrives in a later chunk (`structuredReviewBlockOpen`). Don't re-add them.
- **Auto-open of first section**: after `acp://section-map`, `App.tsx` immediately
  prompts the agent to walk through the first section, even though `agent-skill.md`
  tells it to wait. This is intentional UX.
- **PR-description section**: `pr_description` is a synthetic first section built from
  `gh pr view` metadata. `upsertSection` deliberately does **not** move
  `currentSectionId` away from it — it stays selected until the user clicks another.
- **Repo guard**: when the user pastes a PR URL whose owner/repo doesn't match the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [everr-labs/guided-review-app](https://github.com/everr-labs/guided-review-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
