---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL RULES FOR AI ASSISTANTS

0. **Working Directory**: NEVER CHANGE WORKING DIRECTORY UNLESS SPECIFICALLY ASKED. THE CURRENT USER WORKFLOW IS BASED ON WORKTREES, BY RECKLESSLY CHANGING DIRECTORIES YOU CAN MAKE IRREVERSIBLE DAMAGE.

1. **Testing**: ALWAYS use the existing SVG-based snapshot testing in `tests/svg_snapshots.rs`. NEVER introduce new testing frameworks or approaches.
1a. **Sandbox-Safe Tests**: All tests must run in sandboxed environments (e.g., Nix builds). This means tests MUST NOT: rely on a writable home directory or system directories (`dirs::data_dir()`, `dirs::cache_dir()`, etc.); make network requests; depend on system fonts, a real TTY, or specific environment variables (`TERM`, `COLORTERM`, `TERM_PROGRAM`); assume standard tools exist in `PATH` beyond what's declared as dependencies. Use `tempfile::TempDir` for any filesystem operations, and inject/mock any external dependencies rather than relying on the host environment.
2. **Golden Snapshots**: NEVER update golden snapshot files with `SNAPSHOTS=overwrite` unless explicitly requested by the user. This is critical for test integrity.
3. **Test Updates**: NEVER update any test files or test expectations unless explicitly requested by the user. This includes unit tests, integration tests, and snapshot tests.
4. **File Creation**: Prefer editing existing files over creating new ones. Only create new files when absolutely necessary.
5. **Code Formatting**: NEVER manually reformat code or change indentation/line breaks. ONLY use `cargo fmt` for all formatting. When editing code, preserve the existing formatting exactly and let `cargo fmt` handle any formatting changes.
6. **Final Formatting**: ALWAYS run `cargo fmt` before reporting task completion if any code changes were made. This ensures consistent code formatting and prevents formatting-related changes in future edits.
7. **Comments/Annotations**: NEVER modify the comment storage format or location (`.bookokrat_comments/`) without explicit user request. The YAML-based persistence is critical.
8. **ANSI Art**: The `readme.ans` file contains binary CP437-encoded art. NEVER modify this file.
9. **Vendored Code**: The `src/vendored/` directory contains vendored ratatui-image code. This is NOT a crates.io dependency - it's vendored for customization.
10. **Kitty Graphics Protocol**: For Kitty terminals, ALWAYS use SHM (shared memory) transmission for images. NEVER use base64/direct transmission - it's too slow for 60fps PDF rendering.
11. **Backward Compatibility for Persistent Data**: When modifying any persistent data structures (bookmarks, settings, comments, or any files stored on the user's machine), ALWAYS ensure backward compatibility:
    - New fields in serialized structs MUST be `Option<T>` with `#[serde(skip_serializing_if = "Option::is_none")]` or have `#[serde(default)]`
    - Old app versions must be able to read new data files (serde ignores unknown fields by default - do NOT add `deny_unknown_fields`)
    - New app versions must be able to read old data files (missing fields default to `None` or sensible defaults)
    - If a breaking change is unavoidable, ASK the user how to design a migration path before implementing
    - Test both directions: old data → new app, and consider new data → old app
12. **Settings File Preservation**: The config file (`config.yaml`) uses a **targeted update** approach to preserve user edits. Key rules:
    - `save_settings_to_file()` reads the existing file and only updates **app-managed keys** in-place (version, theme, margin, pdf_*, book_sort_order). All comments, blank lines, and user-managed sections are preserved untouched.
    - `generate_settings_yaml()` is only used for **brand new config files** (first launch). It includes template comments for lookup_command and custom_themes.
    - **App-managed keys** (updated programmatically): `version`, `theme`, `margin`, `transparent_background`, `pdf_scale`, `pdf_pan_shift`, `pdf_render_mode`, `pdf_enabled`, `pdf_settings_configured`, `book_sort_order`
    - **User-managed keys** (only edited by hand in YAML): `lookup_command`, `lookup_display`, `custom_themes` — the app reads but never writes these
    - When adding a new setting: if it's app-managed, add it to `app_managed_key_values()`. If it's user-managed, only add it to `generate_settings_yaml()` (for new files) and to `migrate_settings()` (to append template for upgrading users).
    - **Migrations** (`migrate_settings()`) receive the file content as `&str` and return modified content. Insert new template sections at the correct position relative to existing sections (e.g., before custom_themes). The targeted update then writes the version bump on top.
    - NEVER regenerate the entire config file on save — this destroys user comments and formatting
13. **VHS Test Tapes and Keyboard Shortcuts**: When changing keyboard shortcut mappings, ALWAYS check and update all VHS test tapes in `vhs_tests/tapes/` to use the new keybindings. Test tapes simulate user input, so outdated keybindings will cause test failures.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bugzmanov/bookokrat](https://github.com/bugzmanov/bookokrat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
