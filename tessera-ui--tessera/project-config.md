---
trigger: always_on
description: This document defines how You should assist in the Tessera project to ensure code and documentation are consistent with project architecture, style, and best practices.
---

# Instructions

This document defines how You should assist in the Tessera project to ensure code and documentation are consistent with project architecture, style, and best practices.

---

## Language Policy

- **All code, comments, and documentation comments must be written in English**, including rustdoc, commit messages, and PR descriptions. Use other languages only when absolutely necessary for functional clarification.
- **This document itself is a development guideline and should be maintained in English only. No i18n is required.**

---

## 🧠 Project Overview & Structure

- **Project Type**: Rust UI Framework
- **Core Crates**:
  - **tessera-ui**: Framework core (component tree, rendering, runtime, basic types Dp/Px, event handling, etc.)
  - **tessera-foundation**: Shared foundational UI building blocks (alignment, shape definitions, and future common modifier APIs)
  - **tessera-components**: Basic UI components (row, column, text, button, surface, etc.) and their rendering pipelines
  - **tessera-macros**: The `#[tessera]` procedural macro for simplified component definition
  - **example**: Example project demonstrating framework usage

**Module Path Convention**: All modules must use the `src/module_name.rs` pattern. Do not use `src/module_name/mod.rs`.

---

## 🏗️ Core Development Model

### Component Model & #[tessera] Macro

- Components are stateless Rust functions annotated with `#[tessera]`. Persistent UI state is created with `remember` (returned as `State<T>`) and can be passed around as `State<T>`.
- Inside the component function:
  - Compose child components to build the complete component tree
  - Use `Modifier` chains for node-local behavior such as layout modifiers, input, semantics, drawing, and focus
  - Use the internal layout primitive only inside framework/internal crates when a component must provide a custom layout policy
  - All child component closures must be executed to build the complete component tree

### Component API Pattern

- Truly zero-config components may use `#[tessera] pub fn component()` with no parameters.
- Configurable components must use a single public entrypoint declared with named owned parameters: `#[tessera] pub fn component(foo: Foo, bar: Bar, ...)`.
- Public component calls must use the generated builder syntax: `component().foo(value).bar(value);`.
- Do not introduce public wrapper variants (`*_with_controller`, `*_impl`, or extra public forwarding layers).
- Component parameters must use owned types that satisfy the macro requirements (`Clone`, `Default`, `Send`, `Sync`, and `'static` through the generated props model).
- `#[tessera]` parameter-level `#[default(...)]` is not supported. Do not introduce it in new or migrated code.
- For parameters with optional/default behavior, use `Option<T>` and resolve defaults explicitly in the component body (for example, `unwrap_or`, `unwrap_or_else`, or an explicit `if let Some(...)` fallback).
- Keep truly required parameters as non-`Option` constructor parameters.
- For optional external controllers, use `Option<State<...>>`; when `None`, create internal state with `remember`.
- Callback parameters should use `Callback` / `CallbackWith<...>`.
- Slot parameters should use `RenderSlot` / slot wrappers as needed by signature.
- Do not add `#[prop(skip_setter)]` to `Option<T>` just to keep builder setters "clean". The macro already generates setters that accept `T` and store `Some(T)`.
- Prefer `#[prop(into)]` for public `Option<T>` fields whose inner type has a useful conversion surface.
- Prefer `#[prop(render_slot)]` for public `RenderSlot` / `Option<RenderSlot>` parameters so the generated builder supports closure-style slot setters directly.
- Reserve `#[prop(skip_setter)]` for true internal plumbing fields that must not appear in the public builder surface. Do not use it on public authoring parameters that should already be expressible through the macro-generated setters.
- Exception: when a public builder needs a deliberately custom semantic surface (for example, mutually exclusive modes such as `title(...)` vs `label(...)`), use hidden backing fields with `#[prop(skip_setter)]` and expose explicit hand-written semantic setters instead of leaking mechanical field setters.
- `Callback`/`RenderSlot` are immutable handles in practice; do not rely on closure hot-swap semantics.
- If callback behavior depends on changing values, capture `State<T>` (or other stable handles) and read latest values at call time.
- Do not add runtime callback/slot helper wrappers (`callback`, `callback_with`, `render_slot`, `render_slot_with`); construct handles directly via `Callback::new`, `CallbackWith::new`, `RenderSlot::new`, and `RenderSlotWith::new`.

### Component Tree & Node Metadata

- The component tree is managed internally by `ComponentTree` and node metadata structures, supporting layout, rendering, and event dispatch.

---

## 📏 Layout & Measurement System

- Use `Constraint` and interval-based `AxisConstraint` to describe size constraints.
- Layout policies measure direct child layout nodes through handle-based APIs such as `MeasureScope`, `LayoutChild`, and `LayoutResult`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tessera-ui/tessera](https://github.com/tessera-ui/tessera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
