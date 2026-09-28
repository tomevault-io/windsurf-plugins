---
trigger: always_on
description: VS Code extension for HOI4 modding. Two-part architecture: `client/` (TypeScript VS Code extension) + `server/` (Rust LSP server, `tower-lsp` + `tokio` + `tikv-jemallocator`, Rust 2024 edition).
---

# Hearts of Modding

VS Code extension for HOI4 modding. Two-part architecture: `client/` (TypeScript VS Code extension) + `server/` (Rust LSP server, `tower-lsp` + `tokio` + `tikv-jemallocator`, Rust 2024 edition).

## Reference Docs (`hoi4-wiki/`)

When editing the extension's code (parser, scopes, triggers, effects, semantic tokens, validation, etc.), consult the `hoi4-wiki/` directory first. It contains **Paradox Wiki-format** HOI4 modding reference pages scraped from the official wiki, organized by category:

| Category | Contents |
|----------|----------|
| `scripting/` | Event modding, national focus modding, decision modding, idea/ideology, unit, equipment, technology, doctrine, division, character, building, MIO, country creation, cosmetic tags, balance of power, autonomy/state, achievements, AI modding, AI focuses, faction, bookmark, resources, scripted GUI |
| `documentation/` | Reference: triggers, effects, scopes, modifiers, defines, localisation, on-actions, data structures, ideology modding |
| `graphical/` | Interface modding, graphical assets, entity modding, particle/posteffect/font modding |
| `cosmetic/` | Portrait modding, namelist modding, music/sound modding |
| `map/` | Map modding, strategic region modding, state modding |
| `other/` | Mod structure, mods, nudger, troubleshooting, console commands |

These pages use Paradox Wiki markup (`{{version|1.12}}`, `{{path|events/}}`, `{{Main|Scopes}}`, `<pre>` blocks, `{|` wiki tables) but are otherwise plain markdown. They are the canonical reference for how HOI4 mod files are structured — the parser, scope inference, trigger/effect databases, and validator logic all relate directly to what's documented here. Read these files whenever you need to understand the underlying game mechanics that the extension operates on.

Categories roughly map to extension concerns: `documentation/triggers.md`, `documentation/effects.md`, `documentation/scopes.md` wire directly to `data/hoi4_data.rs` and `scope/scope.rs`; `documentation/localisation.md` to `parser/loc_parser.rs`; `scripting/event-modding.md`, `decision-modding.md`, `national-focus-modding.md` etc. inform scanner logic and semantic token behaviour.

## Build & Dev

| Scope | Commands |
|-------|----------|
| Client | `cd client && npm install && npm run compile` |
| Server | `cd server && cargo build --release` |
| Both + VSIX | `cd client && npm run package` |
| Rust tests | `cd server && cargo test` |
| Rust lint | `cd server && cargo clippy --all-targets -- -D warnings` |
| Rust check | `cd server && cargo check --all-targets` |
| Rust format | `cd server && cargo fmt` |

Client helpers in `package.json`: `npm run cargo:test`, `cargo:check`, `cargo:fmt`, `cargo:clippy` (run from `client/`; `cargo:clippy` is `--all-targets -- -D warnings` — bare `cargo clippy` passes code that CI rejects).

**VS Code debugging:** Use "Launch Extension" config (`.vscode/launch.json`). Falls back to `../server/target/release/server` if `client/server-bin/` not found.

**Validation-rule test convention:** never hand-build a `ValidationContext` literal — the struct has ~35 fields and every new field breaks each literal. Use the shared builder instead: tests in `src/tests/` use `crate::test_support::TestCtx` (e.g. `TestCtx::new().with_file(path, content).walk(input, uri, scope, rules, visitors)`); rule modules' own `#[cfg(test)]` blocks use the same builder via `crate::test_support` (`TestCtx`, or `TestCtxRef` via `wrap_ref` when seeding an external `&ScannerData`). Prefer `.with_file(...)` (runs the real incremental scanner) over hand-stuffing DashMaps; promote a repeated raw seed to a named `with_*` method (e.g. `with_unit_types`, `with_ideas`, `with_event_namespaces`).

## Architecture

**Server module layout** (`server/src/`):

```
server/src/
├── main.rs               # LSP entrypoint, module decls, jemalloc, UTF-16 utils, CancellationToken
├── lib.rs                # (empty — binary crate; tests live in --bin hom-lsp)
├── backend.rs            # Backend struct + ValidationCtx + AST cache + compute_pool (Rayon) + FxHashMap
├── config.rs             # Config struct (ArcSwap + AtomicBool + AtomicU8 log_level + Regex fields)
├── log_level.rs          # LogLevel enum (error/warn/info/debug/trace, AtomicU8)
├── test_support.rs       # TestCtx / TestCtxRef builders for ValidationContext
├── data/                 # Static databases & shared data
│   ├── mod.rs
│   ├── hoi4_data.rs      # Static DB — triggers/effects/scopes/modifiers/loc_commands (hoi4_data.json, minified at build.rs)
│   ├── focus_filters.rs  # Focus search-filter definitions
│   ├── scanner_data.rs   # ScannerData struct (40+ DashMap/DashSet/ArcSwap fields + event/tech dep graphs)
│   ├── entity_lookup.rs  # Adapter over &ScannerData — find_definition, entity_at, etc.
│   ├── interner.rs       # String interning (InternedStr = Arc<str>) for DashMap keys
│   └── layered_value.rs  # VFS layering: LayeredValue<T> preserves vanilla→mod→submod layers
├── lsp/                  # LSP protocol handlers
│   ├── mod.rs
│   ├── handler.rs        # impl LanguageServer for Backend — all LSP protocol handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emberglazee/hearts-of-modding](https://github.com/emberglazee/hearts-of-modding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
