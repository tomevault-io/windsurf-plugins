---
trigger: always_on
description: This file provides Codex-specific repository guidance.
---

# AGENTS.md

This file provides Codex-specific repository guidance.

For general repository rules, Rust workflow, testing expectations, and release hygiene, also follow [CLAUDE.md](/Users/penso/code/polyphony/CLAUDE.md).

## TUI And Ratatui

- Before changing [`crates/tui`](/Users/penso/code/polyphony/crates/tui), read the official ratatui tutorials at [ratatui.rs/tutorials](https://ratatui.rs/tutorials/).
- Use the widget and layout API docs at [docs.rs/ratatui/latest/ratatui](https://docs.rs/ratatui/latest/ratatui/).
- Prefer local source and examples from `/Users/penso/code/ratatui` when you need real patterns from the same project, especially:
  - `/Users/penso/code/ratatui/examples/apps/table/src/main.rs`
  - `/Users/penso/code/ratatui/examples/apps/demo/src/ui.rs`
  - `/Users/penso/code/ratatui/examples/apps/demo2/src/tabs/traceroute.rs`
  - `/Users/penso/code/ratatui/examples/apps/scrollbar/src/main.rs`
- For terminal UX inspiration, inspect `/Users/penso/code/lazygit`.
- Prefer stateful tables, scrollbars for long panes, sparklines for short history trends, and gauges or progress bars for cadence, retry, or completion state.
- Design for both wide and narrow terminals. Do not assume a large screen.
- Check the ratatui version pinned in the workspace `Cargo.toml` and `Cargo.lock` before copying APIs from the website or local checkout. The local `~/code/ratatui` clone may be ahead of the version this repo actually builds against.
- **Never block the TUI loop.** The draw → input → update cycle must stay instant. After handling a keypress, loop back to draw immediately — do not sleep, await network, or fall into a timed select. Network fetches and other async work belong in the orchestrator; the TUI only reads the latest snapshot. Any operation taking more than 100ms must show a loading indicator (e.g. braille spinner) so the user knows something is happening.

### Listing tables

- Every tab is a full-height listing table with a scrollbar when content exceeds the visible area.
- Every listing row must support opening a detail modal (Enter key) that shows expanded information relevant to that row.
- **Column styling:** ID, time, and other secondary/metadata columns use the `theme.muted` color. Primary content (title, name) uses `theme.foreground`.
- **Emoji indicators:** Prefer emoji/Unicode symbols over text for columns that represent an enum with ≤5 values (status, decision, kind). This keeps columns compact. Examples: `●` open, `✓` done, `✕` failed, `◷` waiting, `⊘` cancelled.
- **No highlight arrow symbol.** Row selection uses background highlight only (`row_highlight_style`), not a `▸` prefix — the arrow duplicates the background highlight.
- **First-column activity indicator:** The first column of listing rows can show the item's activity state: a braille spinner (`⠋⠙⠹…`) when actively running, a dot (`●`) for static state like "has workspace", or blank.
- **Footer:** Show selection position (e.g. "3 of 12") and any relevant counts (e.g. retrying agents) in the table block's bottom border.
- **Sorting:** Support sort toggling (e.g. `s` key) where it makes sense. Show the current sort label in the footer.

## Website and Social Image

- The OpenGraph image (`website/assets/og-image.svg` → `og-image.png`) shows the TUI dashboard as a backdrop behind the Polyphony branding. It should visually match what a user actually sees in the product.
- When the TUI layout, columns, or tab structure changes, update the SVG to reflect the new UI. Render the PNG via Playwright (see `og-render.html` pattern) — ImageMagick does not render the SVG faithfully.
- Bump the `?v=` cache-busting parameter on the image URLs in `website/index.html` after regenerating the PNG.

## Type System Conventions

- Prefer enums over `String` for fields with a fixed set of values. Enums catch invalid values at compile time (or deserialization time), enable exhaustive `match`, and eliminate manual string validation.
- Derive `Copy` on fieldless enums (`AgentTransport`, `FeedbackChannelKind`, `AgentEventKind`, etc.) so they can be passed by value without `.clone()`.
- Use `#[serde(rename_all = "snake_case")]` on enums whose serialized form must be lowercase/snake_case.
- **Config crate limitation:** `AgentProfileConfig` and `FeedbackConfig` are deserialized through the `config` crate, which does not honor serde `rename_all` on enum variants. Fields in these structs that accept enum-like values must stay as `String` with manual parsing (see `infer_agent_transport()` and the `feedback.offered` validation in `ServiceConfig::validate()`). Only use typed enums in structs deserialized directly by serde (serde_yaml, serde_json).
- When replacing a `String` field with an enum, update all construction sites, match arms, format strings (use `{:?}` for Debug output of enums where the old code printed the string), and test assertions.

## Symphony References

- For orchestration architecture and workflow-contract reference, inspect the upstream Symphony project at [github.com/openai/symphony](https://github.com/openai/symphony).
- Read the Symphony service specification in [SPEC.md](https://github.com/openai/symphony/blob/main/SPEC.md) and the local checkout at `/Users/penso/code/symphony/SPEC.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [penso/polyphony](https://github.com/penso/polyphony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
