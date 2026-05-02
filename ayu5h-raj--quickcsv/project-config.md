---
trigger: always_on
description: This file contains instructions for AI coding agents working on this project.
---

# Agent Instructions for QuickCSV

This file contains instructions for AI coding agents working on this project.

## Project Overview

QuickCSV is a high-performance CSV viewer for macOS and Web built with Rust and egui. It supports both native (desktop) and web (WASM) platforms.

## Tech Stack

- **Language**: Rust
- **GUI**: eframe/egui
- **File I/O**: memmap2 (memory-mapped files)
- **CSV Parsing**: csv crate
- **Threading**: std::thread with parking_lot locks

## Key Architecture Decisions

1. **Memory Mapping**: We use `memmap2` for zero-copy file access. Never load entire files into RAM.
2. **Progressive Loading**: Show data immediately, index in background.
3. **Virtualized Rendering**: Only parse/render visible rows.
4. **Row Cache**: Cache last 2000 parsed rows for smooth scrolling.

## Coding Standards

- Run `cargo fmt` before committing
- Run `cargo clippy -- -D warnings` to check for warnings
- Use inline format args: `format!("{value}")` not `format!("{}", value)`
- **ALWAYS use Phosphor icons** - Never use emoji or text-only buttons. Use `egui_phosphor::regular::ICON_NAME` for all icons
- **NEVER work directly on main branch** - Always create a feature/fix branch first
- **Only push to main** when bumping version and creating a release tag

## Build Verification

**CRITICAL: Always verify both desktop and web builds after making changes**

QuickCSV supports both native (macOS) and web (WASM) platforms. After any code modification:

1. **Check desktop build**: `cargo check` (or `cargo build`)
2. **Check web/WASM build**: `cargo check --target wasm32-unknown-unknown`

Both builds must compile successfully before committing. The web build uses different code paths (e.g., `#[cfg(target_arch = "wasm32")]`) and may expose issues not visible in the desktop build.

**Before committing, always run:**
```bash
cargo fmt
cargo clippy -- -D warnings
cargo check
cargo check --target wasm32-unknown-unknown
```

## Git Workflow

**CRITICAL: Always work on a branch, never on main directly**

1. **Always create a branch first**: `git checkout -b feature/feature-name` or `fix/bug-name`
2. Make changes and commit with conventional commits:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `perf:` for performance improvements
   - `docs:` for documentation
   - `ci:` for CI/CD changes
3. Push branch and create PR: `git push -u origin feature/feature-name`
4. After PR is merged to main, create tag for release: `git tag v0.X.X && git push origin v0.X.X`
5. **Only push directly to main** when:
   - Bumping version in Cargo.toml
   - Creating release tags
   - Hotfixes (create fix branch first, then merge)

## Release Process

1. Update version in `Cargo.toml` (both `package.version` and `package.metadata.bundle.version`)
2. Commit and push to main
3. Create and push tag: `git tag vX.X.X && git push origin vX.X.X`
4. GitHub Actions will automatically:
   - Build the macOS app bundle
   - Create GitHub Release
   - Update Homebrew tap

### Troubleshooting Release Issues

- **If tag already exists**: Delete and recreate it:
  ```bash
  git tag -d vX.X.X
  git push origin --delete vX.X.X
  git tag vX.X.X
  git push origin vX.X.X
  ```

- **YAML heredoc issues**: Avoid using heredocs (`<<`) in GitHub Actions for multi-line content that contains special characters (like Ruby's `#{}` interpolation). Use `printf` statements instead.

## Homebrew Tap

- Tap repo: `ayu5h-raj/homebrew-tap`
- Auto-updated on release via GitHub Actions
- Uses `postflight` to remove quarantine attribute (app is unsigned)

## Files Overview

- `src/main.rs` - All application code
- `Cargo.toml` - Dependencies and app metadata
- `ARCHITECTURE.md` - Technical documentation for developers
- `.github/workflows/` - CI/CD pipelines
  - `ci.yml` - Runs on every push (format check, clippy, build)
  - `release.yml` - Runs on version tags (build, release, homebrew update)

## Performance Considerations

- For large fields (>100KB), truncate display to 200 chars
- Skip search highlighting on fields >100KB
- Use `Cow<str>` to avoid unnecessary allocations
- Cache parsed rows to avoid repeated parsing

## UI Guidelines

- **Icons**: Always use Phosphor icons from `egui_phosphor::regular::`
  - Examples: `FOLDER_OPEN`, `CLOCK`, `FILE_TEXT`, `X`, `SEARCH`, `FILTER`, etc.
  - Format: `format!("{} Button Text", egui_phosphor::regular::ICON_NAME)`
  - Never use emoji (📂, 📄, ✕) - always use Phosphor icons
- **Consistency**: Use the same icon style throughout the app
- **Hover states**: Add `.on_hover_text()` for better UX
- **Visual feedback**: Use proper button states (hovered, clicked, disabled)

## UI Consistency Standards

**CRITICAL: All dialogs and popups must follow these standards for consistency**

### Dialog/Window Standards

1. **Dialog Titles**:
   - Always include a Phosphor icon: `format!("{} Dialog Name", egui_phosphor::regular::ICON_NAME)`
   - Never use emoji in titles
   - Examples:
     - Filter: `FUNNEL_SIMPLE`
     - Go to Row: `ARROW_CIRCLE_RIGHT`
     - Row Details: `LIST_BULLETS`
     - Cell Viewer: `FILE_CODE` / `FILE_TEXT`
     - Column Manager: `SLIDERS`

2. **Spacing Standards** (apply to ALL dialogs):
   - Top padding: `12.0` (ui.add_space(12.0))
   - Label-to-input gap: `6.0` (ui.add_space(6.0))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayu5h-raj/quickcsv](https://github.com/ayu5h-raj/quickcsv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
