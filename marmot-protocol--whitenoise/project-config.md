---
trigger: always_on
description: This is a secure messaging app that uses the [whitenoise Rust crate](https://github.com/marmot-protocol/whitenoise-rs), which implements secure messaging using the [Marmot Protocol](https://github.com/marmot-protocol/marmot) with MLS (Messaging Layer Security) and Nostr.
---

# AGENTS.md

## Project Overview

This is a secure messaging app that uses the [whitenoise Rust crate](https://github.com/marmot-protocol/whitenoise-rs), which implements secure messaging using the [Marmot Protocol](https://github.com/marmot-protocol/marmot) with MLS (Messaging Layer Security) and Nostr.


## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    Flutter UI Layer                         │
│  (screens/, widgets/, hooks/, providers/)                   │
├─────────────────────────────────────────────────────────────┤
│              Flutter-Rust Bridge Layer                      │
│  (lib/src/rust/ - auto-generated bindings)                  │
├─────────────────────────────────────────────────────────────┤
│                    Rust API Layer                           │
│  (rust/src/api/ - thin wrapper around whitenoise)           │
├─────────────────────────────────────────────────────────────┤
│                Whitenoise Rust Crate                        │
│  (external dependency - core messaging logic)               │
└─────────────────────────────────────────────────────────────┘
```

## Tech Stack

- **Flutter/Dart** - UI and application logic
- **Rust** - Core messaging/crypto functionality via FFI
- **flutter_rust_bridge** - Dart-Rust FFI bindings
- **Riverpod** - State management (shared state)
- **flutter_hooks** - Ephemeral widget state
- **go_router** - Navigation/routing

## Directory Structure

```text
whitenoise/
├── lib/                    # Flutter/Dart source code
│   ├── main.dart           # App entry point
│   ├── routes.dart         # Route definitions (go_router)
│   ├── theme.dart          # Theme colors and styles
│   ├── providers/          # Riverpod providers (shared state)
│   ├── hooks/              # Flutter hooks (ephemeral state)
│   ├── screens/            # Full-page UI components
│   ├── widgets/            # Reusable components (see Widget Naming)
│   ├── services/           # Stateless operations (API calls)
│   ├── extensions/         # Dart extensions
│   ├── utils/              # Utility functions
│   ├── constants/          # Shared constants (fixed, related sets or reused elsewhere only)
│   └── src/rust/           # Auto-generated Rust bridge code (DO NOT EDIT)
├── rust/                   # Rust source code
│   └── src/api/            # API modules exposed to Flutter
├── test/                   # Flutter tests (mirrors lib/ structure)
├── trees/                  # Git worktrees for parallel development
├── assets/                 # Images, SVGs, fonts
└── scripts/                # Build/CI scripts
```

Use `constants/` only for fixed, related sets (e.g. NIP kinds) or constants repeated in multiple places; otherwise keep constants next to the code that uses them.

## Setup Commands

```bash
# Install all dependencies
just deps

# Install Flutter dependencies only
just deps-flutter

# Install Rust dependencies only
just deps-rust
```

## Development Commands

```bash
# Format all code (Rust + Dart)
just format

# Lint all code
just lint

# Run all tests (verbose output)
just test-flutter
just test-rust

# Run tests with coverage (99% minimum)
just coverage

# Generate coverage HTML report
just coverage-report

# Pre-commit checks (REQUIRED before every commit)
just precommit

# Pre-commit with verbose output (for debugging failures)
just precommit-verbose

# Regenerate flutter_rust_bridge code
just generate

# Rebuild Android native libraries (after Rust code/dependency changes)
just build-android-quiet
```

## Quiet Commands for Agents

**IMPORTANT:** When verifying that code works, agents should ALWAYS use the quiet variants. These produce minimal output that is easy to parse while still showing errors on failure.

```bash
# Quiet test commands - USE THESE for verification
just test-flutter-quiet    # Output: "+1093: All tests passed!" or error details
just test-rust-quiet       # Output: "....... test result: ok" or error details
just build-android-quiet   # Output: "✅ Android build complete" or error details

# Quiet pre-commit - USE THIS before committing
just precommit             # Shows step names + ✓/✗, errors only on failure
```

**Why quiet variants?**
- Minimal output reduces context window usage
- Clear pass/fail indicators are easy to parse
- Full error details are still shown when something fails
- No noisy progress indicators or dependency resolution messages

**Example quiet precommit output:**

```text
flutter deps...     ✓
rust deps...        ✓
l10n generation...  ✓
l10n validation...  ✓
auto-fix...         ✓
formatting...       ✓
linting...          ✓
flutter tests...    ✓
rust tests...       ✓
✅ PRECOMMIT PASSED
```

## Code Style

### Dart/Flutter

- Single quotes for strings
- `prefer_const_constructors` enabled
- `prefer_final_locals` enabled
- Line width: 100 characters
- Trailing commas: preserve

### Widget Naming

There are three categories of widgets with different naming rules:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmot-protocol/whitenoise](https://github.com/marmot-protocol/whitenoise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
