---
trigger: always_on
description: Context for Claude Code working on veiland. Read this before making changes.
---

# CLAUDE.md

Context for Claude Code working on veiland. Read this before making changes.

## What veiland is

A Wayland screen locker with process-isolated, GPU-accelerated plugins. Veiland-core handles the lock lifecycle, PAM authentication, keyboard input, and compositing. Plugins handle everything else (clocks, wallpapers, widgets, animated backgrounds) and run as separate processes that communicate with the core over a Unix socket. Plugins render with OpenGL into GPU buffers and share those buffers with the core via DMA-BUF — no CPU-side pixel copying, full process isolation.

## Current status

The locker works: 
- `ext-session-lock-v1`
- surface
- PAM authentication
- keyboard input
- password indicator
- process-isolated GPU plugins over DMA-BUF,
- multi-monitor
- reliability: Hyprland (NVIDIA) survives suspend/resume, DPMS off/on, and monitor unplug/replug (incl. fast replug, unplug-all, hotplug-in while locked). Unplug/replug also passes on Sway; Sway suspend/DPMS sweep still pending. See the two "Formerly-known issue" notes below for the hotplug crashes found and fixed here.


Reference plugins:
- wallpaper
- clock
- label
- vignette
- particle family (particles, sakura, snow, rain, embers, fireflies)
- ambient backgrounds/overlays (gradient, parallax, blobs)
- plus one test plugin: stress (load generator).

The codebase is structured as a Cargo workspace:
- `veiland-core` (locker binary)
- `veiland-plugin` (SDK library)
- `veiland-protocol` (wire format)
- `veiland-text` (text rendering helper)
- and per-plugin crates under `plugins/`.

**Known limitations / open work:**

- **Per-plugin frame rate:** all plugins run at the compositor's repaint rate. Deferred.

**Formerly-known issue, now resolved:** the Hyprland fast-replug crash (unplug + replug within ~5–10s panicking at `eglSwapBuffers` with `invalid object N`) was caused by the `update_output` rebind path handling Hyprland re-advertising a surviving monitor's `wl_output` under a new global. The output-tracking refactor `cb0e608` (2026-06-13) tracks outputs by registry numeric ID and dropped that compositor-specific rebind path, so the quirk now falls out as a plain remove + add. Testing on Hyprland (2026-07-09) no longer reproduces the crash, including fast/no-wait replug. Keep an eye on it under the dmabuf-import care bar, but it is not a documented open limitation anymore.

**Formerly-known issue, now resolved (2026-07-10):** unplugging *all* monitors then replugging panicked at `eglSwapBuffers: BadSurface` (`lock.rs` bootstrap swap) on both Hyprland and Sway — during the rapid teardown+rebuild the freshly-created EGL surface was already stale by the time we swapped, and both swap sites `.expect()`ed. Fix: both swaps (`lock.rs` bootstrap, `mod.rs` repaint) now match on `Err(egl::Error::BadSurface)` and skip+retry (log, `return`, leave `needs_paint`) instead of crashing; other EGL errors still panic. Session stayed locked throughout regardless (compositor compliance). Residual, non-crash: after unplug-*all* the scene takes a moment to repaint (outputs + plugins rebuilt from scratch; the skip-retry doesn't request an immediate frame callback). Single-monitor unplug is instant. That recovery latency is a deferred polish item, not a bug.

The architecturally critical mechanism — cross-process DMA-BUF buffer sharing — is validated and in production. Changes touching the dmabuf import/sampling path warrant the same care as the auth path.

## Decisions already made — do not re-litigate

- **Name:** veiland. The crate is `veiland-core` (it pairs with `veiland-plugin` and names the trusted core in the workspace/threat-model), but the installed binary users invoke is `veiland` — set via `[[bin]]` in `veiland-core/Cargo.toml`. `veiland-plugin` is the plugin library, `veiland-<name>` the individual plugins.
- **License:** GPL-3.0-or-later. Plugins communicate over a Unix socket, so plugin authors can use any license they want.
- **Language:** Rust. Untrusted IPC input from plugins, long-lived security-sensitive process, concurrent event loops. The Wayland/EGL/GBM bindings live in a small number of FFI-wrapping modules; the rest is safe Rust.
- **Graphics:** OpenGL only. Not Vulkan. Lockscreens composite a handful of textured quads — Vulkan's complexity buys nothing.
- **Plugin isolation:** separate processes. Not `.so` modules. Non-negotiable.
- **Plugin rendering:** DMA-BUF buffer sharing as the primary path. Each plugin has its own EGL context and GBM device.
- **OS target:** Linux only. DMA-BUF and GBM are Linux-specific.
- **Compositor target:** any compositor implementing `ext-session-lock-v1`. Tested on Hyprland and Sway.

## Trust boundaries — load-bearing, do not blur

- **Veiland-core (trusted):** owns the `ext-session-lock-v1` lock surface, holds keyboard focus, runs PAM, manages the password buffer, owns the unlock decision. Composites plugin output onto the lock surface. Never loads untrusted code.
- **Plugin processes (untrusted):** render pixels into buffers and hand the buffers to the core. Receive only the events the core chooses to forward (configuration, time ticks, optionally clicks within their own region). Never receive raw keyboard input. Never see the password buffer. Never make the unlock decision.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sylflo/veiland](https://github.com/sylflo/veiland) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
