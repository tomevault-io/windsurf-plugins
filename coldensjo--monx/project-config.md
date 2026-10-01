---
trigger: always_on
description: A monster editor for OpenTibia. **Open workspace → pick a monster → edit → save.**
---

# MONx — Agent Guide

A monster editor for OpenTibia. **Open workspace → pick a monster → edit → save.**

Opens a workspace of up to four folders: the server's `monster/` folder, its `items/` folder, a client folder and optionally `spells/`. Every outfit, corpse and loot item renders as a real sprite because the client assets are loaded alongside the monsters.

**Only the monsters folder is required.** Canary and BlackTek ship no `items.otb` and Nostalrius no client, so a workspace can open with monsters alone — reading, linting and saving all work regardless.

**The client slot takes either kind of client.** A `.spr`/`.dat` folder goes through the inherited SPRx engine; a modern asset bundle (a folder with `catalog-content.json`, as Canary and any 12.x+ client ship) goes through `assets.rs` + `appearances.rs` instead. BlackTek is a third case that needs no third reader: its `assets.dat` is a stock 10.98 `.dat` under another name, and `open_workspace` prefers one found in the items folder over the client folder's.

**The item database has three spellings**: `items.xml`, BlackTek's `items.toml`, and Nostalrius's 7.x `items.srv` — all three read by `items.rs` into one `ItemInfo`, so nothing above it asks which file it came from. `items.otb` is optional; without one the server id *is* the client id, which is how the modern engines, BlackTek and Nostalrius all address things. Ask `ItemIndex::client_id()` for the mapping rather than the OTB, or previews go blank on every engine that has no OTB.

MONx is a fork of **SPRx**, a sprite browser for the same client formats. The sprite/thing engine — `spr.rs`, `dat.rs`, the protocol image server, the virtualized browsers — is inherited whole. What's new is the monster-XML layer on top.

**There is a refactor in progress.** [MODULARITY.md](MODULARITY.md) tracks it: what has been split so far, the two-part method used to prove each split changed no behaviour, and what is left. Read it before splitting a large file or before starting on `Workspace.tsx` or `CreateWizard.tsx`, and delete it when its list is empty.

## Stack

| Layer | Tech |
|-------|------|
| Desktop shell | Tauri 2 |
| Frontend | React 18, TypeScript, Vite |
| Backend | Rust (`src-tauri/`) |
| XML | `quick-xml` |
| Binary formats | `byteorder` (OTB), hand-rolled readers (`spr.rs`, `dat.rs`, `appearances.rs`), `lzma-rs` (modern sprite sheets) |
| Package manager | Bun (`packageManager: bun@1.3.14`) |
| Icons | lucide-react |

No linter config beyond TypeScript strict mode. The frontend has a small `bun test`
suite over the pure modules — see below; the backend is covered by the probes rather
than by tests.

**Verifying changes**

- Frontend: `bun run build` (runs `tsc` then Vite) or `bun run tauri:dev`.
- Frontend strings: `bun run i18n` — fails naming any string `pl.ts` or `pt.ts` is missing. Run it on every change that touches `src/`; see [Text the user reads](#text-the-user-reads).
- Catalogue mirrors: `bun run catalog` — fails when `catalog.rs` and `catalog.ts`, or `engine/` and `engine.ts`, stop agreeing. Run it on every change to any of the four.
- Command table: `bun run commands` — fails when a shell `Command` has no `DEFAULT_BINDINGS` row, a binding names a command that no longer exists, an id is duplicated, or two commands ship on one chord. Run it on every change to `Workspace.tsx`'s command table or `hotkeys.ts`.
- Frontend unit tests: `bun test` — the pure modules only (`lintfix`, `blocks`, `diff`, `lootsim`). No DOM, no test dependencies: they are excluded from `tsconfig.json` and run by Bun's own runner, so `bun run build` still type-checks the app alone. `lintfix` is the one that matters most — it is the only place the frontend rewrites a monster the user did not type, and its tests assert the `--mutate` property: **a fix changes its own path and nothing else.**
- Backend compile: `cargo check` in `src-tauri/`.
- Backend behavior: `probe_monster` is the fastest end-to-end check — it reads and rewrites the whole monster corpus and diffs the bytes, so a round-trip regression fails across every file at once instead of arriving as a bug report. `probe_dat` does the same for sprite composition.

`probe_monster` takes flags for each gate, and exits non-zero if any of them fails. With no
path it runs against the committed fixtures, so a fresh clone can check itself before
`assets/` is populated:

```sh
cargo run --release --example probe_monster                                              # fixtures/engines/ironcore/monsters
cargo run --release --example probe_monster -- fixtures/engines/tvp/monster --engine tvp --mutate
```

Those are a smoke test, not coverage — see `src-tauri/fixtures/README.md`. The real gates
point at a server's own tree:

```sh
cargo run --release --example probe_monster -- ../assets/Ironcore/monsters              # round-trip
cargo run --release --example probe_monster -- ../assets/Ironcore/monsters --canonical --mutate
cargo run --release --example probe_monster -- ../assets/Ironcore/monsters --lint
cargo run --release --example probe_monster -- ../assets/Ironcore/monsters --crud <scratch-dir>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Coldensjo/MONx](https://github.com/Coldensjo/MONx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
