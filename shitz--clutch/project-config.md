---
trigger: always_on
description: Clutch is a cross-platform desktop BitTorrent client written in pure Rust, built on the
---

# Agent Guidelines for Clutch

Clutch is a cross-platform desktop BitTorrent client written in pure Rust, built on the
[iced](https://github.com/iced-rs/iced) GUI framework and strictly following the **Elm
architecture**. Read `system_architecture.md` for the high-level design before making
non-trivial changes.

---

## Quality Gates

After every major change, all three commands **must** pass without warnings or errors:

```sh
cargo fmt
cargo check
cargo clippy -- -D warnings
cargo test
```

Run them in that order. Fix every warning before moving on — the CI pipeline treats
warnings as errors.

---

## License Header

Every `.rs` source file must start with the Apache 2.0 header **exactly** as shown:

```rust
// Copyright 2026 The clutch authors
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//   http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
```

Never create a new file without this header.

---

## Code Conventions

- Edition: **Rust 2024**. Use its idioms (e.g. `use<'a>` for precise capturing).
- Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for commit
  messages.
- Keep `update()` functions **non-blocking**: every network call, file I/O, and crypto
  operation must be dispatched via `iced::Task::perform()` or a background subscription.
  Blocking in `update()` is a bug.
- All RPC calls go through the single `mpsc` queue in `src/rpc/worker.rs`. Never issue an
  HTTP request directly from `update()` or `view()`.
- Represent illegal UI states as unrepresentable types — see the `Screen` enum in `app.rs`.
- Use `secrecy::SecretString` / `SecretBox` for all in-memory credentials. Never store raw
  passwords in plain `String`s.

---

## Theme & UI Elements (`crate::theme`)

All visual styling lives in the public `crate::theme` module, implemented by
`src/theme.rs` plus private `src/theme/*` helpers. Import only from `crate::theme`
— never hard-code colors, fonts, or sizes inline.

### When to use each helper

| Helper                              | Where to use                                                                                                                                                                              |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `theme::clutch_theme(is_dark)`      | Pass to `iced::application().theme()` in `main.rs`. Do **not** call elsewhere.                                                                                                            |
| `theme::icon(codepoint)`            | Render any Material Icons glyph (24 px). Use the named `ICON_*` constants, never raw codepoints inline.                                                                                   |
| `theme::icon_button(content)`       | Every icon-only toolbar button (pause, play, delete, settings, …). Gives a 36×36 circular hover target. Always attach `.on_press(msg)` — without it the button is visually disabled.      |
| `theme::m3_primary_button`          | The **primary CTA** of a screen or dialog (e.g. "Connect", "Save"). Solid brand-blue fill. Use only once per view context.                                                                |
| `theme::m3_tonal_button`            | **Secondary actions** that are important but not the primary CTA (e.g. "Test Connection", "Undo"). Lighter brand wash.                                                                    |
| `theme::segmented_control(...)`     | Mutually exclusive toggle groups (e.g. filter by status, sort direction). Pass `equal_width = true` when the control spans a fixed container; `compact = true` in space-constrained rows. |
| `theme::m3_card(theme)`             | Card-style containers: elevated surface with 16 px radius and a subtle drop shadow. Use inside the inspector panel and settings panels.                                                   |
| `theme::inspector_surface(theme)`   | The inspector panel's own background container. Rounded top corners, no bottom rounding — designed to sit flush against the bottom edge.                                                  |
| `theme::selected_row(theme)`        | The `container` wrapping a highlighted torrent row in the list. Apply only to the selected torrent; all other rows get no explicit style.                                                 |
| `theme::m3_tooltip(theme)`          | `iced_aw::Tooltip` containers. Dark contrasting surface, legible in both modes.                                                                                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shitz/clutch](https://github.com/shitz/clutch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
