---
trigger: always_on
description: Google-Maps-style procedural fantasy world for tabletop games: continent → battlemap zoom in the browser.
---

# Worldspring

Google-Maps-style procedural fantasy world for tabletop games: continent → battlemap zoom in the browser.
Public at github.com/Dun-John/worldspring (source, all rights reserved) and https://dun-john.github.io/worldspring/.
Local notes, not in the repo: `docs/authoring-handoff.md` (git-ignored: current state, backlog, the user's decisions;
read it at the start of a session) and the original plan (milestones M0–M9, all delivered) in
`~/.claude/plans/pasted-content-id-4ec8-help-me-lexical-raccoon.md`.

## Layout
- `crates/worldgen` — pure deterministic generator (Rust). No I/O, no threads inside generators,
  transcendentals only via `libm`, randomness only via `core::rng` hashing keyed by seed + coords.
- `crates/worldgen-wasm` — thin wasm-bindgen wrapper used by browser workers.
- `crates/mapd` — local agent server (127.0.0.1 only): MCP tools at `/mcp`, live sync with the app at `/ws`,
  worlds on disk in `worlds/`. See `docs/AGENT.md`.
- `app/src/ui/shell` — the app's frame: `layout.svelte.ts` (what is open: section World/Edit/Notes/Play and its tab,
  phone/short layouts as `data-layout` on `<html>`), `Dock` (right panel; on phones a bottom sheet over the tab bar),
  `TopBar` (☰ menu, search, breadcrumbs), `SectionBar`, `MapControls` (zoom, Layers, floor picker), `keymap.ts` (every
  keyboard shortcut, one listener; `?` shows them), `history.svelte.ts` (undo). In App, `go(section, tab)` opens a panel
  (asking before unsaved sketch strokes or a site design would be dropped) and `syncTool()` picks the map's pointer
  tool. Panel bodies take `peek` (only their tool strip). Look: tokens and `ws-` classes in `app.css`, icons in
  `ui/icons.ts`; no per-component breakpoints. Performance stats stay hidden unless asked for (` or the ☰ menu; any
  `?bench` run shows them).
- `app/` — Vite + TS + Svelte 5 + PixiJS v8 (WebGL2). `src/gen` workers, `src/render` map, `src/sync` mapd link,
  `src/editor` sketch mode (strokes in `WorldFile.sketch` steer T0: `crates/worldgen/src/t0/sketch.rs`) and scatter
  (battlemap objects put down or cleared by hand, uploaded sprites: `Edits.objects`/`cleared`/`sprites`, applied in
  `battlemap::apply_edits`; MCP in `crates/mapd/src/scatter.rs`),
  `src/play` play mode: the DM's tools plus a players' window (`app/player.html`, its own map and workers) kept in
  step over a BroadcastChannel by ops (`play/state.ts`); sessions per world hash in IndexedDB. DM-only: secret
  doors not yet found, hazards whose rules start "hidden" (traps, sinkholes), NPC markers, notes and plots.
  `src/editor/build.ts` + `ui/BuildPanel.svelte` draw buildings (`Created` kind `building`: one layout each,
  `town/sites.rs` `drawn`; placement check `agent::building_spot`; MCP in `crates/mapd/src/build.rs`).
  `src/editor/site/` + `ui/DesignPanel.svelte` is the dungeon designer: an underground site (`u:`) copied into
  `Edits.designs` and built from there (`crates/worldgen/src/under/design.rs`: build, `check` with the vital
  rules, auto doors, text plans; MCP in `crates/mapd/src/design.rs`).
  `src/ui/Notebook.svelte` is the DM's notebook: NPCs and plot points (`Edits.npcs`/`plots`, MCP in
  `crates/mapd/src/notebook.rs`), authored, never generated.
  A city's sewers (generated per 480-ft section `w:<layout>:<sx>:<sy>`) are shown as one network
  (`render/Sewers.ts`: sections stream round the view, textures prepared in `underField.worker.ts`);
  a section's undercroft opens on its own. In play mode the sewer level is one location `w:<layout>`.
- `scripts/` — `build-wasm.mjs`, `det-wasm.mjs`, `bench.mjs`, `shot.mjs`, `publish.mjs`, `readme-media.mjs`.
- `README.md` — the repo's front page (using the site, running it, license), pictures in `docs/media/`;
  `THIRD_PARTY_NOTICES.md` — notices for code that follows others' work (Lucide icons).

## Commands
- First time: `npm --prefix app install` (also brings `wasm-opt`); Rust with the `wasm32-unknown-unknown` target and
  `cargo install wasm-bindgen-cli --version 0.2.129` (must match `Cargo.lock`).
- `npm run wasm` — build WASM into `app/src/gen/pkg` (required before `npm run dev`).
- `npm run dev` — dev server on :5173. `?seed=N` picks a world, `?bench=1` runs the perf fly-through
  (mountain, region, continent, then down into the biggest city), `?bench=play` the play-mode one (30 tokens,
  fog and line of sight in the biggest city), `?bench=sewer` panning through its sewers, `?bench=dungeon` the
  biggest ruin's dungeon (all its levels), `?bench=edit` scattering and clearing objects while panning a battlemap,
  `?bench=heavy` the stress bench (a world as generated, then loaded with ~6 MB of edits: 50 designed sites, 20k
  objects, 40 sprites, 2k NPCs…; edit latency and a battlemap pan, before and after; it refuses a world that has
  edits), `?gpu=1` adds GPU timer queries to any of them.
- `npm run dev -- -- --host` — the same, open to the LAN (Vite prints the Network URL; allow Node through the
  firewall once). A reverse proxy needs its hostname in `server.allowedHosts` (`app/vite.config.ts`); keep that
  line uncommitted (adding `host: true` there makes `--host` the default).
- `npm run mapd` — the agent server on 127.0.0.1:7777 (`-- --port N --dir worlds --app app/dist`); the app on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dun-John/worldspring](https://github.com/Dun-John/worldspring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
