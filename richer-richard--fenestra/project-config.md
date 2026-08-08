---
trigger: always_on
description: fenestra is designed so that an agent authoring a UI can *see and verify*
---

# Building fenestra UIs as an agent

fenestra is designed so that an agent authoring a UI can *see and verify*
its work without a display server. This file is the working manual.

## The loop

1. Write a view (or a full `App`).
2. Drive it with the harness — semantic queries, not coordinates — and
   assert structure and messages.
3. Render it headlessly to a PNG. **Open and look at the PNG.** Layout
   bugs, clipped text, and bad spacing are visible; do not skip this.
4. Iterate until it looks right, then lock it with a golden test.

```rust
use fenestra::prelude::*;
use fenestra::shell::render_element;

let view: Element<()> = col().p(SP6).gap(SP4).children([
    text("Hello").size(TextSize::Xl2).weight(Weight::Semibold),
    button("Save").into(),
]);
let image = render_element(view, &Theme::light(), (480, 240));
image.save("preview.png").unwrap(); // now actually look at it
```

Headless rendering is deterministic: embedded Inter fonts, scale 1.0,
reduced motion, in-memory clipboard. The same tree renders the same pixels
on every machine of the same GPU class (cross-rasterizer runs use a small
tolerance; see Golden tests).

## Driving a real app: the harness

`Harness` runs the full Elm loop — dispatch, state, focus, editing —
driven by semantic queries (find things the way a user would; never
hardcode coordinates):

```rust
use fenestra::prelude::*;
use fenestra::shell::Harness;

let mut h = Harness::new(MyApp::default(), Theme::light(), (800, 600));
h.click(&by::role(Semantics::Button).name("Add"));   // strict: 0 or 2+ matches panic
h.type_text("hello");                                 // into the focused input
h.key(KeyInput::plain(Key::Enter));

assert_eq!(h.app().items.len(), 1);                   // state
assert!(h.query(&by::label("hello")).is_some());      // structure (None = absent)
let msgs = h.take_messages();                          // behavior: what the UI emitted
h.render().save("after.png").unwrap();                 // pixels — now look at it
```

Verbs: `click right_click double_click triple_click shift_click hover
type_text key tab shift_tab focus drag drop_file wheel`; `pump(ms)` advances the
deterministic clock; `activate_window(key)` / `render_window(key)` for
multi-window apps. Failed lookups print the whole accessibility tree —
read it, it names every role and label on screen.

Two inspector dumps when lost: `h.frame().debug_tree()` (layout rects,
flags, `src=file:line` builder provenance) and
`h.frame().access_yaml()` (Playwright aria-snapshot grammar).

### JSON scenarios (no recompile)

For quick probes, drive any app from JSON instead of Rust:

```json
{"steps": [
  {"click": {"role": "button", "name": "Add"}},
  {"type": "hello"},
  {"assert": {"exists": {"label": "hello"}}},
  {"shot": "after-add"}
]}
```

`fenestra::shell::run_scenario(&mut harness, json, shots_dir)` — typos
and missing targets are loud errors carrying the step index and the
accessibility tree.

`render_app(&mut app, &[SyntheticEvent::...], size, &theme)` remains
for coordinate-level pixel probes.

## Asserting structure (accessibility tree)

Every widget exposes role, state, name, and value. Assert on it instead of
pixels when you care about structure:

```rust
use fenestra::prelude::*;

let frame = build_frame(&view, &theme, &mut fonts, &mut state, (800.0, 600.0), 1.0);
let tree = frame.access_tree();
// Walk AccessNode { id, semantics, label, value, rect, focusable, children }
// e.g. find Some(Semantics::Button) with label == Some("Save").
```

The same tree feeds real assistive technology (AccessKit) in windowed
runs, so labeling your widgets is both testable and genuinely accessible.
Icon-only buttons need `.label("...")`.

## Golden tests

```rust
use fenestra_shell::testing::assert_png_snapshot;

assert_png_snapshot(snapshot_dir(), "my_widget", &image);
```

- `FENESTRA_UPDATE_SNAPSHOTS=1 cargo test` writes/updates goldens.
- Failures write `<name>.actual.png` next to the golden — look at both.
- Tolerance: 3/255 per channel, 0.2% of pixels; macOS/Metal is the
  reference platform. Other rasterizers (CI software adapters) set
  `FENESTRA_SNAPSHOT_BUDGET=0.006`.
- Failures write `<name>.actual.png`, `<name>.diff.png` (offending
  pixels in red — look here first), and `<name>.side.png`
  (golden | actual | diff) next to the golden.

## Vocabulary cheat sheet

Constructors: `div() row() col() stack() text(s) spacer() divider()
path(bez, viewbox, stroke) image_rgba8(w, h, px) raw_input(v, ph)
raw_text_area(v, ph) rich_text([span(s).weight(..).color(..)
.size_px(..).family(..).italic(), ..])`

Layout: `.p/.px/.py/.pt/.pr/.pb/.pl(f32)` padding, `.m*` margins,
`.gap(f32)`, `.w/.h/.min_w/.max_w/.min_h/.max_h(Length)`, `.w_full()
.h_full() .grow() .shrink0() .wrap()`, `.items_start/center/end/baseline()`,
`.justify_start/center/end/between()`, `.absolute() .top/.right/.bottom/
.left(f32)`, `.grid_cols/.grid_rows(tracks) .grid_col/.grid_row(start, span)`,
`.overflow_hidden() .scroll_y() .stick_to_bottom()`

Style: `.bg(paint) .border(w, color) .rounded(r) .rounded_full()
.shadow(ShadowToken) .opacity(f32)`; text: `.size(TextSize) .weight(Weight)
.color(c) .mono() .truncate() .text_align(..)`

Interaction: `.on_click(msg) .on_right_click(msg) .on_double_click(msg)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richer-richard/fenestra](https://github.com/richer-richard/fenestra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
