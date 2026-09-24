---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Amber is a Tauri 2 desktop app (React 19 frontend, Rust backend) for incremental learning.

## Code Comments

Keep comments to one or two lines. If a comment needs more than that, the code likely needs restructuring or a docs page instead.

## UI Guidelines

- Use **Mantine** (`@mantine/core`, `@mantine/hooks`) components for all UI. Prefer built-in Mantine components over building custom ones.
- Use **`@phosphor-icons/react`** for icons.
- Use **`AppTooltip`** (`src/components/AppTooltip/AppTooltip.tsx`) instead of Mantine's `Tooltip` — it takes the same props and accepts a `shortcut` prop (raw `useHotkeys` notation, e.g. `"mod+K"`). Never hand-append a shortcut to a tooltip label. Pass `touch` to also open on tap, but only on targets whose meaning is otherwise unreachable on touch (info icons, study session buttons) — not on action buttons whose label is already visible.
- **Never display a keyboard shortcut on touch input** — there's no keyboard to press it with. Every shortcut shown in the UI must be rendered through `useShortcutDisplay()` (`src/commands/useShortcutDisplay.ts`), which formats it and yields `undefined` on a coarse pointer; `AppTooltip`'s `shortcut` prop and `useCommandShortcut(id)` already go through it. Never call `formatShortcut` directly from a component.
- Avoid custom CSS. Use Mantine's built-in style props (`p`, `px`, `h`, `w`, `gap`, `justify`, `align`) and inline `style` objects only when Mantine props are insufficient. Do not create `.module.css` files for layout or cosmetic concerns that Mantine already covers.

## Commands

```bash
# Development
npm run tauri dev        # Start full Tauri dev environment
npm run dev               # Vite dev server only (no Tauri shell)

# Build
npm run tauri build       # Production build
npm run build             # TypeScript check + Vite build only
npm run android:dev       # Tauri Android dev build
npm run android:build     # Tauri Android production build

# Linting & Formatting
npm run lint               # ESLint
npm run format             # Prettier

# Testing
npm run test               # Vitest unit tests
npm run uitest              # Vitest with the interactive UI
npm run coverage            # Coverage report
```

### Rust backend

```bash
cd src-tauri
cargo build --features wry
cargo test --workspace --features wry
cargo clippy --all-targets --features wry
```

`Cargo.toml` has **no default runtime feature** — each platform's `tauri.<platform>.conf.json` opts into its own runtime (**CEF on Linux, WRY elsewhere**). Bare `cargo build`/`test`/`clippy` therefore fails to compile (`tauri::Wry` not found); always pass `--features wry` for local dev/CI regardless of host OS. Never add a `default` feature — it would silently combine with a platform's own choice.

`[patch.crates-io]` redirects `tauri`/`tauri-plugin-*`/`tauri-runtime-wry` to a `feat/cef` fork (adds CEF support upstream lacks). Behavior may diverge from published Tauri — keep this in mind when a bug looks upstream.

## Backend Architecture (`src-tauri/src/`)

The backend follows **onion architecture** (Clean Architecture) with custom dependency injection.

### Layers (inner → outer)

| Layer          | Directory                                                                 | Role                                                                    |
| -------------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Domain         | `entities/`, `value_objects/`                                             | Pure data models, no I/O                                                |
| Application    | `services/`                                                               | Reusable business logic; keeps presentation handlers thin               |
| Presentation   | `<module>_api.rs` (or an `api/` subdir for larger modules like `backend`) | Tauri command handlers; resolves dependencies and delegates to services |
| Infrastructure | `repositories/infrastructure/`, `infrastructure/`                         | SQLite, HTTP clients                                                    |

### Dependency Injection

A custom `injector` crate (with derive macros in `injector_derive`) wires everything together. Services are registered in `common/utils/create_injector.rs`. Every Tauri command handler follows this pattern:

```rust
#[tauri::command]
async fn some_command(injector: State<'_, Arc<Injector>>, ...) -> Result<Dto, ApiError> {
    let scope = injector.start_scope();
    let service = scope.resolve::<dyn SomeService>();
    let result = service.do_work(...).await?;
    scope.save_changes().await?;  // Unit of Work — commits DB transaction
    Ok(result)
}
```

### Bulk Operations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambercrew/amber](https://github.com/ambercrew/amber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
