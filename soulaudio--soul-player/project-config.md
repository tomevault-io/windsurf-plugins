---
trigger: always_on
description: Instructions for Claude Code when working with Soul Player.
---

# CLAUDE.md

Instructions for Claude Code when working with Soul Player.

---

## Project Overview

**Soul Player**: Cross-platform music player (Desktop/Server/Mobile)
- Stack: Cargo + Yarn workspaces + Tauri
- Storage: SQLite multi-user schema
- Audio: Symphonia decoder + platform-specific output
- Languages: Rust (backend/libs) + TypeScript/React (frontend)
- Structure: `applications/` (platform apps), `libraries/` (shared Rust libs)

---

## Critical Rules (MUST Follow)

### 1. Multi-User Always
ALL database queries MUST include `user_id`. Desktop uses `user_id = 1`, server uses authenticated ID.

```rust
// ✅ CORRECT
pub async fn get_playlists(pool: &SqlitePool, user_id: i64) -> Result<Vec<Playlist>> {
    sqlx::query_as!(Playlist, "SELECT * FROM playlists WHERE owner_id = ?", user_id)
        .fetch_all(pool).await.map_err(Into::into)
}
```

### 2. Database: Compile-Time Queries Only
Use `query!`/`query_as!` macros ONLY. Never `query().bind()`. Setup: [docs/SQLX_SETUP.md](./docs/SQLX_SETUP.md)

### 3. Platform-Agnostic Core
Libraries (`libraries/*`) MUST NOT depend on platform crates. Use traits. Platform code in `applications/` only.

### 4. Audio Safety: No Allocations
Audio callbacks MUST NOT allocate. Pre-allocate buffers in constructors.

### 5. Test Quality: No Shallow Tests
- Test: business logic, edge cases, error paths
- Skip: getters, setters, trivial constructors
- Use testcontainers with real SQLite (never in-memory)
- Target: 50-60% meaningful coverage

### 6. Error Handling
- Libraries: `thiserror` + `Result`, no `.unwrap()` in public APIs
- Applications: `.expect()` with clear messages is fine

### 7. Always Localize UI Strings
ALL user-facing strings MUST use localization. NEVER hardcode text.

```typescript
// ✅ CORRECT
<button>{t('playback.play')}</button>
<div>{t('errors.file_not_found', { filename })}</div>
```

### 8. Structured Logging Only
Use `tracing` crate ONLY. NEVER `println!`, `eprintln!`, `dbg!()`. Exception: `init_logging()` may use `eprintln!` before initialization.

```rust
// ✅ CORRECT
tracing::info!("[SCAN] Processing: {}", file_path.display());
tracing::error!("[SCAN] TIMEOUT on file: {}", file_path.display());
```

**Log locations**: `%APPDATA%\Soul Player\logs\` (Win), `~/Library/Application Support/soul-player/logs/` (Mac), `~/.config/soul-player/logs/` (Linux)

### 9. Frontend Styling: CSS-First (Tailwind v4)

**Core Principles**:
1. CSS variables for theming (in `:root`)
2. Data attributes for state (`data-state`, `aria-current`) - NOT class-based
3. Separate hover and selected states
4. Opacity-based hover effects - NOT color changes
5. Avoid custom Tailwind plugins

**Standard Patterns**:

```tsx
// Navigation Item (Mutually Exclusive)
<button
  data-state={isActive ? 'active' : 'inactive'}
  aria-current={isActive ? 'page' : undefined}
  className={cn(
    'transition-all',
    isActive
      ? 'text-primary bg-accent/20'
      : 'text-muted-foreground hover:opacity-80 hover:bg-foreground/10'
  )}
/>

// Toggle Button (Active + Hover)
<button
  data-state={isEnabled ? 'on' : 'off'}
  aria-pressed={isEnabled}
  className={cn(
    'transition-opacity hover:opacity-80',
    isEnabled ? 'text-primary' : 'text-muted-foreground'
  )}
/>
```

**Interaction Values**:
- Hover (text): opacity 0.8
- Hover (background): `foreground/10`
- Selected: opacity 1.0, `accent/20` background
- Disabled: opacity 0.5, no hover

**Related Files**: `applications/shared/src/styles/globals.css`, `applications/shared/src/theme/ThemeManager.ts`

---

## Essential Commands

Soul Player uses `cargo xtask` for all development automation. Use `cargo xt` as shorthand.

### First-Time Setup
```bash
corepack enable && yarn
cargo xtask setup all        # Complete setup (deps + sqlx + env)
# Or step-by-step:
cargo xtask setup deps       # Install system dependencies
cargo xtask setup sqlx       # Setup SQLx database
cargo xtask setup env        # Setup environment files
```

### Development
```bash
cargo xtask dev desktop      # Desktop app with hot reload
cargo xtask dev marketing    # Marketing site dev server
# Or use yarn directly:
yarn dev:desktop
yarn dev:marketing
```

### Quality Checks
```bash
cargo xtask check precommit  # Full pre-commit pipeline (MUST pass before commit)
# Or individual checks:
cargo xtask check fmt [--fix]        # Rust formatting
cargo xtask check clippy [--fix]     # Clippy lints
cargo xtask check test               # Rust tests
cargo xtask check typescript         # TypeScript type checks
cargo xtask check lint [--fix]       # ESLint
```

### Build
```bash
cargo xtask build desktop [--release]   # Build desktop app
cargo xtask build wasm [--watch]        # Build WASM modules
cargo xtask build all                   # Build everything
```

### Testing
```bash
cargo xtask test audio e2e    # Audio E2E tests
cargo xtask test import e2e   # Import tests
cargo xtask test cache e2e    # Cache tests
```

### Cleanup
```bash
cargo xtask clean dev         # Clean dev artifacts (fast)
cargo xtask clean full        # Nuclear clean (removes node_modules)
```

### Database Migrations
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soulaudio/soul-player](https://github.com/soulaudio/soul-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
