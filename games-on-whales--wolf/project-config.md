---
trigger: always_on
description: Wolf is a low-latency streaming server for [Moonlight](https://moonlight-stream.org/) that lets multiple
---

## What Wolf is

Wolf is a low-latency streaming server for [Moonlight](https://moonlight-stream.org/) that lets multiple
remote clients share a single Linux host to play games. Each client gets an on-demand virtual desktop
(Wayland compositor, no physical monitor needed) whose apps run in isolated Docker/Podman containers.
It implements the Moonlight protocol (pairing over HTTPS, RTSP handshake, ENet control channel, RTP
video/audio) and hands video/audio off to GStreamer pipelines. Linux + Docker first; C++20.

## Companion repositories

Wolf is split across three repos; the other two are integral and pulled in at build time — when
touching virtual input or virtual-display behavior, the real implementation often lives there:

- **[games-on-whales/inputtino](https://github.com/games-on-whales/inputtino)** — virtual input device
  library (`uinput`/`uhid`). Wolf uses it for gamepads (incl. gyro/accel) and pen/touch; mouse and
  keyboard don't go through it (see Architecture). Surfaced via `src/core/.../input.hpp`.
- **[games-on-whales/gst-wayland-display](https://github.com/games-on-whales/gst-wayland-display)** —
  the custom micro Wayland compositor (Rust, built on [Smithay](https://github.com/Smithay/smithay);
  we track a fork at [games-on-whales/smithay](https://github.com/games-on-whales/smithay)). Creates
  on-demand desktops and exposes the raw framebuffer as a GStreamer plugin + C API. Installed as
  `libgstwaylanddisplay` (surfaced via `src/core/.../virtual-display.hpp`); its `.so` must be on
  `GST_PLUGIN_PATH` at runtime and to run the tests.

Prebuilt guest-app containers live in [games-on-whales/gow](https://github.com/games-on-whales/gow).

## Build & test

Two supported paths, both documented in `docs/modules/dev/pages/manual_build.adoc`:

- **Devcontainer (recommended)** — `docker/wolf.Dockerfile` target `wolf-builder` via `.devcontainer/`,
  so you build in the exact environment of the official image with all deps preinstalled (VS Code:
  *Dev Containers: Clone Repository in Container Volume*, pick the Clang kit).
- **Manual host build** — build Wolf outside Docker (Docker must still be installed for Wolf to do
  anything useful). The doc covers building GStreamer and `gst-wayland-display` from source, apt deps,
  the required `LD_LIBRARY_PATH`/`PKG_CONFIG_PATH`/etc. env, and a `runwolf.sh` template of `WOLF_*`
  runtime vars.

Most C++ deps are fetched at configure time via CMake `FetchContent` (fmt, tomlplusplus, reflect-cpp,
eventbus, Catch2, immer, boost via `BoostLoader`); system libs still needed include Boost, GStreamer,
Wayland, libinput, libevdev, libudev, OpenSSL, PulseAudio, libdrm, libpci.

```bash
# Configure (matches CI). CI uses C++20; manual_build.adoc still shows 17 — prefer 20.
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_STANDARD=20 -DCATCH_DEVELOPMENT_BUILD=ON

ninja -C build wolf         # server binary → build/src/moonlight-server/wolf
ninja -C build wolftests    # test suite (Catch2)
cd build/tests && ./wolftests
```

Run a single test / subset with Catch2 selectors: `./wolftests "test name"`, `-c "section"`,
`"[tag]"`, or `--list-tests`. Tests need `HOST_APPS_STATE_FOLDER`, `GST_PLUGIN_PATH` (where
`libgstwaylanddisplay` is installed), `XDG_RUNTIME_DIR`, and `RUST_LOG` set (see the `test` job in
`.github/workflows/linux-build-test.yml`).

Hardware/environment-dependent tests are gated by CMake options so CI can skip what a runner lacks:
`TEST_DOCKER`, `TEST_NVIDIA`, `TEST_VIRTUAL_INPUT`, `TEST_UHID`, `TEST_RUST_WAYLAND`, `TEST_EXCEPTIONS`,
`TEST_SDL`. CI runs a g++/clang matrix with `BUILD_SHARED_LIBS` both ON and OFF — keep both working.

C++ is auto-formatted with clang-format (`.clang-format`, 120 col) and enforced in CI; run
`clang-format -i` on changed files before committing. `.clang-tidy` is also present.

## Architecture

**Style (deliberate, per the original author):** functional — no global state, side effects avoided,
immutable inputs → new outputs. Shared/persistent state lives in [immer](https://github.com/arximboldi/immer)
persistent containers wrapped in `immer::atom<...>` (e.g. `SessionsAtoms`, `PairedClientList`): treat a
snapshot as read-only and swap in a new one to update, never mutate in place. This is what makes Wolf's
heavy concurrency (many simultaneous users) lock-free and safe. Prefer pure functions over shared
mutable state, and keep protocol logic decoupled from the server runtime.

**Coordination:** components talk through a shared **event bus** (`dp::event_bus`, `EventBusType` in
`events/events.hpp`) rather than direct calls — `PairSignal`, `StreamSession`, `CreateLobbyEvent`,
`StartRunner`, `PlugDeviceEvent`, docker lifecycle events, etc. For cross-component behavior, define/
handle an event instead of adding a direct dependency. New events go in `events/events.hpp`;
`reflectors.hpp` exposes them for serialization (all config/API/event serialization uses reflect-cpp —
annotate types rather than hand-writing parsers).

### How a stream works

See `docs/modules/dev/pages/how-it-works.adoc` for the full picture.

- **Virtual desktop** — `gst-wayland-display` creates a desktop on demand and feeds its raw framebuffer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [games-on-whales/wolf](https://github.com/games-on-whales/wolf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
