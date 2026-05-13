---
trigger: always_on
description: Terminal UI for the Claude status page (Statuspage v2 API). Single binary,
---

# partly-claudy — agent guide

Terminal UI for the Claude status page (Statuspage v2 API). Single binary,
async tokio + ratatui 0.30. Read this before making non-trivial changes.

## Architecture

```
src/main.rs     CLI parsing (clap) → ratatui::init() → run() → ratatui::restore()
src/api.rs      Statuspage v2 DTOs + Source { Live, Fixture } async fetch_summary()
src/bars.rs     compute(components, incidents, today) → Vec<UptimeRow> for 90-day bars
src/events.rs   EventLoop spawns 3 tokio tasks (input / render tick / refresh)
                merges into mpsc<AppEvent> { Key, Tick, Resize, Loaded, LoadFailed }
src/app.rs      App state + handle(AppEvent) reducer (no I/O), Pane { Services, Incidents }
src/theme.rs    AppTheme — pinned + embedded + opaline + ~/.taho/themes/ discovery
src/ui/mod.rs   render(frame, &mut app) — header / banner / body / footer + modal
src/ui/*.rs     header, banner, services (dot+name+bars+pct+meta per row),
                timeline (Incidents), modal, footer, help, theme_picker, scroll, skeleton
themes/*.toml   embedded Opaline themes (TAHO + Claude Code variants)
```

The reducer in [src/app.rs](src/app.rs) is pure: no I/O, no async. All
network/disk work is in [src/events.rs](src/events.rs), which posts results
back through the channel as `AppEvent::Loaded(Box<Summary>)` or
`AppEvent::LoadFailed(String)`. UI state lives entirely on `App`.

## Key crates

- **ratatui 0.30** — `ratatui::init()` / `ratatui::restore()` for terminal
  lifecycle. `Layout::vertical(...)` / `Layout::horizontal(...)` for splits.
  `.right_aligned()` / `.left_aligned()` on `Paragraph` (do not use the
  renamed `HorizontalAlignment` directly).
- **tui-overlay 0.1** — `Overlay::new().anchor(Anchor::Center).slide(Slide::Top)`
  for the detail modal, with an `OverlayState` on `App`. Render order:
  main UI first, then `frame.render_stateful_widget(overlay, area, &mut state)`,
  then read `state.inner_area()` and render the modal body into it. Tick
  the state every frame from `App::handle(AppEvent::Tick)`. Modal opens
  on Enter, closes on Esc.
- **tui-skeleton 0.3** — used by [src/ui/skeleton.rs](src/ui/skeleton.rs).
  `SkeletonBlock` for solid + braille panels (uptime bar uses
  `.braille(true)`). `SkeletonStreamingText` for incident bullet rows
  (typewriter fill, `repeat(true)` keeps it cycling). `elapsed_ms`
  comes from `App::loading_elapsed_ms()` which rewinds to
  `refresh_started_at` on manual refresh — stream replays from frame 0.
- **opaline 0.4** — token-based theme engine. Themes in
  [themes/](themes/) are `include_str!`'d at compile time. Discovery
  walks `~/.taho/themes/` so user themes are shared with taho-admin.
  Selection persists to `~/.taho/partly-claudy/settings.toml`. Tokens
  resolved via `AppTheme` (`bg`, `panel_bg`, `text`, `muted`, `dim`,
  `accent`, `success`, `warning`, `danger`, `info`). **Do not** hardcode
  `Color::Red`/`Color::Green` — go through `AppTheme`.

## Adding a new pane / widget

1. Add a file under `src/ui/`. Pattern: `pub fn render(frame, area, app)`.
2. If the widget needs focus, add a variant to `Pane` in
   [src/app.rs](src/app.rs) and include it in `cycle_focus`'s order.
3. Use [`services::pane_block`](src/ui/services.rs) for the chrome —
   it bundles the rounded border, focus-dependent border color, and
   the `pane_title` leader. Add right-aligned context with
   `.title_top(Line::from(...).right_aligned())`.
4. Borrow colors from `AppTheme` — never hardcode RGB.
5. If the widget shows live data, render a bespoke skeleton when
   `app.is_loading()`. See `skeleton::render_services` /
   `render_incidents` — generic shimmer is anti-pattern; reproduce
   the loaded shape so the layout doesn't reflow on first data.
6. For scroll affordance, use [`scroll::overlay`](src/ui/scroll.rs)
   after `render_stateful_widget` — read `ListState::offset()` to
   detect items above/below the visible window. Reserve a 1-cell
   gutter on the right so indicators don't clobber content.

## Adding a new event source

All async work happens in [src/events.rs](src/events.rs). Spawn another
tokio task in `EventLoop::new`, add a variant to `AppEvent`, and handle
it in `App::handle`. Keep the reducer pure.

## Refresh & loading state

- `r` key in main view fires `events.refresh_now()` and calls
  `app.begin_refresh()` (which sets `refreshing = true`,
  stamps `refresh_started_at`, toasts "refreshing..."). Gated on
  `!app.modal_open()` so it doesn't fire under help/theme picker/modal.
- `Loaded` and `LoadFailed` clear `refreshing`. On `LoadFailed` the prior
  data stays visible — `refreshing` flips false but `summary` is preserved.
- `is_loading()` returns true during initial load (no summary) **and**
  during in-flight refresh — skeleton replays in both states.
- The Services pane height is sized via `services::SKELETON_ROWS` (= 6,
  the typical Statuspage tenant) when `is_loading()`, so the skeleton has
  vertical room before the first fetch returns.

## Statuspage API

See [docs/statuspage-api.md](docs/statuspage-api.md) for the API surface,
uptime math, and fetch strategy.

## Testing

CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)) runs the same
gate locally: `cargo fmt --check` → `cargo clippy --all-targets --

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taho-inc/partly-claudy](https://github.com/taho-inc/partly-claudy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
