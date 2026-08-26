---
trigger: always_on
description: These instructions apply when building or reviewing a Fission-based app in this
---

# Fission App Guidelines

These instructions apply when building or reviewing a Fission-based app in this
tree.

## Source-Grounded Work

- Start from the real app entrypoint, then trace into screens, reusable widgets,
  and lower-level render behavior before changing UI code.
- For UI reviews, group findings by visible region. Do not stop at a screenshot
  description; trace the component and widget code that creates each issue.
- When a visual problem looks small, keep looking for related spacing,
  alignment, overflow, typography, state, and target-specific issues before
  reporting completion.
- Keep edits scoped to the component, widget, route, shell, or target behavior
  being changed. Avoid broad cleanup unless it is required for the task.

## Widget Structure

- Prefer one reusable widget per file when introducing app UI.
- Model widgets as concrete structs and implement `From<YourWidget> for Widget`.
- Use `#[fission_component]` for components that own retained local widget
  state. Do not model retained UI state as ordinary mutable struct fields.
- Keep screen modules focused on app state, routing, effects, and composition.
  Move reusable presentation pieces into widget modules.
- Avoid screen-level helper functions that build large `Widget` trees. If a UI
  fragment is meaningful or reused, make it a named widget struct instead.
- Small private helper functions are acceptable for narrow formatting,
  conversion, or leaf construction, but they should not hide reusable component
  boundaries. In fact, completely avoid functions that return Widget altogether if possible.
  Fission is a retained UI like Flutter, just like Flutter, functions break some of the optimisations Fission can do with Widget objects so building a UI with functions is an antipattern that should be avoided in almost all cases.

Local-state component shape:

```rust
use fission::prelude::*;

#[fission_component]
pub struct DisclosureSection {
    pub title: String,

    #[local_state(default = false)]
    open: bool,
}

#[fission_reducer(ToggleOpen)]
fn on_toggle_open(open: &mut bool) {
    *open = !*open;
}

impl From<DisclosureSection> for Widget {
    fn from(section: DisclosureSection) -> Widget {
        let (ctx, _) = fission::build::current::<()>();
        let open = section.open();
        let toggle = ctx.bind_local(ToggleOpen, open.clone(), reduce!(on_toggle_open));

        Column {
            gap: Some(8.0),
            children: widgets![
                Button {
                    on_press: Some(toggle),
                    child: Some(Text::new(section.title).into()),
                    ..Default::default()
                },
                if open.get() {
                    Text::new("The details are visible.").into()
                } else {
                    Text::new("The details are hidden.").into()
                },
            ],
            ..Default::default()
        }
        .into()
    }
}
```

Use `fission::build::current::<()>()` only when the component is intentionally
state-agnostic and does not read app state. Use the concrete app state type when
the component reads `GlobalState`, reads environment values tied to that app, or
binds reducers that update app state.

## State, Reducers, Routing and Runtime Data

- Choose the smallest state bucket that matches the lifetime and ownership of
  the data.
- Use `GlobalState` for durable app truth: product data, routing, sync,
  persistence, user preferences, shared filters, and values read by distant
  screens.
- Use `#[local_state]` for retained UI memory owned by one widget identity:
  open/closed flags, isolated draft text, local selected tabs, hoverless
  interaction state, or a counter-like local value.
- Use reducers on `GlobalState` when an action updates app data. Use
  `ctx.bind_local(...)` when an action updates one local-state field.
- Use reducers for explicit state transitions. Match the dispatched action to the
  reducer registration used by the widget.
- Do not store `BuildCtxHandle` or `ViewHandle` in structs, reducers, services,
  async tasks, statics, or other long-lived places. They are build-scope handles.
- Do not mutate `GlobalState` during component conversion. Dispatch actions and
  update state in reducers.
- Do not start network requests, file writes, or host operations during component
  conversion. Request work through effects, jobs, services, capabilities, or
  resources.
- For local-state components rendered from dynamic, reorderable, insertable, or
  filterable data, assign stable widget identities with
  `.id(WidgetId::explicit(...))`. Use a durable data id, not the list index.
- Do not pad production UI with fixture, mock, or demo data. Production paths
  should render persisted data, an explicit empty state, or a clear error state.
- Keep fixture parsing, fixture environment variables, and test-only data inside
  test infrastructure.
- When adding routed screens, verify the frontend route shape and any backend or
  deep-link route shape intentionally match.
- Use Fission's native Router and RouterParams (https://fission.rs/reference/widgets/router/). This ensures routing behaves consistently across ios, android, windows, linux, mac, terminal, static and server rendered sites


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keldra-store/keldra](https://github.com/keldra-store/keldra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
