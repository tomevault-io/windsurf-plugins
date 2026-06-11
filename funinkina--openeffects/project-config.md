---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & test commands

```bash
# Build everything
cargo build --workspace
cargo build --workspace --release

# Build a single crate
cargo build -p openeffectsd
cargo build -p openeffects
cargo build -p openeffectsctl

# Run all tests
cargo test --workspace

# Run tests for one crate
cargo test -p shared
cargo test -p openeffectsd

# Run a single test by name
cargo test -p shared config::tests::toml_round_trip
cargo test -p openeffectsd pipeline::effects::tests::effects_bin_builds_without_panic

# Integration tests that require a live daemon are marked #[ignore]; run them with:
cargo test -p openeffects-integration-tests -- --include-ignored

# Lint / format
cargo clippy --workspace -- -D warnings
cargo fmt --all -- --check
```

The CI workflow (`.github/workflows/ci.yml`) runs `cargo fmt --check` and `cargo test --workspace`. Building the `gui` crate requires `gtk4` and `libadwaita` dev packages (pkg-config) in addition to the GStreamer/PipeWire deps below.

## Architecture

### Process model

Three binaries communicate exclusively over D-Bus session bus (`org.openeffects.Daemon`, `/org/openeffects/Daemon`):

```
openeffectsd ──D-Bus──► openeffectsctl      (ad-hoc CLI)
             ──D-Bus──► openeffects         (GTK4/libadwaita GUI, on-demand)
```

The daemon is the only process that touches GStreamer, PipeWire, or cameras. All clients are stateless D-Bus consumers.

### D-Bus interfaces

Three interfaces live at the same object path, all defined in `data/dbus/*.xml`:

| Interface                  | Purpose                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| `org.openeffects.Daemon1`  | Pipeline lifecycle (`Start`, `Stop`), `Status` property, `StatusChanged` signal             |
| `org.openeffects.Effects1` | Effect toggles and params (`SetEnabled`, `SetParam`, `GetAllState`), `EffectChanged` signal |
| `org.openeffects.Devices1` | Camera enumeration and selection, `VirtualCameraInfo` property                              |

String constants for all three are in `shared/src/dbus.rs`. **When you modify a `.xml` file, `build.rs` in `daemon`, `cli`, and `gui` automatically regenerates proxy code into `$OUT_DIR/proxies.rs`** via `zbus_xmlgen`. You do not need to hand-edit generated code.

### Daemon internals (`daemon/`)

- `src/main.rs` — registers three zbus `#[interface]` structs on the session bus and drives the pipeline event loop
- `src/state.rs` — `DaemonState` holds `AppState` (config) + runtime fields; `DaemonStatus` enum guards valid state transitions
- `src/dbus_server.rs` — implements all three D-Bus interfaces; state mutations go through `Arc<RwLock<DaemonState>>`; pipeline commands go through `mpsc::Sender<PipelineCommand>`
- `src/pipeline/` — the virtual camera is a **two-stream + userspace bridge** design (see the "On-Demand PipeWire Virtual Camera" model). The provide side is a **native libpipewire** node; the capture side is GStreamer (elements are not `Send`, so they stay on the worker thread):
  - `provider.rs` — native `pw_stream` `Video/Source` node (`media.class=Video/Source`, `node.name=openeffects`), runs its own `pw_main_loop` on a dedicated thread. **The on-demand hinge**: its `state_changed` callback maps `STREAMING → CaptureCmd::Start` (open camera, LED on) and `PAUSED`/`UNCONNECTED` → `CaptureCmd::Stop` (tear capture, LED off). `process()` serves the latest frame from the bridge (black placeholder until the first frame) and stamps the `SPA_META_Header` meta; `param_changed` answers the `Buffers` **and** `Meta(Header)` params after format negotiation.
  - `bridge.rs` — `Bridge`: a `Mutex<Option<Vec<u8>>>` latest-frame slot, `Arc`-shared between the appsink writer and the provider reader. Newest frame overwrites the previous; `clear()` on capture stop so no stale frame is served on reconnect.
  - `builder.rs` — builds the capture pipeline only: `source → capsfilter(native WxH@fps, raw|jpeg) → decodebin → videoconvert → videoscale → capsfilter(I420 WxH@fps) → effects_bin → appsink`, where the appsink callback writes each processed frame into the bridge. The **source capsfilter pins the camera to the same mode the virtual camera advertises** (MJPEG allowed — decodebin decodes it), so videoscale never rescales and the aspect ratio is preserved; letting negotiation pick a default raw mode (e.g. 4:3 640x480) and scaling it to the output is what stretched the feed. Source falls back to `videotestsrc` if no camera is available. `resolve_camera()` + `probe_format()` decide camera and mode; `cameras::preferred_format()` probes modes via DeviceMonitor caps (no device open) and prefers exact 1280x720, else the mode with area closest to it at ≥ 24 fps.
  - `probe.rs` — now just holds `PIPEWIRE_NODE_NAME` (`"openeffects"`); there is no GStreamer output sink to probe anymore.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [funinkina/openeffects](https://github.com/funinkina/openeffects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
