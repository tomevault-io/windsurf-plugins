---
trigger: always_on
description: validates every line through `recordings::parse_lines` before writing so a bad
---

# AGENTS.md

Guidance for AI assistants (and humans) working in the LazyMQTT codebase.

## What this is

LazyMQTT is a terminal-UI MQTT client written in Rust, inspired by MQTT Explorer.
It lets a user save broker connections, store per-connection topic subscriptions,
watch incoming messages in a live collapsible topic tree, inspect and publish
messages, clear retained messages, and copy payload text to the clipboard. A
small in-process plugin system can observe the message stream and annotate or
re-render it. The design goal is speed: an async MQTT task feeds a non-blocking
render loop.

## Working agreement (read first)

Two rules apply to every change, no exceptions:

1. **Branch per feature.** Never commit a new feature directly to `master`.
   Start each feature (or non-trivial change) on its own branch off `master`
   (`git checkout -b <name>`), and merge via pull request. `master` stays
   releasable at all times.
2. **Leave the code cleaner than you found it.** The codebase is growing, so
   pay down technical debt as you go: when a change touches messy, duplicated,
   or awkward code, refactor it in the same branch — extract shared helpers,
   remove duplication, keep modules small and focused. Don't just bolt new
   code on. (Example: the JSON→styled-span colorizer was extracted into
   `plugin/builtin/jsonfmt.rs` so json-view and protobuf-view share it.) Keep
   refactors scoped to what's relevant so the diff stays reviewable, and keep
   `fmt`/`clippy`/`test` green.

## Build & run

```bash
cargo run                  # debug
cargo build --release      # optimized (LTO, stripped)
cargo test                 # unit tests
cargo clippy               # lint before committing
cargo fmt                  # format (rustfmt defaults)
```

There is a small `#[cfg(test)]` suite (selection/yank math, plugin dispatch,
annotations, JSON view). Add tests in the relevant module when you add logic
worth testing (tree building, topic parsing, config round-trips, plugin
behavior). Two long-standing clippy warnings are pre-existing (a derivable
`PublishBuffer::Default` and a needless borrow in `ui.rs`); don't count them as
new. Keep the tree warning-free otherwise.

## Architecture

The app is a single-threaded UI loop plus a set of async MQTT tasks that
communicate over channels. Keep that boundary clean.

```
main.rs      Terminal setup + the render/input loop. Owns the tokio runtime.
app/         The App state object + its behavior, one submodule per concern.
  mod.rs       App struct, App::new, shared free helpers, re-exports, tests.
  screen.rs    Screen/Command enums + the command-menu registry.
  view.rs      Focus/PaneFold/DetailKind + the DetailLine render model.
  forms.rs     FormBuffer/PublishBuffer/AlertForm buffers + Status.
  connection.rs  connect/disconnect, send, push_message, plugin dispatch.
  commands.rs  the command registry entry point + `m`-menu building.
  broker.rs    topic selection, payload/history line building, text selection.
  alerts.rs / schemas.rs / recordings.rs / theme.rs  each screen's App-side logic.
  textarea.rs  reusable multi-line text buffer + cursor (recording & schema editors).
config.rs    Connection + Subscription structs; JSON persistence to disk.
paths.rs     Config dir (~/.config/lazymqtt, XDG on every OS) + one-time
             migration from the old macOS Application Support location.
theme.rs     Theme (color specs) + Palette (resolved Colors) + presets +
             theme.json persistence. See the theming note below.
mqtt.rs      Async client task. Message, MqttEvent, MqttCommand, MqttHandle.
tree.rs      TopicTree: aggregates messages into a hierarchy split on '/'.
ui/          ratatui rendering, one module per screen; never mutates state.
  mod.rs       `draw` dispatcher (Screen -> screen module).
  common.rs    shared widgets: title_block, pane_title, scrollbar, center_rect.
  <screen>.rs  broker, connections, publish, alerts, recordings, theme,
               plugins, menu, help, statusbar.
events/      keyboard/paste handling, one module per screen; mutates App.
  mod.rs       handle_key/handle_paste dispatchers + strip_newlines.
  <screen>.rs  broker, connections, publish, alerts, recordings, theme,
               plugins, menu.
plugin/      In-process plugin API + host + built-in plugins.
  mod.rs       Plugin trait, PluginHost (dispatch, enable/disable, inspect).
  api.rs       PluginEvent / PluginAction / Annotation / Inspector* types.
  config.rs    per-plugin enable/disable, persisted under plugins/.
  builtin/     bundled plugins (json-marker, json-view, xml-view,
               protobuf-view, topic-alerts, json-schema,
               publish-templates, payload-generator, traffic-analytics,
               topic-recorder) + jsonfmt (shared JSON colorizer).
  topics.rs    shared MQTT topic-filter matching (`+`/`#`).
  schemas.rs   per-connection topic→schema mappings + subset validator.
  templates.rs global publish presets (topic/payload/QoS/retain).
  generators.rs global payload generators (counter/random/timestamp).
```

`App`'s methods are split across `app/*.rs` as separate `impl App` blocks; the
state types live in `app/{screen,view,forms}.rs` and are re-exported from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScottFelder/lazymqtt](https://github.com/ScottFelder/lazymqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
