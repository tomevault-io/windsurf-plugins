---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dofek** (דּוֹפֶק — Hebrew for "pulse") is a dual-interface, AI-aware system monitor for Windows, Linux, and macOS (Apple Silicon), built with Rust. The TUI uses Ratatui + crossterm, the GUI uses Tauri 2 (WebView2 on Windows, WebKitGTK on Linux, WKWebView on macOS). Both share a common core library for data collection. It uses the `sysinfo` crate for CPU/memory/process/network/hostname data, NVML for NVIDIA GPU metrics and per-process VRAM, and a plugin system for extensibility via JSON-over-stdio.

Targets: Windows 11 (Windows 10 build 19041+), Linux x86_64 (Ubuntu 24.04, Fedora 40, Arch), macOS 12+ on Apple Silicon (`aarch64-apple-darwin` only — Intel Macs are not supported). Single binary per interface, no runtime dependencies.

## Build & Run

Cargo aliases are defined in `.cargo/config.toml` — run all commands from the repo root.

```bash
# Dev (debug, fast compile)
cargo tui                          # Run TUI
cargo gui                          # Run GUI (hot-reload)

# Release builds (LTO + strip)
cargo build-tui                    # → target/release/dofek-tui[.exe]
cargo build-gui                    # → target/release/dofek-gui[.exe] + native bundles

# Native installer / packages (bundles both TUI + GUI)
.\build-all.ps1                    # Windows → target/release/bundle/msi/dofek_*.msi
./build-all.sh                     # Linux   → target/release/bundle/{deb,rpm,appimage}/dofek_*
```

**Prerequisites:** Rust toolchain (stable, edition 2024), Tauri CLI (`cargo install tauri-cli --version "^2"`) for GUI builds, plus per-OS:
- **Windows:** Visual Studio Build Tools with C++ workload.
- **Linux (apt):** `libwebkit2gtk-4.1-dev libayatana-appindicator3-dev librsvg2-dev libssl-dev libgtk-3-dev` — and `rpm` if you want `.rpm` bundles.
- **macOS:** Xcode Command Line Tools (`xcode-select --install`). Apple Silicon only — Intel Macs are not supported.

**Optional for enhanced functionality:**
- NVIDIA GPU + drivers for GPU metrics and per-process VRAM (NVML — `nvml.dll` on Windows, `libnvidia-ml.so` on Linux). Gracefully degrades without it.
- **Windows only:** LibreHardwareMonitor with web server on port 8085 — fallback for CPU temp/power and non-NVIDIA GPU data. On Linux, dofek reads CPU temps directly from `/sys/class/hwmon` via `sysinfo::Components`, so LHM is not needed.

## Architecture

### Dual-Interface Model
```
    dofek (workspace)
    ├── dofek (lib + TUI binary)
    │     ├── sysinfo crate ──── CPU, memory, processes (with CPU%)
    │     ├── NVML ──────────── GPU metrics + per-process VRAM (NVIDIA, multi-GPU)
    │     ├── LHM HTTP ─────── GPU fallback (optional, non-NVIDIA)
    │     ├── Windows API ───── network stats (GetIfTable2), local time
    │     ├── Plugin system ─── JSON-over-stdio child process plugins
    │     └── Ratatui TUI ───── rendering (trading-terminal layout)
    │
    ├── dofek-gui (Tauri 2 desktop app)
    │     ├── Reuses dofek core lib for data collection + plugins
    │     ├── Tauri IPC ────── get_snapshot / get_gpu_info / settings commands
    │     └── Vanilla HTML/CSS/JS frontend with Canvas charts
    │
    └── plugins/
          ├── dofek-ollama ──── Ollama model status + inference tracking
          ├── dofek-docker ──── Docker container monitoring
          └── dofek-net-ping ── TCP-connect latency sampler
```

### Threading Model (sync, no tokio)
- **Main thread**: Render loop + event handling. Receives data via `mpsc::channel`.
- **Data collector thread** (`data::spawn_collector`): Refreshes sysinfo, queries NVML, enumerates network, polls plugins. Sends `DataSnapshot` over channel. The `sysinfo::System` instance lives here (persists across polls for CPU% delta computation).
- **Event reader thread** (`event::spawn_event_reader`): Reads crossterm keyboard events, sends `AppEvent` over channel.

### Module Structure

- `src/main.rs` — Entry point: terminal init, thread spawning, main event/render loop
- `src/lib.rs` — Shared library (used by both TUI and GUI)
- `src/app.rs` — App state: `DataSnapshot`, `HistoryBuffers`, `ChartTab`, `CategoryFilter`, `GpuTab`
- `src/config.rs` — CLI (clap) + TOML config loading with `[categories]` and `[[plugins]]` sections
- `src/settings.rs` — User settings (persisted to `%APPDATA%/dofek/`)
- `src/event.rs` — Crossterm event reader thread, `AppEvent` enum
- `src/data/` — Data collection layer:
  - `mod.rs` — `DataSnapshot` struct (with `gpus: Vec<GpuSensors>`), collector thread
  - `sysinfo_source.rs` — sysinfo-backed CPU, memory, and process extraction
  - `gpu.rs` — NVML wrapper: multi-GPU device metrics + per-process VRAM
  - `lhm.rs` — LHM HTTP client (optional GPU fallback, multi-GPU aware)
  - `process.rs` — `ProcessInfo`, `AiState`, `ProcessCategory` definitions
  - `network.rs` — Per-interface rx/tx bytes with delta-based rate computation. Windows uses `GetIfTable2`; Linux and macOS share the `sysinfo::Networks` path. The macOS branch additionally filters Apple-internal pseudo-interfaces (`awdl0`, `llw0`, `gif0`, `stf0`, `anpi0`/`1`, `ap1`). All three platforms share the `NetworkTracker` state struct.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsafSaar/dofek](https://github.com/AsafSaar/dofek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
