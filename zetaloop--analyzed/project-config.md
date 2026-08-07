---
trigger: always_on
description: `analyzed` is a drop-in replacement for rust-analyzer that runs the analysis in a shared daemon. The daemon runs the upstream rust-analyzer main loop for every connected session; sessions with a compatible toolchain and Cargo configuration attach to the same analysis backend, sharing work instead of duplicating it.
---

# analyzed

`analyzed` is a drop-in replacement for rust-analyzer that runs the analysis in a shared daemon. The daemon runs the upstream rust-analyzer main loop for every connected session; sessions with a compatible toolchain and Cargo configuration attach to the same analysis backend, sharing work instead of duplicating it.

Each release pins one upstream version (the `ra_ap_*` crates) and must pass the upstream test suites. To an editor, `analyzed` must behave like a correct `rust-analyzer` instance, with the same configuration, features, and diagnostics. External behavior stays identical to upstream; only the internal sharing differs.

## Architecture

A SharedWorld is the sharing boundary. It holds one AnalysisHost, one RootDatabase, and the loaded workspaces for that world.

- The registry groups sessions by `SharedAnalyzerWorldKey`. Incompatible toolchains or Cargo/load settings get their own world.
- Workspace roots, linked projects, excluded paths, and client analysis settings belong to the view. A session is a view over a world, not another database, and may see only part of the merged crate graph.
- When a workspace loads into a world, its source roots and crate graph merge into the shared database. Crate sharing relies on salsa interning: identical crate inputs re-intern to the same ID, so shared dependencies are analyzed once.
- SharedWorld is the only path to the database. New write paths must go through its input application code instead of touching the host directly, otherwise other sessions see inconsistent state or miss coordination.
- Open files stay on the shared base while their contents match disk. Unsaved changes use session-local overlay file IDs and overlay crate cones. When the buffer converges back to the on-disk text, the overlay is removed and the session returns to the shared path.

Do not add per-test or per-session world separation. The project only proves its value when real sharing works. Type interner GC is process-global: it runs only when no session in any world is busy.

## Patching

The `analyzed-ra*` crates patch unpacked upstream sources in `build.rs`, then compile the result from `OUT_DIR`.

We keep our modifications to the upstream code minimal. Copying large blocks of upstream logic into our own files is the wrong approach. Instead:

- Patches are declared in `build.rs` with typed parameters and applied through the syntax editing helpers in `analyzed-bridge`. Anchor on symbols and structure only: no ordinal positions, no statement text, no textual search.
- Work at the symbol level where possible: adjust visibility, rename a function, add a field or parameter.
- When a function body must change: rename the upstream function to `_original_name`, inject a replacement with the original name that delegates to `_original_name` where possible. When delegation isn't feasible, mark `_original_name` as `#[allow(dead_code)]`.
- For logic that's inlined inside a large function and can't be reached otherwise: extract the relevant region into a method, then follow the same rename-and-replace approach. Do this only when necessary.
- Logic duplicated from upstream (excluding variable renames and type adjustments) should not exceed roughly 30% of the function. A few-line helper is fine. The threshold is about real duplication, not counting lines.
- Don't alter upstream execution flow or response timing to avoid duplication. The external behavior must stay identical to upstream.
- Injected names carry no ownership markers. A replacement keeps the upstream name; a new item takes a name in upstream style, checked for collisions; an owned module whose name clashes with an upstream file takes a `shared_` prefix. The patch declarations in `build.rs` are the authoritative list of injection points.

Patches apply in order; an earlier rename changes what later anchors resolve. Don't edit generated files. Fix the patch source and rebuild. When bumping the upstream pin, update the whole `ra_ap_*` family together with the `[package.metadata.upstream]` release and tag pins, and mirror upstream workspace dependency changes in the bridge manifests; the build derives and verifies the version identity, there is no manual version mapping. The release facts (target matrix, runners, build flags, PGO setup, packaging) live in `xtask`'s target table, and the release workflow derives its job matrix from `cargo xtask matrix`. They mirror the upstream release configuration; nothing syncs them to upstream automatically, so reconcile them in the same bump.

## Platform & IPC

Endpoints:

- Linux: `$XDG_RUNTIME_DIR/analyzed/daemon.sock`
- macOS: `$TMPDIR/analyzed/daemon.sock`
- Windows: `\\.\pipe\analyzed.<USERNAME>`, overlapped I/O

The daemon leaves nothing in the user's home directory. On Unix, the runtime directory holds only the socket.

## Verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zetaloop/analyzed](https://github.com/zetaloop/analyzed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
