---
trigger: always_on
description: OpenVids is a standalone, agent-native desktop video editor derived from an initial HyperFrames snapshot.
---

# OpenVids

OpenVids is a standalone, agent-native desktop video editor derived from an initial HyperFrames snapshot.

## Architecture

- **Desktop Shell**: Tauri 2 (`apps/desktop`), shipping on macOS (`.app` + `.dmg`) and Windows x64 (NSIS per-user `*-setup.exe`, WebView2 download bootstrapper).
- **Frontend / Editor**: HyperFrames-derived Studio served from a local loopback HTTP server (`cli` launches the Studio server; SPA + `/api`; composition iframe).
- **Core Constraints**:
  - Studio and the composition iframe rely on same-origin synchronous DOM access. Never move Studio to `tauri://`, never introduce a second wrapper editor UI, and do not migrate off Tauri without a concrete blocker.
  - Project and source files on disk are the single source of truth.
  - Hard kills (SIGKILL, crash) are recovered, never cleaned up by handlers: history keeps open agent windows in `<history home>/open-windows.json` and files a dead owner's writes to the turn's own entry on the next start (so `Revert this turn` still works); the Studio server sweeps `renders/work-*` and `.*.hf-transaction-*` at start and the engine kills Chrome whose owner died (`<tmp>/hyperframes-browsers/<pid>.json`, `sweepOrphanBrowsers`). On macOS/Linux teardown is process groups + SIGTERM/SIGKILL; on Windows it is kill-on-close Job Objects in `apps/desktop/src-tauri/src/proc.rs` (`KILL_ON_JOB_CLOSE`, `CREATE_NO_WINDOW`), so quit and Task-Manager kills both reap bun/Chrome/ffmpeg, and the orphan sweep has a win32 `taskkill` path. Keep new long-lived state recoverable the same way.
  - Anonymous usage statistics are sent by the Tauri shell only (`apps/desktop/src-tauri/src/telemetry.rs`; gated by `telemetry.enabled`, `DO_NOT_TRACK`, `OPENVIDS_TELEMETRY`); Studio, `studio-server`, `agent-runtime` and `cli` never send them, and a new event or field lands only together with the README's "Usage statistics" section.
  - Bug reports are an explicit user action from the Tauri shell only (`apps/desktop/src-tauri/src/report.rs`, window label `report`): text, screenshots, an optional email and the redacted log tail leave the machine, the email and logs are never public, the text and screenshots become a public GitHub issue, and the telemetry preference/`DO_NOT_TRACK` do not apply to them. Docs never name the servers behind statistics and reports or how they are run.

## Windows notes

- Custom frameless window (`decorations(false)`; the pages draw minimize / maximize / close); `OPENVIDS_SYSTEM_FRAME=1` falls back to the OS frame. No visible menu bar on the custom frame: the native menu exists for its `Ctrl+` accelerators (`Ctrl+O` / `Ctrl+Shift+O` / `Ctrl+R`, mirrored as in-page handlers) plus the system-frame fallback. `Ctrl+,` (Settings) is not a native accelerator: it works only through the in-page handlers (`apps/desktop/src-tauri/src/home_page/home.js`, `packages/studio/src/hooks/useOpenvidsDesktopShortcuts.ts`).
- FFmpeg is not bundled. Resolution order: an explicit `HYPERFRAMES_FFMPEG_PATH` / `HYPERFRAMES_FFPROBE_PATH` in the environment, then the copy the app's download button installed (`managed_env()` points every child at it and prepends it to `PATH`, so it beats an FFmpeg on `PATH`), then `PATH`. The button fetches the official gyan.dev essentials build into `%USERPROFILE%\.openvids\ffmpeg` (`ffmpeg_install.rs`, applied per spawn via `managed_env()`). The archive URL and its SHA-256 are pinned in the binary, the whole download-and-swap holds an exclusive `ffmpeg.lock`, and an install that a crash or kill interrupted is recovered at the next start (`recover_managed_install`) before any sidecar runs: the newest complete backup is restored, never the only complete copy deleted.
- Rust `home_dir()` resolves exactly like Node `os.homedir()` (`USERPROFILE` on Windows), so `~/.openvids` is the same directory on both sides.
- Drops onto the Projects page are file-picker-only in v1 (`drop_paths.rs` has no Windows pasteboard equivalent); copy says Explorer / Recycle Bin via `.win` locale keys. `stage-runtime.mjs` stages `bun.exe` + win32-only natives and strips `*.map`/`*.d.ts` for `MAX_PATH` headroom under the NSIS per-user install dir. CI runs macOS + Windows; symlink-based tests skip without Developer Mode / elevation.

## Package Manager & Commands

OpenVids uses **Bun** (not pnpm or npm).

```bash
bun install               # Install dependencies
bun run desktop:dev       # Launch desktop application in development mode
bun run desktop:build     # Build desktop application release bundle
bun run desktop:check     # cargo check for the Tauri shell (no TypeScript typecheck; use bun run typecheck)
bun run build             # Build workspace packages
```

### Linting & Formatting

Uses **oxlint** and **oxfmt** (not eslint, not prettier, not biome):

```bash
bunx oxlint <files>        # Lint
bunx oxfmt <files>         # Format
bunx oxfmt --check <files> # Check formatting
```

## Packages & Directories

### Packages (`packages/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bazodev/open-vids](https://github.com/bazodev/open-vids) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
