---
trigger: always_on
description: These instructions apply to all work in this repository. They summarize build/test commands, style, and repo workflows for agentic coders. Base assumptions come from `CLAUDE.md` and project conventions.
---

# AGENTS GUIDE

These instructions apply to all work in this repository. They summarize build/test commands, style, and repo workflows for agentic coders. Base assumptions come from `CLAUDE.md` and project conventions.

## Build, Lint, Test
- Preferred dev command: `just dev` (builds release, installs to `~/.local/bin/meeting`, restarts COSMIC panel).
- Standard builds: `just build-release` (default), `just build-debug` for debug profile.
- Run app quickly: `just run` (release profile with `RUST_BACKTRACE=full`).
- Lint: `just check` (runs `cargo clippy --all-features -W clippy::pedantic`). Use `just check -- --fix` only if fixing lint issues intentionally.
- JSON lint output: `just check-json`.
- Clean: `just clean` or `just clean-dist` (also clears vendored deps).
- Packaging helpers: `just build-vendored`, `just build-flatpak`, `just install-flatpak`, `just build-deb`, `just vendor`, `just update-flatpak-sources`.
- Reload panel manually if needed: `just reload-applet` (restarts `cosmic-panel`).

## Running Tests
- There are currently no Rust tests; add focused tests near touched code when useful.
- Run full suite: `cargo test` (respect `CARGO_TARGET_DIR` if set).
- Run single test or module: `cargo test <pattern>` (e.g., `cargo test meeting_url`); list with `cargo test -- --list`.
- For doctests in a single file: `cargo test --doc --lib -- <pattern>`.
- Use `CARGO_TARGET_DIR=target` unless a different dir is already configured.

## Tooling and Versions
- Rust edition 2024; rely on standard `rustfmt` defaults unless file shows a different style.
- No repo `rustfmt.toml`; use default formatter behavior.
- No Cursor or Copilot rules found (no `.cursor/rules`, `.cursorrules`, or `.github/copilot-instructions.md`).
- `libcosmic` is a git dependency; keep `Cargo.lock` unchanged unless intentionally updating deps.
- Use `tokio` with full features; prefer async patterns already in code (background tasks, subscriptions).

## Branching and Workflow
- Do not work directly on `main`. Switch to `dev` or a feature branch before changes. If on `main`, stash and move.
- Before commits: run `cargo fmt` and `just check` at minimum. For functional changes, use `just dev` so the installed binary reflects updates.
- Keep commits atomic and scoped. Do not push unless user requests.
- Releases are tagged `vX.Y.Z`; tags without `v` do not trigger release workflow.

## Platform Compatibility
- Target: COSMIC desktop across distros. Assume systemd for best experience, but provide graceful fallbacks for non-systemd (e.g., if `org.freedesktop.login1` unavailable, avoid panic and skip wake/unlock hooks).
- D-Bus services used (declared in Flatpak manifest): `org.gnome.evolution.dataserver.Calendar8`, `org.gnome.evolution.dataserver.Sources5`, `org.gnome.OnlineAccounts`, `org.freedesktop.login1`. Always check availability and degrade gracefully.
- Use XDG base dirs; never hardcode `~/.config` paths beyond allowed ones (current access to `~/.config/cosmic` and `~/.config/evolution`).

## Flatpak Notes
- Flatpak sandbox limits D-Bus and filesystem. Any new D-Bus access must be added to `com.dangrover.next-meeting-app.json` using `--talk-name` / `--system-talk-name`.
- Filesystem access is restricted; favor existing allowed dirs. Provide fallbacks if feature cannot run inside sandbox.
- Test both native and Flatpak flows; when a capability is missing in Flatpak, fail soft with user-facing messaging or no-op.

## COSMIC UI Patterns
- Follow COSMIC applet layout: outer column with `.padding([8, 0])`; use `cosmic::applet::menu_button()` for clickable rows, `cosmic::applet::padded_control()` for headings/non-interactive areas, and `cosmic::applet::padded_control(widget::divider::horizontal::default()).padding([space_xxs, space_s])` for dividers.
- Settings lists should use `widget::list_column()` with dividers matching peers.
- Ensure actions like "Join" buttons align with existing spacing and icon usage in `widgets` module.

## Localization
- Localization via Fluent in `i18n/<lang>/meeting.ftl`. Use `fl!("message-id")` macros for strings; avoid hardcoded text.
- When adding strings, update `i18n/en/meeting.ftl` and mirror to other languages as needed. Keep placeholders and attributes consistent.
- Initialize localization using `i18n::init(&requested_languages)` pattern; do not bypass requester.

## Imports and Formatting
- Run `cargo fmt` after changes. Keep `use` statements grouped by crate: std, external, crate modules, then self modules; avoid unused imports.
- Prefer fully qualified module paths in code over glob imports unless existing file uses globs intentionally.
- Order dependencies consistently; prefer alphabetical within groups.
- Keep lines readable; follow rustfmt on max width; avoid manual alignment that rustfmt will undo.

## Types and Naming
- Use descriptive names; avoid single-letter locals except common indices. Favor `meeting`, `config_ctx`, `runtime`, etc.
- Use `const` for shared literals (e.g., app IDs, URL schemes).
- Prefer `&str` over `String` when borrowing suffices; clone only when required.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dangrover/cosmic-ext-applet-next-meeting](https://github.com/dangrover/cosmic-ext-applet-next-meeting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
