---
trigger: always_on
description: This file is written for AI coding agents (Copilot, Claude Code, Cursor,
---

# AGENTS.md — Guide for AI coding agents using `dioxus-bulma`

This file is written for AI coding agents (Copilot, Claude Code, Cursor,
ChatGPT, etc.) so they can write idiomatic, working code with this crate
without trial-and-error. Human contributors are welcome to read it too — see
[`README.md`](README.md) for a more prose-style introduction.

If you are an agent: **read this whole file before generating code that uses
`dioxus-bulma`**. It is short and saves users from having to hand-correct
common mistakes.

---

## TL;DR

```toml
# Cargo.toml
[dependencies]
dioxus       = "0.7"
dioxus-bulma = "0.7"

# Optional, only if the user is using dioxus-router:
# dioxus-bulma = { version = "0.7", features = ["router"] }
```

```rust
use dioxus::prelude::*;
use dioxus_bulma::prelude::*;

fn App() -> Element {
    rsx! {
        BulmaProvider {
            theme: BulmaTheme::Light,
            load_bulma_css: true,
            BulmaTitle { "Hello, Dioxus Bulma!" }
            BulmaSubtitle { "Build beautiful web apps with Rust" }
            Button {
                id: "primary-cta",
                color: BulmaColor::Primary,
                size: BulmaSize::Large,
                onclick: move |_| (),
                "Click me"
            }
        }
    }
}

fn main() { dioxus::launch(App); }
```

---

## What this crate provides

A complete set of typed Dioxus components for [Bulma CSS](https://bulma.io/).
Every component:

- Is a Dioxus `#[component]` with a strongly-typed `Props` struct.
- Accepts an **`id: Option<String>`** prop forwarded to the rendered root
  element — use this for end-to-end tests, accessibility hooks, CSS targeting,
  etc.
- Accepts **`class: Option<String>`** (appended to the Bulma classes) and
  **`style: Option<String>`** (forwarded as `style="..."`).
- Uses Bulma's modifier conventions through the `BulmaColor` and `BulmaSize`
  enums where applicable.
- Takes `children: Element` when the underlying Bulma element supports
  children.

A non-exhaustive list, grouped roughly the same way as Bulma:

| Group | Components |
| --- | --- |
| Layout | `Container`, `Section`, `Columns`/`Column`, `Hero`/`HeroBody`/`HeroHead`/`HeroFoot`, `Level`/`LevelLeft`/`LevelRight`/`LevelItem`, `Media`/`MediaLeft`/`MediaContent`/`MediaRight`, `Tile` |
| Elements | `Button`, `Buttons`, `BulmaBox` (a.k.a. `Box`), `Block`, `Content`, `Delete`, `Icon`, `Image`, `Notification`, `Progress`, `Table`/`TableContainer`, `Tag`/`Tags`, `Title`/`Subtitle` (`BulmaTitle`/`BulmaSubtitle` in the prelude — see below) |
| Form | `Field`, `FieldLabel` (the `Label` component, renamed in the prelude), `Help`, `Control`, `Input` (+ `InputType`), `Textarea`, `Select`/`Option`, `Checkbox`, `Radio`, `File` |
| Components | `Card` (`CardHeader`/`CardHeaderTitle`/`CardContent`/`CardFooter`/`CardFooterItem`), `Breadcrumb`/`BreadcrumbItem`, `Dropdown` (+ `DropdownTrigger`/`DropdownMenu`/`DropdownItem`/`DropdownDivider`), `Menu`/`MenuLabel`/`MenuList`/`MenuItem`, `Message`/`MessageHeader`/`MessageBody`, `Modal`/`ModalCard`/..., `Navbar`/`NavbarBrand`/`NavbarMenu`/..., `Pagination`/..., `Panel`/..., `Tabs`/`Tab` |

For the canonical inventory, look at
[`src/prelude.rs`](src/prelude.rs) and
[`src/components/mod.rs`](src/components/mod.rs).

---

## Things agents commonly get wrong

### 1. Use the prelude — but watch out for renames

```rust
use dioxus::prelude::*;
use dioxus_bulma::prelude::*;
```

The prelude renames a couple of items to avoid collisions with `dioxus`,
`std`, etc.:

| Bulma name (in `dioxus_bulma::components`) | In `dioxus_bulma::prelude` |
| --- | --- |
| `Title` | `BulmaTitle` |
| `Subtitle` | `BulmaSubtitle` |
| `Box` (aliased to `BulmaBox`) | `BulmaBox` (and `Block`) |
| `Label` (form field label) | `FieldLabel` |

Why:

- `dioxus::prelude` already exports a *document* `Title` component (it sets
  `<title>`). The Bulma typography component lives at `BulmaTitle` so both
  can coexist.
- `Box` would collide with Rust's `std::boxed::Box`. The component type is
  named `BulmaBox`; the file also re-exports it as `Box` for users who want
  the original Bulma name and don't use `std::boxed::Box`.
- `Label` is too generic and was overloaded; the prelude calls it
  `FieldLabel`.

If you want the original names, import them explicitly from
`dioxus_bulma::components`:

```rust
use dioxus_bulma::components::{Title, Subtitle, Label, Box, Option as BulmaOption};
```

### 2. Setting an `id` (or any other unique attribute)

Just use the `id` prop. It is `Option<String>` on every component.

```rust
Button { id: "save-btn", "Save" }
Notification { id: "alert-1", color: BulmaColor::Warning, "Heads up" }
```

The `id` is rendered on the *root* HTML element of the component, which is
also the element that carries the Bulma class (`button`, `notification`,
`box`, etc.). This is the right thing to target from Playwright/Cypress and
from CSS.

### 3. Colors, sizes and modifiers — use the enums

Don't write `class: "is-primary is-large"` by hand. Use:

```rust
Button {
    color: BulmaColor::Primary,
    size: BulmaSize::Large,
    rounded: true,
    outlined: true,
    "Action"
}
```

Available enums: `BulmaColor` (Primary, Link, Info, Success, Warning, Danger,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rexlunae/dioxus-bulma](https://github.com/rexlunae/dioxus-bulma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
