---
trigger: always_on
description: **Always search for existing usage patterns in the NEW crates (widgets, code_editor, studio) before making syntax changes.** The old `widgets` and `live_design!` syntax is deprecated. When unsure about the correct syntax for something, grep for similar usage in `widgets/src/` to find the correct pattern.
---


# Makepad Project Guide

## Important: When Converting Syntax

**Always search for existing usage patterns in the NEW crates (widgets, code_editor, studio) before making syntax changes.** The old `widgets` and `live_design!` syntax is deprecated. When unsure about the correct syntax for something, grep for similar usage in `widgets/src/` to find the correct pattern.

```bash
# Example: find how texture declarations work in new system
grep -r "texture_2d" widgets/src/
```

**Critical: Always use `Name: value` syntax, never `Name = value`.** The old `Key = Value` syntax no longer works. For named widget instances, use `name := Type{...}` syntax.

## Running UI Programs

```bash
RUST_BACKTRACE=1 cargo run -p makepad-example-splash --release & PID=$!; sleep 15; kill $PID 2>/dev/null; echo "Process $PID killed"
```

## Cargo.toml Setup

```toml
[package]
name = "makepad-example-myapp"
version = "0.1.0"
edition = "2021"

[dependencies]
makepad-widgets = { path = "../../widgets" }
```


## Widgets DSL (script_mod!)

The new DSL uses `script_mod!` macro with runtime script evaluation instead of the old `live_design!` compile-time macros.

### Imports and App Setup

```rust
use makepad_widgets::*;

app_main!(App);

script_mod!{
    use mod.prelude.widgets.*
    
    load_all_resources() do #(App::script_component(vm)){
        ui: Root{
            main_window := Window{
                window.inner_size: vec2(800, 600)
                body +: {
                    // UI content here
                }
            }
        }
    }
}

impl App {
    fn run(vm: &mut ScriptVm) -> Self {
        crate::makepad_widgets::script_mod(vm);  // Register all widgets
        // Platform-specific initialization goes here (e.g., vm.cx().start_stdin_service() for macos)
        App::from_script_mod(vm, self::script_mod)
    }
}

#[derive(Script, ScriptHook)]
pub struct App {
    #[live] ui: WidgetRef,
}

impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions: &Actions) {
        // Handle widget actions
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        self.match_event(cx, event);
        self.ui.handle_event(cx, event, &mut Scope::empty());
    }
}
```

### Available Widgets (widgets/src/lib.rs)

Core: `View`, `SolidView`, `RoundedView`, `ScrollXView`, `ScrollYView`, `ScrollXYView`
Text: `Label`, `H1`, `H2`, `H3`, `LinkLabel`, `TextInput`
Buttons: `Button`, `ButtonFlat`, `ButtonFlatter`
Toggles: `CheckBox`, `Toggle`, `RadioButton`
Input: `Slider`, `DropDown`
Layout: `Splitter`, `FoldButton`, `FoldHeader`, `Hr`
Lists: `PortalList`
Navigation: `StackNavigation`, `ExpandablePanel`
Overlays: `Modal`, `Tooltip`, `PopupNotification`
Dock: `Dock`, `DockSplitter`, `DockTabs`, `DockTab`
Media: `Image`, `Icon`, `LoadingSpinner`
Special: `FileTree`, `PageFlip`, `CachedWidget`
Window: `Window`, `Root`
Markup: `Html`, `Markdown` (feature-gated)

### Widget Definition Pattern

```rust
// Rust struct
#[derive(Script, ScriptHook, Widget)]
pub struct MyWidget {
    #[source] source: ScriptObjectRef,  // Required for script integration
    #[walk] walk: Walk,
    #[layout] layout: Layout,
    #[redraw] #[live] draw_bg: DrawQuad,
    #[live] draw_text: DrawText,
    #[rust] my_state: i32,  // Runtime-only field
}

// For widgets with animations, add Animator derive:
#[derive(Script, ScriptHook, Widget, Animator)]
pub struct AnimatedWidget {
    #[source] source: ScriptObjectRef,
    #[apply_default] animator: Animator,
    // ...
}
```

### Script Module Structure

```rust
script_mod!{
    use mod.prelude.widgets_internal.*  // For internal widget definitions
    use mod.widgets.*                    // Access other widgets
    
    // Register base widget (connects Rust struct to script)
    mod.widgets.MyWidgetBase = #(MyWidget::register_widget(vm))
    
    // Create styled variant with defaults
    mod.widgets.MyWidget = set_type_default() do mod.widgets.MyWidgetBase{
        width: Fill
        height: Fit
        padding: theme.space_2
        
        draw_bg +: {
            color: theme.color_bg_app
        }
    }
}
```

### Key Syntax Differences (Old vs New)

| Old (live_design!) | New (script_mod!) |
|-------------------|-------------------|
| `<BaseWidget>` | `mod.widgets.BaseWidget{ }` |
| `{{StructName}}` | `#(Struct::register_widget(vm))` |
| `(THEME_COLOR_X)` | `theme.color_x` |
| `<THEME_FONT>` | `theme.font_regular` |
| `instance hover: 0.0` | `hover: instance(0.0)` |
| `uniform color: #fff` | `color: uniform(#fff)` |
| `draw_bg: { }` (replace) | `draw_bg +: { }` (merge) |
| `default: off` | `default: @off` |
| `fn pixel(self)` | `pixel: fn()` |
| `item.apply_over(cx, live!{...})` | `script_apply_eval!(cx, item, {...})` |

### Runtime Property Updates with script_apply_eval!

Use `script_apply_eval!` macro to dynamically update widget properties at runtime:
```rust
// Old system (live! macro with apply_over)
item.apply_over(cx, live!{
    height: (height)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [project-robius/robrix](https://github.com/project-robius/robrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
