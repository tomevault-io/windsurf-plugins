---
trigger: always_on
description: A native music streaming client, built with Rust and [GPUI](https://github.com/zed-industries/zed)
---

# sonora

A native music streaming client, built with Rust and [GPUI](https://github.com/zed-industries/zed)
(Zed's UI framework), streaming through [librespot](https://github.com/librespot-org/librespot) and [ytmusic-rs](https://github.com/noligh132/ytmusic-rs). Cargo workspace, edition 2024, resolver 3.

## Read this before writing code

1. **A component probably already exists.** Check `crates/ui/src/lib.rs`, `crates/views/src/shared/cells.rs`
   and the tables below before writing a new element. See [Before you build a component](#before-you-build-a-component).
2. **Never call the Spotify Web API.** There is no `reqwest` call to `api.spotify.com` anywhere and
   there must not be one. All data comes from librespot's `spclient`. See [Backend](#backend-how-data-actually-arrives).
3. **Never hardcode a color, radius, or size.** Everything comes from `cx.theme()`.
   See [Theme and metrics](#theme-and-metrics).
4. **Network work runs on the tokio runtime (`Io`), never on GPUI's executor.**
   See [Async: two runtimes](#async-two-runtimes).
5. **New assets must be registered in `crates/sonora/src/assets.rs`** or they silently fail to load.
6. **Never push changes without the user's explicit confirmation.** Committing does not imply permission
   to run `git push`; ask immediately before every push.

## Crate layout

```
crates/
  sonora/     binary: main, window, actions, asset registry, HTTP client shim
  views/      screens plus app chrome: title bar, sidebar, player bar, filter/search field
  state/      GPUI entities holding app state; owns all async orchestration
  music/      provider traits + models; spotify/ = librespot data access and playback (no GPUI)
  ui/         design system: theme, metrics, and reusable elements (gpui only)
  router/     Destination enum, navigation history, Link trait
  input/      text input element + global actions and keybindings
  i18n/       Fluent localization: the `t!` macro, locale selection, embedded .ftl
```

Dependency direction is strict; do not create a back edge:

```
sonora → views → state → music
         all ui-side crates → ui, router, input → ui → gpui
         every ui-side crate → i18n → gpui
```

- `music` holds the provider abstraction (`MusicApi`, `MusicProvider`, `Player`, `PlaybackFactory`)
  and the models in its root; each provider lives in a submodule (`music::spotify`,
  `music::youtube`, `music::local`). `state` and `views` see only the root traits and models — never
  a provider module. Only `sonora/src/main.rs` names a concrete provider.
- `ui` depends only on `gpui`, `serde` and `i18n`. It must never know about `music`, `state`, or
  playback.
- `music` must never depend on `gpui`. It is plain async Rust.
- `state` depends on `ui` only for `ThemeOverrides`, `MIN_FONT`, `MAX_FONT` (settings persistence).
- Widgets that need app state (player bar, sidebar) live in `views/src/chrome/`, not `ui`.
- `i18n` is a leaf: it depends on `fluent-bundle`, `unic-langid`, `sys-locale` and `gpui` (for
  `SharedString`) and on nothing else in the workspace.

## Building

### Any platform: what the build needs

The GPUI renderer is Vulkan-based, so a Vulkan ICD is a **runtime** requirement, not just a build
one. Link-time deps: `vulkan-loader`, `wayland`, `libxkbcommon`, `libxcb`, `libx11`, `libxcursor`,
`libxi`, `fontconfig`, `freetype`, `alsa-lib`, `dbus`, plus `pkg-config`.

`.cargo/config.toml` passes `-fuse-ld=mold` for `x86_64-unknown-linux-gnu`, so **mold must be on
PATH** for that target. If it isn't, either install mold or build with
`RUSTFLAGS="" cargo build …` to drop the flag.

### Nix (primary)

```sh
nix develop          # or: direnv allow  (.envrc runs `use flake`)
cargo run --locked --package sonora
nix run              # build + run the packaged binary
nix build            # ./result/bin/sonora
```

The devShell supplies `rustc`, `rustfmt`, `rust-analyzer`, `mold`, `pkg-config`, `sccache` and the
runtime libs via `LD_LIBRARY_PATH`. It does **not** ship `cargo` or `cargo-clippy` — those come from
the ambient system profile here. If `cargo` is missing inside the shell, that's why.

When `Cargo.lock` changes, `cargoHash` in `flake.nix` goes stale. Build once, take the `got:` hash
from the failure, and paste it in.

### Arch / CachyOS

```sh
sudo pacman -S --needed base-devel rust pkgconf alsa-lib dbus fontconfig freetype2 \
  libx11 libxcb libxcursor libxi libxkbcommon libxkbcommon-x11 wayland \
  vulkan-icd-loader mold
# plus a Vulkan driver: vulkan-radeon | vulkan-intel | nvidia-utils

cargo run --locked --package sonora
cargo build --release --locked --package sonora && ./target/release/sonora
```

### Debian/Ubuntu and Fedora

Not exercised in this repo; package sets translated from the dependency list above.

```sh
# Debian/Ubuntu
sudo apt install build-essential pkg-config mold libasound2-dev libfontconfig1-dev \
  libfreetype-dev libx11-dev libxcb1-dev libxcursor-dev libxi-dev \
  libxkbcommon-dev libxkbcommon-x11-dev libwayland-dev libvulkan-dev libdbus-1-dev \
  mesa-vulkan-drivers

# Fedora
sudo dnf install @development-tools pkgconf-pkg-config mold alsa-lib-devel fontconfig-devel \
  freetype-devel libX11-devel libxcb-devel libXcursor-devel libXi-devel \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nolight132/sonora](https://github.com/nolight132/sonora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
