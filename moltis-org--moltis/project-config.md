---
trigger: always_on
description: Moltis engineering guide for Claude/Codex agents: Rust architecture, testing, security, and release workflows
---


# CLAUDE.md

Rust version of openclaw ([docs](https://docs.openclaw.ai), [code](https://github.com/openclaw/openclaw)).
All code must have tests with high coverage. Always check for security.

## Cargo Features

Enable new feature flags **by default** in `crates/cli/Cargo.toml` (opt-out, not opt-in):
```toml
[features]
default = ["foo", ...]
foo = ["moltis-gateway/foo"]
```

## Workspace Dependencies

Add new crates to `[workspace.dependencies]` in root `Cargo.toml`, reference with `{ workspace = true }`.
Never add versions directly in crate `Cargo.toml`. Use latest stable crates.io version.

## Config Schema and Validation

When adding/renaming fields in `MoltisConfig` (`crates/config/src/schema.rs`), also update
`build_schema_map()` in `crates/config/src/validate.rs`. New enum variants for string-typed
fields need updates in `check_semantic_warnings()`.

## Rust Style and Idioms

- **File size limit: 1,500 lines.** CI enforces this via `scripts/check-file-size.sh`. Split large files into modules by domain. Existing oversize files are allowlisted for incremental decomposition.
- Do not add implementation code to `mod.rs` or `lib.rs`. Keep those files for module wiring, exports, and crate setup, move real logic into dedicated sibling modules.
- Use traits for behaviour boundaries. Prefer generics for hot paths, `dyn Trait` for heterogeneous/runtime dispatch.
- Derive `Default` when all fields have sensible defaults.
- Use concrete types (`struct`/`enum`) over `serde_json::Value` wherever shape is known.
- **Match on types, never strings.** Only convert to strings at serialization/display boundaries.
- Prefer `From`/`Into`/`TryFrom`/`TryInto` over manual conversions. Ask before adding manual conversion paths.
- Prefer streaming over non-streaming API calls.
- Run independent async work concurrently (`tokio::join!`, `futures::join_all`).
- Never use `block_on` inside async context.
- **Forbidden:** `Mutex<()>` / `Arc<Mutex<()>>` — mutex must guard actual state.
- Use `anyhow::Result` for app errors, `thiserror` for library errors. Propagate with `?`.
- **Never `.unwrap()`/`.expect()` in production.** Workspace lints deny these. Use `?`, `ok_or_else`, `unwrap_or_default`, `unwrap_or_else(|e| e.into_inner())` for locks.
- Use `time` crate (workspace dep) for date/time — no manual epoch math or magic constants like `86400`.
- Prefer `chrono` only if already imported in the crate; default to `time` for new code.
- Prefer crates over subprocesses (`std::process::Command`). Use subprocesses only when no mature crate exists.
- Prefer guard clauses (early returns) over nested `if` blocks.
- Prefer iterators/combinators over manual loops. Use `Cow<'_, str>` when allocation is conditional.
- Keep public API surfaces small. Use `#[must_use]` where return values matter.

### Tracing and Metrics

All crates must have `tracing` and `metrics` features, gated with `#[cfg(feature = "...")]`.
Use `tracing::instrument` on async functions. Record metrics at key points (counts, durations, errors).
See `docs/metrics-and-tracing.md`.

## Build Commands

```bash
cargo build                  # Debug build
cargo build --release        # Release build
cargo run / cargo run --release
```

## Web UI (TypeScript + Preact + Vite)

TypeScript/TSX source in `crates/web/ui/src/`, built with Vite to `crates/web/src/assets/dist/`.
CSS and static assets in `crates/web/src/assets/`. Release mode embeds via `include_dir!`.
Both `dist/` and `style.css` are committed (unminified) so `cargo build` works without Node.js
and diffs merge cleanly. See `docs/src/frontend.md` for the full architecture guide.

### Build Commands

```bash
cd crates/web/ui
npm run build          # Vite: TS/TSX → dist/ (MUST commit dist/ after)
npm run build:css      # Tailwind: input.css → ../src/assets/css/style.css
npm run build:sw       # esbuild: src/sw.ts → ../src/assets/sw.js
npm run build:all      # All three above
npm run dev            # Vite watch mode (rebuilds on save)
npx tsc --noEmit       # Type check (strict, must be 0 errors)
```

**After changing TS/TSX files**, always:
1. `biome check --write crates/web/ui/src/`
2. `cd crates/web/ui && npm run build`
3. `cd crates/web/ui && npx tsc --noEmit`
4. Commit both the source changes AND the `dist/` output

### TypeScript Rules

- **File size limit: 1,500 lines** (same rule as Rust). Split large files into modules by domain.
  - Pages: extract sections/modals into `pages/sections/`, `pages/channels/`, `pages/chat/`, etc.
  - Utilities: extract sub-modules into sibling directories (`providers/`, `sessions/`, `ws/`).
  - Keep shared signals, types, and re-exports in the main file; move logic into sub-modules.
- All UI code is **TypeScript** with **JSX** (Preact). No HTM tagged templates.
- Add typed Props interfaces for all Preact components.
- Use `@preact/signals` with generic type parameters: `signal<string[]>([])`.
- Prefer typed interfaces over `Record<string, unknown>` — define concrete shapes where property access is known.
- Use `targetValue(e)` / `targetChecked(e)` from `typed-events.ts` for form event handlers.
- No `any` types — use `unknown` with type guards or specific interfaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moltis-org/moltis](https://github.com/moltis-org/moltis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
