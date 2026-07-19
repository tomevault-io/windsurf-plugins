---
trigger: always_on
description: Flow Desktop (`io.github.aedev.flow.desktop`) is a privacy-first YouTube and YouTube Music client built with **Tauri 2 (Rust backend) + React 19/TypeScript (frontend)**. It is the desktop companion to [Flow for Android](FlowApp_mobile/AGENTS.md). It plays YouTube content via a native, hand-rolled Innertube client (SABR/DASH/HLS, no `protoc`), supports downloads with a native muxer, SponsorBlock/DeArrow/Return YouTube Dislike integrations, P2P sync with the Android app, and two on-device recommen
---

# Working with Flow Desktop as an AI agent

Flow Desktop (`io.github.aedev.flow.desktop`) is a privacy-first YouTube and YouTube Music client built with **Tauri 2 (Rust backend) + React 19/TypeScript (frontend)**. It is the desktop companion to [Flow for Android](FlowApp_mobile/AGENTS.md). It plays YouTube content via a native, hand-rolled Innertube client (SABR/DASH/HLS, no `protoc`), supports downloads with a native muxer, SponsorBlock/DeArrow/Return YouTube Dislike integrations, P2P sync with the Android app, and two on-device recommendation engines: **FlowNeuro** (video) and **MusicBrain** (music). No account, no analytics SDK, no telemetry.

Frontend lives at `src/`, backend at `src-tauri/src/`. Run all `cargo` commands from inside `src-tauri/` — it needs `.cargo/config.toml`'s `reqwest_unstable` rustflag. Use `pnpm` (not `npm` or `yarn`) for JS.

## Design system — Design.md is the source of truth

**[Design.md](Design.md)** is the absolute source of truth for all UI/UX design and React architecture in this repository. You must read and adhere to it before writing or editing any React/Tailwind code. It defines the anti-slop rules, the color/depth system, layout patterns (Bento grid vs. content-feed grids), typography scale, component blueprints (radii, buttons, icons), interaction/animation rules, the container/presentational architecture (`components/ui/` dumb primitives, `components/[domain]/` smart containers, `pages/` route composition, `lib/` hooks-only backend access, `store/` Zustand), and the performance rules (optimistic UI, debounced search, graceful FOSS-API degradation).

Quick-reference summary (Design.md governs on any conflict or ambiguity):
- ❌ No gradients, no glassmorphism/backdrop-blur, no drop shadows/glows, no colored card borders, no arbitrary hex codes — everything maps to the CSS variables in `src/App.css`.
- Depth comes from contrast + 1px neutral borders (`border-neutral-800`), not shadows.
- Radii: pills/chips `rounded-full`, cards/dialogs `rounded-2xl`, thumbnails `rounded-xl`, inputs/menus `rounded-lg`/`rounded-md`.
- Never call `invoke(...)` or `fetch(...)` directly inside a component's `useEffect` — wrap backend calls in a custom hook under `src/lib/`.
- Never hardcode UI strings — use `react-i18next` and `src/locales/`.

If your proposed code contains `shadow-`, `bg-gradient-`, business logic inside `components/ui/`, or an arbitrary hex color instead of a `src/App.css` variable, rewrite it before presenting it to the user — see Design.md's Agent Execution Directive.

## graphify

This project has a knowledge graph at `graphify-out/` with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when `graphify-out/graph.json` exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts.
- Dirty `graphify-out/` files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If `graphify-out/wiki/index.md` exists, use it for broad navigation instead of raw source browsing.
- Read `graphify-out/GRAPH_REPORT.md` only for broad architecture review or when query/path/explain don't surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Backend architecture (src-tauri/src/)

- `api/innertube/` — native Innertube client (no `protoc`), plus `dearrow.rs`, `extractor.rs`, `http.rs` (shared `reqwest` client — never build a new client per request).
- `streaming/sabr/` — hand-rolled SABR/DASH stack; `streaming/proxy.rs` for range-request proxying.
- `flow_neuro/` — video recommendation engine (ranker, scoring, signals, tokenizer, resident brain store).
- `music_brain/` — entity-based music recommendation engine (ACT-R heavy rotation, separate from FlowNeuro). Video and music recommendation state must never leak into each other — `watch_history` is shared between both surfaces (`is_music` flag), so any video-feed recall/seed path must filter out music rows.
- `sync/` — CRDT-based P2P LAN sync with the Android app (codec, merge, ledger, transport, QR pairing). This is the most fragile subsystem — see `src-tauri/tests/sync_*.rs` (golden, fuzz, scale tests) before changing anything here.
- `security/` — input validation (`validation.rs`).
- `commands/` — Tauri `#[tauri::command]` handlers, one file per domain (downloads, music, sync, notifications, recommendation, shorts, youtube).
- `services/` — business-logic layer between commands and api/db.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlowNeuro/Flow-Desktop](https://github.com/FlowNeuro/Flow-Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
