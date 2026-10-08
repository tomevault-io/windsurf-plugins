---
trigger: always_on
description: Open-source RGB lighting orchestration engine for Linux, Windows, and macOS, written in Rust.
---

# Hypercolor

Open-source RGB lighting orchestration engine for Linux, Windows, and macOS, written in Rust.

## Quick Start

```bash
# Primary interface: use justfile recipes
just verify          # fmt + lint + test (run after every change)
just check           # Type-check without building
just build           # Debug build (dev profile)
just build-preview   # Near-release performance, fast compile
just release         # Full release build (LTO, stripped)

# Run
just daemon          # Daemon on :9420 (preview profile, debug logging)
just daemon-servo    # Daemon with Servo HTML effect rendering
just tui             # TUI client (auto-starts daemon)
just app             # Unified desktop app with system tray
just cli             # CLI tool (`hypercolor`)
just dev             # Daemon + UI dev server together

# UI / SDK
just ui-dev          # Leptos dev server on :9430, proxies API to :9420
just sdk-dev         # TypeScript SDK dev server with HMR
just effects-build   # Build all HTML effects -> effects/hypercolor/*.html
just effect-build X  # Build single effect by name

# Testing
just test            # All workspace tests
just test-crate X    # Test specific crate
just test-one X      # Run single test by name pattern
just linux-guest-proof run X  # Linux installer scenario in a rootless systemd guest

# Quality
just lint            # Clippy with -D warnings
just lint-fix        # Clippy auto-fix
just deny            # License + advisory audit (cargo-deny)
just doc-open        # Build and open API docs
```

All `just` commands route through `scripts/cargo-cache-build.sh`, which manages
sccache/ccache, clang+lld linking on Linux, and Servo build state.

## Project Structure

```
crates/
  hypercolor-color/                # Color kernel: pixel types, conversions, hex, blending, device encoding; bottom of the graph
  hypercolor-types/                # Shared data vocabulary above the color kernel; every crate depends on it
  hypercolor-core/                 # Engine: render loop, device backends, Servo effect renderer, event bus, spatial sampler, input pipeline, scene/session management
  hypercolor-hal/                  # Hardware abstraction: USB/HID/SMBus protocol encoding and transport for the local driver families
  hypercolor-gpu-frame/            # Neutral imported-GPU-frame vocabulary (format, timings, origin, leases) shared by every interop crate
  hypercolor-worker-retention/     # Leaf: process-wide reaper for workers that outlive bounded shutdown
  hypercolor-pipewire-interop/     # Audited XDG Portal + PipeWire capture boundary (native on Linux, stubs elsewhere)
  hypercolor-linux-gpu-interop/    # Linux GL/Vulkan texture import boundary for Servo frames
  hypercolor-linux-input/          # Linux evdev host keyboard and pointer capture
  hypercolor-linux-session/        # Linux session/power monitors, systemd notify, procfs, parent-death guard
  hypercolor-macos-capture/        # macOS ScreenCaptureKit acquisition and retained frame ownership
  hypercolor-macos-gpu-interop/    # macOS IOSurface/Metal texture import boundary
  hypercolor-macos-input/          # macOS CGEventTap keyboard and pointer capture
  hypercolor-macos-media/          # macOS now-playing media Automation adapters
  hypercolor-macos-session/        # macOS session and system-power monitoring
  hypercolor-macos-owner/          # Shared durable macOS daemon ownership, launchd adapter, kqueue lifetime guard
  hypercolor-windows-gpu-interop/  # Windows D3D11/Vulkan texture import boundary
  hypercolor-windows-pawnio/       # Windows SMBus access via the PawnIO kernel driver, with a broker service; stubbed on other platforms
  hypercolor-windows-capture/      # Windows DXGI Desktop Duplication screen capture
  hypercolor-windows-input/        # Windows Raw Input host keyboard and pointer capture
  hypercolor-windows-session/      # Windows session and power event monitors
  hypercolor-windows-telemetry/    # Windows WMI/PawnIO sensors and board identity (stubs elsewhere)
  hypercolor-windows-helper/       # Signed elevated helper for Windows privileged operations
  hypercolor-platform-fs/          # Audited platform filesystem operations
  hypercolor-install/              # Transactional release installer library: install journal, coordinator, Linux systemd and macOS launchd platforms, payload validation
  hypercolor-persistence/          # Durable file replacement and the process-wide flush registry every store writes through
  hypercolor-driver-api/           # Stable trait/type boundary between the daemon and all driver implementations
  hypercolor-driver-support/       # Native credential and discovery services layered on the driver API
  hypercolor-driver-builtin/       # Compile-time bundle assembling HAL + network drivers into a registry via feature flags
  hypercolor-driver-hue/           # Philips Hue Bridge driver (Entertainment API over DTLS)
  hypercolor-driver-nanoleaf/      # Nanoleaf panels driver (HTTP pairing + UDP external control)
  hypercolor-driver-wled/          # WLED driver (DDP / E1.31 sACN streaming)
  hypercolor-driver-govee/         # Govee smart-lighting driver (LAN UDP + Govee Cloud API)
  hypercolor-openrgb-sdk/          # Clean OpenRGB SDK protocol client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperb1iss/hypercolor](https://github.com/hyperb1iss/hypercolor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
