---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

This is a monorepo for **Desktop Homunculus**, a cross-platform desktop mascot application built with the Bevy game engine. It renders transparent-window VRM 3D characters with WebView-based UI overlays.

```
desktop-homunculus/
├── engine/              # Main Bevy application (Rust workspace)
│   ├── crates/          # Rust plugin crates (homunculus_*)
│   │   └── homunculus_cli/  # Rust CLI binary (hmcs)
│   ├── src/main.rs      # App entry point — composes all plugins
│   └── assets/mods/     # Installed mods (runtime)
├── packages/
│   ├── sdk/             # @hmcs/sdk — TypeScript SDK for mods/extensions
│   ├── ui/              # @hmcs/ui — Shared React component library (Radix + Tailwind)
│   ├── cli/             # @hmcs/cli — Node CLI wrapper (distributes platform-specific Rust binary)
│   └── openclaw-plugin/ # @hmcs/openclaw-plugin — OpenClaw plugin bridging external agents
├── mods/                # Mods (NPM packages): persona/, character-settings/, settings/, menu/, assets/, elmer/, voicevox/, app-exit/, stt/, rpc-test/
├── docs/website/        # Docusaurus documentation site
└── sandbox/             # Dev sandbox — aggregates all mods for workspace linking validation
```

Sub-directories have their own CLAUDE.md with detailed architecture: `engine/`, `packages/sdk/`, `packages/ui/`.

## Development Commands

### Workspace (from repo root)

```bash
pnpm install          # Install all workspace dependencies
pnpm build            # Build all packages in dependency order (turbo)
pnpm dev              # Start all dev watchers (turbo)
pnpm check-types      # Type-check all packages (turbo)
pnpm test             # Run all TypeScript tests (turbo)
make setup            # pnpm install + engine tooling setup + CEF framework download
make debug            # pnpm build (excl. docs) + cargo run (debug with inspector)
make debug-cuda       # Same as debug but with CUDA STT support
make test             # pnpm test (TS) + cargo test --workspace (Rust)
make fix-lint         # cargo clippy --fix + cargo fmt (Rust) + pnpm lint:fix (TS)
make gen-open-api     # Regenerate OpenAPI spec + Docusaurus API docs + pnpm build
make release-macos    # pnpm build + native arch release → PKG
make release-windows  # pnpm build + MSI installer via WiX 4.x (Windows only)
make install-cli      # cargo install the hmcs CLI binary
make stage-runtime    # Download and stage Node.js, pnpm, tsx for bundling
make install-openclaw-plugin  # Build @hmcs/openclaw-plugin and install it into OpenClaw
```

### Engine (Rust) — run from `engine/`

```bash
make debug               # cargo run --features develop (bevy_egui inspector + CEF debug)
make test                # cargo test --workspace
make fix-lint            # cargo clippy --workspace --fix --allow-dirty && cargo fmt --all
make gen-open-api        # Regenerate OpenAPI spec via gen_openapi binary
```

Single test:
```bash
cargo test -p homunculus_http_server            # All tests in one crate
cargo test -p homunculus_http_server test_health # Single test by name
```

Release builds use `--profile dist` (not `--release`), which enables `lto = "thin"` and `strip = true`:
```bash
make release-macos           # Native arch → .app bundle → PKG
make release-macos-arm       # Apple Silicon
make release-macos-x86       # Intel
make release-macos-universal # Universal binary (ARM + x86)
```

### First-time setup (from `engine/`)

```bash
make setup               # Install all Rust/Node tools + download CEF framework (~300MB, skipped if present)
make setup-cef            # Download CEF framework only (macOS; skips if already installed)
```

### TypeScript SDK — run from `packages/sdk/`

```bash
pnpm build               # Rollup → ESM/CJS + bundled .d.ts
pnpm dev                 # Watch mode
pnpm check-types         # tsc --noEmit
```

### Shared UI Library — run from `packages/ui/`

```bash
pnpm build               # Vite library build → dist/ (ES + UMD + rolled .d.ts)
pnpm check-types         # tsc --noEmit
pnpm lint                # ESLint
```

### UI Mod Apps — run from `mods/{settings,menu}/ui/`

```bash
pnpm dev                 # Vite dev server
pnpm build               # Vite build → dist/
```

### Documentation Site — run from `docs/website/`

```bash
pnpm dev                 # Docusaurus dev server (English)
pnpm dev:ja              # Docusaurus dev server (Japanese)
pnpm build               # Production build
```

## Architecture Overview

The engine is built from ~20 independent Bevy plugins in `engine/crates/`, following a Core → API → HTTP layering. The HTTP API (Axum on `localhost:3100`) bridges async requests to Bevy's single-threaded ECS via the `ApiReactor` pattern. See `engine/CLAUDE.md` for detailed Rust architecture, code examples, and crate descriptions.

Asset path resolution: dev mode uses `assets/` relative to `CARGO_MANIFEST_DIR`; release uses `../Resources/assets` (inside `.app` bundle).

### OpenClaw Integration

DH does not include an in-process agent runtime. For AI-powered interaction, run
[OpenClaw](https://docs.openclaw.ai) externally and install the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [not-elm/desktop-homunculus](https://github.com/not-elm/desktop-homunculus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
