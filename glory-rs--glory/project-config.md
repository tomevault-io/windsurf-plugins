---
trigger: always_on
description: Project conventions and design guardrails for AI coding agents and contributors.
---

# AGENTS.md

Project conventions and design guardrails for AI coding agents and contributors.
Companion to [README.md](README.md). For the broader analysis and the prioritised
backlog see [`_report.md`](_report.md) and [`_todos.md`](_todos.md).

## What Glory is

Glory is an experimental Rust web framework. Components are built as **builder-pattern
Rust types** (`div().class("...").on(click, ...)`), not via an `rsx!` / JSX macro. The
reactive primitive is `Cage<T>` (mutable) / `Bond<T>` (derived) / `Lotus<T>` (the
read-only union). Targets today: browser (CSR via `wasm-bindgen`) and SSR (HTML
string output, integrated with Salvo). Roadmap: desktop webview, native rendering,
LiveView-style server driving — see `_report.md` §3 / `_todos.md` §3–§4.

## Crate layout

```
crates/
  core/         signals, view/widget/scope/node, web/csr + web/ssr,
                widgets (Each / Switch / Loader), SSR holder
  glory/        thin re-export front door, depends on core (+ optional routing)
  routing/      client-side router (currently wasm32-only)
  cli/          glory-cli build tool (wasm compile, css, hot reload server)
  hot-reload/   placeholder / partial; AST-diff types for future hot reload
examples/       per-app crates (counter, todomvc, hackernews-salvo, ssr-*, ...)
```

## Feature flags and combinations

`glory-core` features (also re-exported through `glory`):
- `web-csr` — client-side rendering in the browser. Implies `single-app`.
- `web-ssr` — server-side rendering, in-memory `Node` tree → HTML string.
- `salvo` — Salvo HTTP integration, implies `web-ssr`.
- `single-app` — **internal**. Automatic with `web-csr`. Switches the
  reactive scheduler to a single global instance instead of `HolderId`-keyed
  thread locals. Don't toggle manually outside CSR.

**Never enable `web-csr` + `web-ssr` in the same build.** They define `Node`
incompatibly (`web_sys::Element` vs an in-memory struct). Tests rely on this
exclusion — see `[package.metadata.cargo-all-features].skip_feature_sets`.

## Reactivity rules

- `Cage<T>` is a copyable reactive handle (`Copy`, `!Send`, `!Sync`).
  Cross-closure sharing no longer needs `.clone()`, though existing clones
  are harmless handle copies. Scope-owned cages are invalidated by `Owner`
  on scope drop; use `try_get*` / `try_revise*` when stale-handle recovery
  matters.
- `Bond<T>` re-runs its mapper when any dependency's `(id, version)` changes.
  Output is not value-compared by default — chain `.with_eq(|a, b| ...)`
  or `.with_partial_eq()` (for `T: PartialEq`) when you specifically want
  PartialEq gating, so observers only re-run on actual value change.
- Inside `Widget::build` / `Widget::patch`, calling `.get()` on a `Cage` or
  `Bond` **subscribes** the current view via `TRACKING_STACK`. Inside an
  event handler it does NOT subscribe (no active tracking layer). Use
  `.get_untracked()` if you want to read without subscribing.
- Mutations: call `Cage::revise(|mut v| ...)`. Repeated writes inside one
  user action should be wrapped in `reflow::batch(...)` to avoid intermediate
  re-renders. CSR event handlers are auto-batched by
  `Element::add_event_listener`, so writes inside a `.on(click, ...)`
  callback already flush once at the end of the event tick.

## Widget lifecycle

`Widget::build` → `Widget::attach` (default no-op) → `Widget::flood` (attach
children) → later `Widget::patch` when a bound signal revises → eventually
`Widget::detach`. Two facts that consistently trip people up:

- `show_in(parent)` stores + attaches; `store_in(parent)` only stores.
- `Scope::attach_child` early-returns when the child is already attached.
  If a widget needs to reposition a child in DOM order (e.g. `Each` reorder),
  set `view.scope.is_attached = false` AND `view.scope.placement =
  ViewPlacement::Unset` first, then call `attach_child`. See `Each::patch`.
- `Element::build` in both CSR and SSR sets `scope.first_child_node =
  scope.last_child_node = Some(self.node.clone())`. That value is the anchor
  Sibling-positioning uses; do not change it to `last_element_child()` —
  leaf elements like `<li>text</li>` would lose their anchor.

## Internal data structures

- `Scope::child_views: IndexMap<ViewId, View>` — order mirrors DOM order.
  Never call `.remove(...)` (deprecated, aliases `swap_remove`); use
  `shift_remove` to preserve order.
- `Scope::visible_views: IndexSet<ViewId>` — same rule.
- `Cage::view_ids`, `Bond::view_ids` — set membership; order irrelevant, but
  use `shift_remove` for consistency and to silence the deprecation warning.

## Code style

- `rustfmt` is enforced via `rustfmt.toml`. Use `cargo fmt --all` before
  committing.
- Prefer `Cow<'static, str>` for attribute keys/values at the API surface,
  since attrs come in as both string literals and dynamic strings. The
  rich `AttributeValue` enum is a roadmap item (`_todos.md` §3 P0).
- Don't add `unsafe` without justifying it in a code comment.
- New `Widget` types belong in `crates/core/src/widgets/`. Element factories
  (`div`, `li`, ...) are macro-generated by `generate_tags!` — extend that
  macro, don't hand-write a new one.

## Tests

- `cargo test -p glory-core --lib` runs the LIS unit tests under the
  default feature set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glory-rs/glory](https://github.com/glory-rs/glory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
