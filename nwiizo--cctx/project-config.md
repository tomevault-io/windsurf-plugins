---
trigger: always_on
description: **cctx** (Claude Context) is a fast, secure, and intuitive command-line tool for managing multiple Claude Code `settings.json` configurations. Built with Rust for maximum performance and reliability.
---

# 🔄 CLAUDE.md - cctx Project Documentation

## 📋 Project Overview

**cctx** (Claude Context) is a fast, secure, and intuitive command-line tool for managing multiple Claude Code `settings.json` configurations. Built with Rust for maximum performance and reliability.

## 🏗️ Architecture

### 🎯 Core Concept
- **🔧 Context**: A saved Claude Code configuration stored as a JSON file
- **⚡ Current Context**: The active configuration (`~/.claude/settings.json`)
- **📁 Context Storage**: All contexts are stored in `~/.claude/settings/` as individual JSON files
- **📊 State Management**: Current and previous context tracked in `~/.claude/settings/.cctx-state.json`

### 📁 File Structure
```
📁 ~/.claude/
├── ⚙️ settings.json           # Current active context (managed by cctx)
└── 📁 settings/
    ├── 💼 work.json          # Work context
    ├── 🏠 personal.json      # Personal context
    ├── 🚀 project-alpha.json # Project-specific context
    └── 🔒 .cctx-state.json   # Hidden state file (tracks current/previous)
```

### 🎯 Key Design Decisions
1. **File-based contexts**: Each context is a separate JSON file, making manual management possible
2. **Simple naming**: Filename (without .json) = context name
3. **Atomic operations**: Context switching is done by copying files
4. **Hidden state file**: Prefixed with `.` to hide from context listings
5. **Predictable UX**: Default behavior always uses user-level contexts for consistency
6. **Progressive disclosure**: Helpful hints show when project/local contexts are available

## 🎯 Command Reference

### 🚀 Basic Commands
- `cctx` - List contexts (defaults to user-level, shows helpful hints)
- `cctx <name>` - Switch to context
- `cctx -` - Switch to previous context

### 🏗️ Settings Level Management
- `cctx` - Default: user-level contexts (`~/.claude/settings.json`)
- `cctx --in-project` - Project-level contexts (`./.claude/settings.json`)
- `cctx --local` - Local project contexts (`./.claude/settings.local.json`)

### 🛠️ Management Commands
- `cctx -n <name>` - Create new context from current settings
- `cctx -d <name>` - Delete context
- `cctx -r <old> <new>` - Rename context
- `cctx -c` - Show current context name
- `cctx -e [name]` - Edit context with $EDITOR
- `cctx -s [name]` - Show context content
- `cctx -u` - Unset current context

### 📥📤 Import/Export
- `cctx --export <name>` - Export to stdout
- `cctx --import <name>` - Import from stdin

## Implementation Details

### Language & Dependencies
- **Language**: Rust (edition 2021)
- **Key Dependencies**:
  - `clap` - Command-line argument parsing
  - `serde`/`serde_json` - JSON serialization
  - `dialoguer` - Interactive prompts
  - `colored` - Terminal colors
  - `anyhow` - Error handling
  - `dirs` - Platform-specific directories

### Error Handling
- Use `anyhow::Result` for all functions that can fail
- Provide clear error messages with context
- Validate context names (no `/`, `.`, `..`, or empty)
- Check for active context before deletion

### 🎨 Interactive Features
1. **fzf integration**: Auto-detect and use if available
2. **Built-in fuzzy finder**: Fallback when fzf not available
3. **Color coding**: Current context highlighted in green
4. **Helpful hints**: Shows available project/local contexts when at user level
5. **Visual indicators**: Emojis for different context levels (👤 User, 📁 Project, 💻 Local)

## 🚀 Release Management

### Simplified Release System

The project uses a streamlined release process with one primary tool:

#### **quick-release.sh** - Primary Release Script

A simple, reliable release script that handles the entire release process:

```bash
# One-command release
./quick-release.sh patch      # 0.1.0 -> 0.1.1
./quick-release.sh minor      # 0.1.0 -> 0.2.0  
./quick-release.sh major      # 0.1.0 -> 1.0.0
```

**What it does:**
1. ✅ Validates git state (clean working tree, on main branch)
2. ✅ Runs quality checks (fmt, clippy, test, build)
3. ✅ Updates version in Cargo.toml
4. ✅ Creates git commit and tag
5. ✅ Pushes to GitHub
6. ✅ Triggers GitHub Actions for:
   - Building release binaries for all platforms
   - Creating GitHub release with artifacts
   - Publishing to crates.io

#### **GitHub Actions Workflows**

**CI Pipeline** (`.github/workflows/ci.yml`):
- Multi-platform testing (Ubuntu, macOS, Windows)
- Rust stable version only
- Format checking, linting, tests
- Security audit
- MSRV (1.81) testing

**Release Pipeline** (`.github/workflows/release.yml`):
- Triggered by version tags (v*.*.*)
- Builds binaries for:
  - Linux x86_64 (glibc and musl)
  - Windows x86_64
  - macOS x86_64 and aarch64
- Creates GitHub release with all artifacts

**Publish Pipeline** (`.github/workflows/publish.yml`):
- Triggered by version tags
- Runs final quality checks
- Publishes to crates.io

#### **Justfile Integration**

For those who prefer `just`:
```bash
just release-patch    # Same as ./quick-release.sh patch
just release-minor    # Same as ./quick-release.sh minor
just release-major    # Same as ./quick-release.sh major
```

### Release Process

1. **Make your changes and commit them**
2. **Run the release command:**
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nwiizo/cctx](https://github.com/nwiizo/cctx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
