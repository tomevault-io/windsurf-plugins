---
trigger: always_on
description: Guidance for agents working in this repository.
---

# AGENTS.md

Guidance for agents working in this repository.

## Project Shape

Super Platinum is a Rust desktop Slack client built with Dioxus Desktop.

Important boundaries:

- `crates/super-platinum-core/` owns renderer-neutral domain code: cache, config/session,
  Slack API/realtime/models, workspace state helpers, palette ranking, commands,
  reducers, supervisors, and the stable agent protocol.
- `src/desktop/main.rs` is the Dioxus application entry point.
- `src/desktop/state/` owns the serial shell state and mutation boundary
  (projections, selection, timeline window, composer, fixtures).
- `src/desktop/bootstrap/`, `messaging.rs`, and `realtime.rs` own native async
  Slack work (session, history, discovery, persistence).
- `src/desktop/view/`, `overlays.rs`, and `src/desktop/styles/` own the typed
  DOM UI and CSS cascade modules.
- `src/desktop/media.rs` owns the opaque `super-platinum-media://` protocol.
- `src/desktop/agent.rs` is the optional live control plane (`SUPER_PLATINUM_AGENT=1`).
- `src/desktop/auth.rs` owns the Slack sign-in WebView flow (tao/wry).

Keep changes inside the smallest boundary that matches the task. Domain behavior
belongs in `super-platinum-core`; DOM focus, selection, scrolling, capture, and other
renderer-owned behavior belongs in `src/desktop/`. Keep implementation modules
focused: aim for 300–700 lines and split before 1,000 lines when cohesive.

## Dioxus Documentation Rule

Do not guess Dioxus APIs from memory. This project uses an exact pinned git
revision rather than a crates.io range.

For application setup, components, signals, hooks, document evaluation, desktop
configuration, custom protocols, or runtime behavior, check the Dioxus 0.7 docs
first:

<https://dioxuslabs.com/learn/0.7/>

When documentation disagrees with the pinned revision in `Cargo.toml`, the
repository and compiler win. Prefer small compile-backed changes.

## Development Commands

Use locked Cargo commands by default:

```sh
cargo fmt --check
cargo check --locked
cargo test --locked
```

For most Rust changes, run `cargo fmt --check` and `cargo test --locked` before
calling the work done. Also run the independently locked core suite when domain
behavior changes. Use `cargo check --locked` for faster iteration.

If a build fails with stale dependency artifacts under `target/debug/deps`, a clean rebuild has fixed that class of local issue before:

```sh
cargo clean
cargo build --locked
```

Do not treat local environment noise, such as shell startup warnings, as the root cause of Rust or app failures without evidence.

## Persistence And Secrets

Be careful around `crates/super-platinum-core/src/config/`.

- The app stores Slack session secrets through the configured secret backend.
- `STORAGE_QUALIFIER` and `KEYRING_SERVICE` still say `snack` after the Super
  Platinum rebrand. That is deliberate: renaming either one orphans the existing
  config directory, warm cache, and Keychain item, signing every user out. Change
  them only together with a migration.
- Tests should not touch the real macOS Keychain or platform keyring.
- Keep test-only secret isolation behind `cfg(test)`.
- When changing session format, preserve migration behavior and add round-trip tests for both current and legacy shapes.

The app should not introduce repeated keychain prompts on boot or during tests.

## Performance Expectations

This is intended to feel fast in dev and release builds.

- Do not add synchronous disk or network work to Dioxus render/event paths.
- Prefer async tasks or background work for cache writes and Slack calls.
- Keep rendered message lists bounded or lazily computed where possible.
- Be careful with supervisors and periodic ticks; avoid always-on work unless needed.
- Preserve `[profile.dev]` settings unless there is a measured reason to change
  them.

## UI Expectations

Super Platinum should feel like a focused desktop Slack client, not a marketing page.

The visual source of truth is `docs/design-system.md`. Read it before changing UI
styles or adding a component, and reuse the tokens and interaction states defined
in `src/desktop/styles/base.css` and `design-system.css`.

- Keep the UI quiet, dense, and readable.
- Use the existing CSS custom properties and appearance helpers.
- Prefer existing Dioxus components over one-off presentation logic.
- Keep controls stable in size; avoid layout shifts on hover, loading, or text changes.
- Do not add decorative chrome that competes with channels, messages, threads, and search.

### Icons — Hard Rule

**Any and all interface icons must come from Google Material Symbols Rounded.**
Do not use text glyphs or emoji as interface icons, hand-drawn SVG paths, another
icon family, or platform-specific symbols. Render icons through the typed,
zero-dependency helper in `src/desktop/icons.rs`, which keeps the official rounded
24 px SVG paths inline, offline, and `currentColor`-aware. When a needed symbol is
missing, add its official `materialsymbolsrounded` 24 px path from Google's
`material-design-icons` repository to that module and reuse it from there. Brand
marks, user-authored emoji, workspace custom emoji, avatars, and message content
are content rather than interface icons and are the only exceptions.

### Surfaces


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [3kh0/super-platinum](https://github.com/3kh0/super-platinum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
