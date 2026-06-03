---
trigger: always_on
description: Guidance for AI assistants working in this repository. Read this before making changes.
---

# AGENTS.md

Guidance for AI assistants working in this repository. Read this before making changes.

## Project overview

**vidcord** is a cross-platform desktop app that compresses video files under Discord's size limits. It is built with **Tauri 2** (Rust backend + React/TypeScript frontend) and shells out to the system **FFmpeg** binary for all video work. It does **not** bundle FFmpeg — the system `ffmpeg`/`ffprobe` must be on `PATH`.

- **App version**: kept in sync across `package.json`, `src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`, and any `vX.Y` references in source/docs (see "Bumping the version" below)
- **Window**: fixed-size 460×630, non-resizable, transparent (macOS Tahoe "liquid glass" styling)
- **Supported OS/arch**: Windows (x86_64 + aarch64), macOS (universal), Linux (x86_64 + aarch64)
- **Node**: `^20.19.0 || >=22.12.0` (see `package.json` engines)
- **Rust**: stable toolchain, edition 2021

## Repository layout

```
.
├── src/                       # React frontend (TypeScript)
│   ├── App.tsx                # Monolithic root component — wires hooks, UI, trim timeline, compression
│   ├── App.css                # All styling (dark theme + light theme via data-theme)
│   ├── ErrorBoundary.tsx      # Top-level crash recovery UI
│   ├── main.tsx               # ReactDOM entry
│   ├── index.css              # Global CSS vars (--accent, --surface, --blur…)
│   ├── components/
│   │   ├── PreviewPane.tsx    # Video preview + scrub thumbnail + filmstrip
│   │   ├── ProgressSection.tsx# Compression progress bar + ETA
│   │   ├── Toast.tsx          # Single toast row
│   │   └── EncodersDialog.tsx # Lazy-loaded FFmpeg encoder list dialog
│   ├── hooks/
│   │   ├── useCompression.ts  # Event listeners + pure bitrate/dimension helpers (tested)
│   │   ├── useEncoders.ts     # detect_encoders + FFmpeg-missing tracking
│   │   ├── useSettings.ts     # load_settings / debounced save_settings
│   │   └── useToasts.ts       # Toast queue with per-id timer cleanup
│   ├── __tests__/             # Vitest tests (node env, Tauri APIs mocked)
│   └── __mocks__/@tauri-apps/ # Invoke/listen stubs so pure helpers run in Node
├── src-tauri/                 # Rust backend
│   ├── src/
│   │   ├── main.rs            # Thin entry — calls lib::run()
│   │   ├── lib.rs             # Tauri builder, plugins, Linux env shims, file-open routing
│   │   ├── ffmpeg.rs          # probe / preview / filmstrip / encoder detection / VAAPI discovery + caches
│   │   ├── gpu.rs             # Vendor detection (lspci / system_profiler / Get-CimInstance)
│   │   ├── log.rs             # Rotating ~/…/vidcord/vidcord.log (5 MB cap)
│   │   ├── settings.rs        # Typed settings persisted via atomic rename
│   │   └── commands/          # #[tauri::command] handlers
│   │       ├── compression.rs # compress_video, cancel_compression, preview/probe wrappers
│   │       ├── encoders.rs    # detect/install/list FFmpeg; platform install flows
│   │       ├── files.rs       # show_in_file_explorer, resolve_output_path, get_os, PendingFile
│   │       └── updates.rs     # GitHub latest-release checker (semver)
│   ├── capabilities/default.json  # Tauri permissions (dialog, opener, core)
│   ├── tauri.conf.json        # Product config, CSP, file associations, bundle targets
│   ├── Cargo.toml             # release/ci/dev profiles (see "Build profiles")
│   └── .cargo/audit.toml      # RUSTSEC ignore list for Tauri upstream advisories
├── .github/workflows/build.yml# Multi-platform CI + release workflow
├── index.html                 # Vite entry
├── vite.config.ts             # React plugin, manual chunks (react / tauri / app)
├── vitest.config.ts           # Node env + Tauri-api mock aliases
├── eslint.config.js           # ESLint flat config (TS + react-hooks)
├── .prettierrc                # 100-col, 2-space, double-quote, ES5 trailing commas
├── tsconfig.json              # Strict TS, ES2021 target, react-jsx
├── FFMPEG_SETUP.md            # End-user FFmpeg install guide (per platform)
├── CHANGELOG.md               # User-facing release notes
└── README.md                  # Public overview
```

## Development workflows

### Run the app locally

```sh
npm install
npm run tauri dev          # starts Vite on :5173, launches the Tauri window
```

FFmpeg must be on `PATH` for the app to probe videos or compress.

### Quality gates (mirror CI)

```sh
npm run lint               # eslint src
npm run typecheck          # tsc --noEmit
npm test                   # vitest run
npm run format             # prettier --write src
cargo fmt --check --manifest-path src-tauri/Cargo.toml
cargo clippy --manifest-path src-tauri/Cargo.toml --tests -- -D warnings
cargo test --manifest-path src-tauri/Cargo.toml
cargo audit                # in src-tauri/ (respects .cargo/audit.toml)
```

CI treats any clippy warning as an error — keep new Rust code warning-clean.

### Build a release binary

```sh
npm run tauri build        # outputs to src-tauri/target/release/bundle/
```

### Build profiles (src-tauri/Cargo.toml)

- `release` — LTO, `codegen-units=1`, `strip`, `panic=abort`. Used for tagged releases.
- `ci` — inherits release with `lto=false`, `codegen-units=4`, `opt-level=1`. Used by non-tag CI builds for speed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyroz1/vidcord](https://github.com/cyroz1/vidcord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
