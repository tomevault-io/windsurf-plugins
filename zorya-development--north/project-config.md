---
trigger: always_on
description: First-time setup (builds base image + dev image via compose):
---

# CLAUDE.md

## Build & Development Commands

First-time setup (builds base image + dev image via compose):

```bash
docker compose build
```

Start dependencies, then enter the app container:

```bash
docker compose up -d db redis
docker compose run --rm -ti --service-ports app bash
```

Inside the container, use `just` commands:

```bash
just dev               # Run dev server (cargo-leptos watch)
just test              # Run all tests
just test crate_name   # Run tests for specific crate
just fmt               # Format code
just lint              # cargo clippy
just migrate           # Apply migrations (diesel)
just migration name    # Create new migration
just migrate-revert    # Revert last migration
just migrate-redo      # Revert + reapply last migration
just build             # Release build
just seed              # Seed admin account
just check             # fmt + lint + test
just dupes             # Detect duplicated code (jscpd via npx, runs on host)
```

E2E tests (Playwright, run from host — not inside app container).
Test app runs `cargo leptos watch` — code changes auto-rebuild, no manual build needed.

```bash
just playwright              # Launch Playwright UI mode (port 8080)
just playwright-exec --grep "pattern"  # Run specific tests in already-running containers
just playwright-down         # Tear down test containers
```

For non-interactive / CI use: `docker compose exec app just <command>`

## Architecture Overview

GTD-inspired task management system. Single Rust binary serving SSR + WASM via Leptos 0.7 + Axum.

### Dependency Graph

```
dto → db → core → server-fns → repositories → stores → app
                                                        ↑
                                                       ui
server (Axum binary, calls core directly)
```

### Layer Rules (strict boundaries — never skip layers)

- **Page** (container/controller/view) → talks to **Stores** only
- **Store** (RwSignal, optimistic updates) → talks to **Repositories** only (never server-fns)
- **Repository** (thin facade) → talks to **Server Functions** only
- **Server Fn** (`#[server]` RPC boundary) → delegates to **Core**
- **Core** (business logic, Diesel queries) → reused by both Server Functions and REST API

### Data Flow

```
Read:  Page → Store → Repository → ServerFn ──RPC──→ Service → Diesel → PG
       View ← Memo(filter) ← RwSignal ← store.load()

Write: View → Callback → Store.update()
         ├─→ update_in_place()   (optimistic)
         └─→ Repository → ServerFn ──RPC──→ Service → Diesel → PG

REST:  HTTP → Axum Router → Auth Middleware → Route Handler → Core → Diesel → PG
```

## Key Decisions

These are non-obvious conventions. Violating them causes bugs or inconsistency.

- **Views never access stores.** Controller is the view-model. Views receive data (Memo/Signal) and callbacks; they only own pure UI state (toggles, input refs, local flags).
- **Pages own data loading.** Layout is purely structural (auth guard + context providers). Each page calls `refetch()` or creates its own `Resource`.
- **Container/controller/view pattern:** `container.rs` (entry, wires controller to view via inline `Callback` props), `controller.rs` (logic, store interaction), `view.rs` (pure rendering). Simpler components use two-file container/view.
- **Sub-components:** Extract inline `#[component]`s into a `components/` subdirectory (one file per component, `mod.rs` re-exports).
- **containers/ vs components/:** `containers/` = complex stateful domain components (pickers, task list, sidebar). `components/` = simpler/presentational (date picker, layout, modals).
- **Context:** Use `provide_context()` directly — no `.provide()` wrappers. Consume via `expect_context::<T>()` or `use_app_store()`.
- **Sequential tasks:** `compute_actionable()` in Rust, not SQL window functions.
- **Sort keys:** Fractional indexing (`sort_key` varchar), not integer `position`. See `dto/sort_key.rs`.
- **Diesel models:** `XxxRow` (Queryable), `NewXxx` (Insertable), `XxxChangeset` (AsChangeset).
- **Error handling:** `thiserror` for domain errors, `ServiceResult<T>`, no `anyhow`.
- **Time:** Always `chrono::Utc`, never naive datetimes.
- **Logging:** `tracing` crate, never `println!`.
- **Atoms:** Prefer `<Text variant=TextVariant::HeadingLg>` over raw Tailwind for typography. See `docs/UI_KIT.md`.

### Reactive Safety (avoiding disposed-signal panics)

Signals created inside a `<Show>` or conditional scope are disposed when that scope is torn down. Accessing them after disposal panics in WASM. Follow these rules:

1. **Never wrap `<For>` in a `{move || { ... }}` reactive closure.** The closure re-runs on signal changes, destroying and recreating the entire `<For>` and all child components. Instead, use stable `<Show>` components for loading/empty states and let `<For>` always be mounted (hide with `style:display`). The `<For>` handles its own keyed diffing.

2. **Blur before disposing focused elements.** When closing a modal/panel that may contain a focused input, call `html_el.blur()` on the active element *before* triggering the close signal. Otherwise the browser fires `blur` events into handlers with disposed signals. See `TaskDetailModal` container for the pattern.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zorya-development/north](https://github.com/zorya-development/north) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
