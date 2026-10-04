---
trigger: always_on
description: Guidance for anyone, human or coding agent, working on Roshan. Read this
---

# AGENTS.md

Guidance for anyone, human or coding agent, working on Roshan. Read this
before changing code. User-facing docs are in [README.md](README.md),
build details in [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md), and
contribution rules in [CONTRIBUTING.md](CONTRIBUTING.md).

## What Roshan is

A small, private desktop utility (Rust + GPUI). Users save **sessions**
("Morning", "Deep work"…): ordered lists of real apps, shell commands and
links. Pressing **Run** launches them one by one, with an optional pause after
each item. Roshan opens at sign-in (can be turned off) and waits idle, so the
computer boots light and the user's setup starts when they decide.

It is **not** a startup manager, process manager, optimizer, automation
platform or tracker. Reject features that push it in those directions.

## Non-negotiable rules

- **Real data only.** Never add demo entries, a built-in app catalog,
  hardcoded app names, or bundled third-party icons. App names and icons come
  from the OS at runtime. Test fixtures stay inside `#[cfg(test)]`.
- **Private.** No accounts, telemetry or analytics. The update check in
  `roshan-platform/src/update.rs` is the **only** network access: GitHub's
  latest-release API plus, on Windows, the release's `Roshan-Setup-*.exe`
  and `SHA256SUMS.txt`. No user data is sent. Do not add other requests.
- **Honest status.** Report only what the OS told us: `Launched`, `Failed`
  (with the OS reason) or `Skipped`. Never claim an app is "running" or
  "ready".
- **Light.** No background services, no filesystem scanning while idle.
  Discovery runs only when the app picker opens; icons load lazily and are
  cached on disk.
- **Never write to a broken config.** If `roshan.toml` cannot be parsed, it is
  moved aside, never overwritten.
- **Commits.** Use only the repository owner's git identity. Do not add
  AI co-author trailers or "generated with" lines to commits or PRs.

## Layout

```text
crates/
  roshan-core/        Model, TOML config, launch engine. No OS or UI code.
    model.rs          Session, LaunchItem, ItemKind, AppTarget
    config.rs         load/save (atomic), schema, normalization
    engine.rs         sequential runner, RunEvent, CancelToken, Launcher trait
    version.rs        release versions and their ordering
  roshan-platform/    OS integration behind one facade (lib.rs).
    windows.rs        AppsFolder discovery, IShellItemImageFactory icons,
                      ShellExecuteEx launch, registry start-at-login,
                      single-instance mutex
    macos.rs          .app scan, NSWorkspace icons, `open`, LaunchAgent
    linux.rs          XDG .desktop discovery, icon themes, Exec launch,
                      terminal detection, XDG autostart
    desktop_entry.rs  pure .desktop / Exec= parsing (tested on all OSes)
    unix.rs           shared macOS/Linux helpers
    icon_cache.rs     on-disk PNG/SVG icon cache
    open_target.rs    URL/path classification, allowed URL schemes
    update.rs         GitHub release check, verified download, runs the
                      Windows installer silently to update
  roshan/             The GPUI app.
    main.rs           args (--run, --startup, --updated, --version), window setup
    app.rs            all state and behavior (Roshan struct)
    view.rs           root view, title bar, overlays host
    screens/          welcome, sessions list, session, settings, updates
    overlays/         add picker, item editor, dialogs, sheet helpers
    ui.rs             shared widgets (buttons, segmented, tiles, logo)
    theme.rs          palette (light/dark) and component theme sync
    i18n.rs           languages, string lookup, RTL helpers
    locales/          en.toml, fa.toml
    assets/           icons (Lucide subset), fonts (Vazirmatn), logo
    build.rs          logo sizes + Windows icon/version resource
vendor/gpui-pre-windows/  Patched GPUI Windows backend (see ROSHAN_PATCH.md)
docs/                 DEVELOPMENT.md, media/ (posters and their HTML sources)
packaging/windows/    Inno Setup installer (roshan.iss) + build-installer.ps1
packaging/linux/      .desktop file
```

Keep the dependency direction: `roshan` → `roshan-platform` → `roshan-core`.
OS code belongs only in `roshan-platform`, behind `#[cfg]` modules with the
same function set per OS.

## Commands

```sh
cargo build                     # debug build
cargo run                       # run the app
cargo test                      # all unit tests (Windows: 30+)
cargo fmt --all
cargo clippy --all-targets -- -D warnings
cargo run -p roshan-platform --example list_apps [filter]   # see discovery
```

Before finishing any change: `fmt`, `clippy -D warnings` and `test` must pass.
The macOS and Linux platform code can be type-checked from Windows:

```sh
rustup target add aarch64-apple-darwin x86_64-unknown-linux-gnu
cargo clippy -p roshan-core -p roshan-platform --target aarch64-apple-darwin --all-targets -- -D warnings
cargo clippy -p roshan-core -p roshan-platform --target x86_64-unknown-linux-gnu --all-targets -- -D warnings
```

Cross-checking needs a C cross compiler for the target, because the HTTPS
stack (`ring`) builds C code. Without one, rely on CI: the Linux job compiles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sajjadmrx/roshan](https://github.com/sajjadmrx/roshan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
