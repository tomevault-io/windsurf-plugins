---
trigger: always_on
description: This is an RDP client with:
---

BE BRIEF.

# Agent Notes

## Project Overview

This is an RDP client with:

- `RDPilot.Client`: .NET/Avalonia UI.
- `RDPilot.Wrapper`: native C shared library wrapping FreeRDP 3 APIs.

The client uses `DllImport("freerdp_wrapper")` to load the native wrapper copied beside the .NET output. Linux builds produce `RDPilot.Wrapper/build/native/libfreerdp_wrapper.so`.

Native sessions are handle-based. `rdp_session_connect` returns an opaque `rdp_session*`; resize, input, clipboard, disconnect, and free calls must use that handle. Do not reintroduce singleton native state for connection/session data.

## Build And Run

Use the solution build; the .NET project configures/builds the native CMake wrapper automatically.

```sh
dotnet build RDPilot.slnx
dotnet run --project RDPilot.Client/RDPilot.Client.csproj
```

System dependencies currently expected on Linux:

- .NET SDK 10+
- CMake
- C compiler
- pkg-config/pkgconf
- FreeRDP 3 development files: `freerdp3`, `freerdp-client3`, `winpr3`

## Native/Managed Boundary

Policy lives in safe C#. The native wrapper is a thin FreeRDP host shim with no policy decisions.

**Managed-owned (safe C#, no `unsafe` blocks):**
- Resize debounce/coalescing: `ViewportResolutionUpdateScheduler` (quiet-period + min-interval).
- Pointer-move coalescing/throttling: `PointerMoveScheduler` (latest-only, ~125 Hz min interval).
- Option normalization: `RdpSessionOptions` (color depth, connection type, DPI scale clamping, resolution clamping).
- Clipboard file-list shaping: C# sends paths via native `clear/add/commit` primitives.
- All callback marshaling uses safe `Marshal.PtrToStringUTF8` / `Marshal.ReadIntPtr`.

**Native-owned (FreeRDP mechanics only):**
- FreeRDP event loop, channel setup/callbacks, GDI/RDPGFX framebuffer lifecycle.
- CLIPRDR protocol handling (text, files, bitmap format negotiation).
- RDP-thread input delivery via a simple ordered queue (no coalescing/throttling policy).
- RDP-thread resolution application via `SendMonitorLayout` (no debounce policy).
- Direct FreeRDP settings writes (values are pre-normalized by C#).

Do not reintroduce `unsafe` into `RDPilot.Client.csproj`. Do not reintroduce policy logic (debounce, coalescing, normalization) into the native wrapper.

## Native Wrapper Notes

Important FreeRDP 3 details discovered during dynamic-resolution work:

- Raw `freerdp_new()` does not automatically load client channels.
- The wrapper must set `g_instance->LoadChannels = freerdp_client_load_channels` before connect.
- The wrapper must register the static addin provider with `freerdp_register_addin_provider(freerdp_channels_load_static_addin_entry, 0)`.
- `FreeRDP_SupportDisplayControl = TRUE` enables the `disp` dynamic channel through FreeRDP's client channel loader.
- `FreeRDP_DynamicResolutionUpdate = TRUE` is also set for dynamic-resolution behavior.
- Dynamic resolution uses `DispClientContext->SendMonitorLayout` after the `DisplayControlCaps` callback fires.
- Do not send monitor layout updates directly from Avalonia/UI thread. Queue them and send from the RDP thread.

Current resize behavior (managed-owned policy):

- `ViewportResolutionService` ignores sizes below `640x480` and minimized-window events.
- `ViewportResolutionUpdateScheduler` applies a quiet-period and minimum-interval debounce before forwarding to `RdpSessionViewModel.UpdateResolution`, which normalizes via `RdpSessionOptions` and forwards to native.
- Native queues the latest target size and applies `SendMonitorLayout` on the next RDP loop iteration (no debounce policy).
- Native resizes local GDI framebuffer after a successful layout send so Avalonia can resize its bitmap.

Input behavior (managed-owned policy):

- Do not call FreeRDP input APIs directly from Avalonia/UI event handlers.
- `PointerMoveScheduler` coalesces mouse moves latest-only and throttles to ~125 Hz (8 ms min interval). Pending moves are flushed before button/wheel events so clicks carry a near-current position.
- Native input is a simple ordered FIFO queue drained on the RDP thread. No coalescing or throttling policy in native code.
- The RDP loop is the canonical FreeRDP event-driven form (matches `wfreerdp`/`xfreerdp`): `freerdp_get_event_handles` → `WaitForManyObjects(..., INPUT_LOOP_TIMEOUT_MS=10)` → `freerdp_check_event_handles` (single call). Network data wakes the loop immediately; idle polls are capped at ~100 Hz.
- Current low-latency profile uses 16-bit color, compression, bitmap cache, WAN connection type, disabled audio/device redirection, and disabled desktop visual effects.

## Rendering Notes

The default rendering mode is now `gfx-gdi` (RDPGFX with FreeRDP's standard `gdi_graphics_pipeline_init`, aligned with `wfreerdp`). The custom `rdpgfx-surface` path has been removed; only `gfx-gdi` and the legacy `classic-gdi` mode remain. `classic-gdi` is kept for fallback only and is not the production path. Set `RDPILOT_RENDERING_MODE=classic-gdi` to force the legacy path.

Shared-buffer presentation (mirrors `wfreerdp.exe`):

- `FreeRDP_SoftwareGdi` is now `TRUE` in gfx mode and the wrapper uses FreeRDP's standard `gdi_graphics_pipeline_init` (no `gdi_graphics_pipeline_init_ex`, no `SurfaceCommand`/`EndFrame`/`UpdateSurfaceArea` overrides, no surface-renderer dirty tracking).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ErnieBernie10/RDPilot](https://github.com/ErnieBernie10/RDPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
