---
trigger: always_on
description: **Generated:** 2026-09-07T19:52:14+03:00
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-09-07T19:52:14+03:00
**Commit:** 90d4531
**Branch:** main

## OVERVIEW
Tidemark tracks AI-provider quota windows and pace. Six Rust crates separate daemon state, provider I/O, shared vocabulary, the generated D-Bus proxy, GTK presentation, and `tidemarkctl`; Rust edition 2024, MSRV 1.92, GTK 4.22, libadwaita 1.9.

## STRUCTURE
```text
tidemark/
|-- crates/
|   |-- tidemark/        # GTK GUI; consumes IPC, never core
|   |-- tidemark-cli/    # tidemarkctl; D-Bus client, no core/display/Tokio
|   |-- tidemarkd/       # Polling, history, secrets and IPC ownership
|   |-- tidemark-core/   # External I/O and provider implementations
|   |-- tidemark-ipc/    # Single generated D-Bus client proxy
|   `-- tidemark-types/  # Shared domain and wire vocabulary
|-- data/               # Desktop assets, user service, packaging payloads
|-- scripts/            # Layering, integration, packaging and release checks
|-- nix/                # Package and NixOS module
|-- docs/adr/           # Ownership and integration decisions
|-- CONTEXT.md          # Architecture
`-- PLAN.md             # Implementation log
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Add a provider | `crates/tidemark-core/src/providers/`, `crates/tidemarkd/src/registry.rs` | Core implementation plus daemon registration; simple catalog and hand-written descriptors coexist |
| Change IPC vocabulary | `crates/tidemark-types/src/wire.rs` | Keep daemon and GUI compatible |
| Change the D-Bus proxy | `crates/tidemark-ipc/src/lib.rs` | Keep the one generated client contract synchronized with the daemon interface |
| Change the CLI | `crates/tidemark-cli/` | Preserve JSON/Waybar shapes, exit codes, secret input and layering |
| Polling or mutation ordering | `crates/tidemarkd/src/engine.rs`, `service.rs` | Owned state and published mirror |
| GUI or daemon reconnect | `crates/tidemark/src/window.rs`, `bus.rs` | Presentation and IPC client |
| Credentials or browser sources | `crates/tidemark-core/src/oauth_file.rs`, `secrets.rs`, `browser/` | Ownership and explicit source selection matter |
| Charts or notice identity | `crates/tidemark-core/src/storage/` | Segmentation, retention and migrations |
| Installation | `README.md`, `PKGBUILD`, `data/packaging/`, `nix/` | deb/rpm metadata also lives in GUI Cargo manifest |
| Ownership rationale | `CONTEXT.md`, `docs/adr/` | Normative design and binding decisions; dated superpowers plans/specs are historical, not current-code proof |
| Release and CI | `scripts/release.sh`, `.github/workflows/` | Release helper commits, tags and pushes |

## CODE MAP
Declarations/import shapes are reported by subtree analysis; exact numeric reference counts are unmeasured.

| Symbol | Type | Location | Refs | Role |
|--------|------|----------|------|------|
| `Engine` / `Command` / `Publication` | Runtime types | `crates/tidemarkd/src/engine.rs` | Unmeasured | State, serialized mutations, publication |
| `Daemon` / `Published` | Service types | `crates/tidemarkd/src/service.rs` | Unmeasured | D-Bus interface and shared mirror |
| `catalog` / `account` | Functions | `crates/tidemarkd/src/registry.rs` | Unmeasured | Definitions and configured clients |
| `Provider` / `ProviderError` | Trait / error | `crates/tidemark-core/src/providers/mod.rs` | Unmeasured | Provider contract |
| `Spec` / `HandSpec` / `CATALOG` | Descriptors / catalog | `crates/tidemark-core/src/providers/keyed/mod.rs` | Unmeasured | Simple and custom provider definitions |
| `History` | Storage type | `crates/tidemark-core/src/storage/mod.rs` | Unmeasured | Persisted usage history |
| `Config` / `CredentialFile` | Configuration / credential types | `crates/tidemark-core/src/{config,oauth_file}.rs` | Unmeasured | Preferences and vendor credential updates |
| `ProviderStatus` / `Preferences` | Wire types | `crates/tidemark-types/src/wire.rs` | Unmeasured | Cross-process dictionary contract |
| `ids` | Module | `crates/tidemark-types/src/lib.rs` | Unmeasured | Installed identity and schema constants |
| `MainWindow` / `CardGrid` | GUI types | `crates/tidemark/src/{window,grid}.rs` | Unmeasured | Coordination and reorderable layout |
| `DaemonProxy` / `Update` | Client / update types | `crates/tidemark/src/bus.rs` | Unmeasured | IPC client and reconnect stream |

**Poll to pixel:** `tidemarkd::main::run` loads `Config`, `History`, and `Keyring`; `registry::accounts` supplies accounts. `Engine::poll_due` lazily constructs providers and fetches concurrently; `Engine::apply` calls `History::ingest(&Snapshot)` and `ProviderStatus::set_reading`. The publisher updates the mirror with `Published::upsert`, then emits `Daemon::provider_changed`.
On the GUI side, `bus::watch` drives `DaemonProxy` on `glib::spawn_future_local`; signals become `Update::Changed`. `MainWindow::handle` calls `show_all`/`show_one`; `Card::apply` uses `ProviderStatus::to_snapshot`; `QuotaBar::set` draws value and pace on a Cairo-backed `gtk::DrawingArea`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zbndev/tidemark](https://github.com/zbndev/tidemark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
