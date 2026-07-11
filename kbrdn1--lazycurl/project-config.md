---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LazyCurl is a Terminal User Interface (TUI) HTTP client that combines Lazygit's multi-panel interface with Postman's API testing capabilities. Built with Go and Bubble Tea framework, it provides a keyboard-driven, vim-motion interface for managing HTTP requests, collections, and environments.

## Development Commands

### Build & Run

```bash
make build          # Compile to bin/lazycurl
make run            # Build and launch the application
make clean          # Remove binaries and caches
```

### Testing

```bash
make test           # Run all tests
make test-coverage  # Run tests with coverage report (generates coverage.html)
go test ./internal/api/...  # Test specific package
```

### Code Quality

```bash
make fmt            # Format code with gofmt
make lint           # Run golangci-lint (if installed)
```

### Development

```bash
make dev            # Live reload with air (auto-installs if missing)
make deps           # Download and tidy dependencies
```

### Multi-Platform Builds

```bash
make build-all      # Cross-compile for Linux/macOS/Windows (AMD64 & ARM64)
```

### Git Worktrees (Feature Branches)

**For humans**: Use the interactive menu:

```bash
make worktree       # Interactive worktree manager with gwq
```

**For Claude Code**: Use non-interactive `make worktree-create`:

```bash
# Create worktree for a feature (copies all necessary files + runs make deps)
make worktree-create TYPE=feat ISSUE=35 DESC=js-scripting

# This automatically:
# - Creates branch: feat/#35-js-scripting
# - Creates worktree at: ~/cc-worktree/LazyCurl/feat-35-js-scripting
# - Copies: CLAUDE.md, .lazycurl/, .claude/commands/, .specify/
# - Creates: specs/035-js-scripting/ folder
# - Runs: make deps

# List worktrees
make worktree-list

# Cleanup stale references
make worktree-cleanup
```

**After creation**: The user launches a new Claude Code session in the worktree:

```bash
cd ~/cc-worktree/LazyCurl/feat-35-js-scripting
claude
```

**Branch naming convention**: `<type>/#<issue>-<description>`

- Types: `feat`, `fix`, `hotfix`, `docs`, `test`, `refactor`, `chore`, `perf`, `ci`, `build`
- Example: `feat/#35-js-scripting`, `fix/#42-request-chaining`

**Remove worktree when done**:

```bash
gwq remove -b feat/#35-js-scripting
```

## Architecture

### Application Structure

**Bubble Tea Pattern**: LazyCurl follows the Elm architecture via Bubble Tea:

- **Model**: `internal/ui/model.go` - Central application state with 4 panels
- **Update**: Message-driven state updates with keybinding dispatch
- **View**: Lipgloss-styled rendering with Lazygit-inspired 3-panel layout

**Panel System**:

```
┌─────────────────┬──────────────────┐
│  Collections    │   Request        │
│  (left 1/3)     │   (top 40%)      │
│                 ├──────────────────┤
│                 │   Response       │
│                 │   (bottom 60%)   │
└─────────────────┴──────────────────┘
│          Status Bar                │
└────────────────────────────────────┘
```

- **CollectionsPanel**: Tree view of JSON collections (`.lazycurl/collections/*.json`)
- **RequestPanel**: Interactive request builder (method, URL, headers, body)
- **ResponsePanel**: Formatted HTTP response viewer
- **EnvironmentsPanel**: Overlay for variable management (toggled with 'e')

### Key Architectural Patterns

**Configuration System** (`internal/config/`):

- **Two-tier config**: Global (`~/.config/lazycurl/config.yaml`) + Workspace (`.lazycurl/config.yaml`)
- Global: Theme, keybindings, editor preference, workspace history
- Workspace: Project name, default environment, collection references
- All configs use YAML serialization via `gopkg.in/yaml.v3`

**Data Layer** (`internal/api/`):

- **Collections**: Hierarchical structure with folders, requests stored as JSON
- **Environments**: Variable substitution with `{{variable}}` syntax
- **HTTP Client**: Request execution with variable interpolation
- **Response Formatting**: JSON/XML/HTML formatting via `internal/format/`

**External Editor Integration** (`internal/api/`, `internal/ui/components/`):

- **Editor Detection** (`external_editor.go`): Auto-detect from `$VISUAL` → `$EDITOR` → fallback (`nano`, `vi`)
- **Temp File Management** (`temp_file.go`): Create temp files with smart extensions (`.json`, `.xml`, `.html`, `.txt`)
- **Content Type Detection**: Heuristic analysis based on content prefix (`{`/`[` → JSON, `<?xml` → XML, `<!doctype` → HTML)
- **Message Types** (`editor_messages.go`): `ExternalEditorRequestMsg`, `ExternalEditorFinishedMsg`, `ExternalEditorErrorMsg`
- **Error Categorization**: Typed errors (`EditorErrorNoEditor`, `EditorErrorNotFound`, `EditorErrorTempFile`, `EditorErrorReadContent`)

**External Editor Message Flow**:

```text
User presses Ctrl+E (INSERT mode)
    ↓
Editor.handleInsertMode() → ExternalEditorRequestMsg{Field, Content, ContentType}
    ↓
RequestView.Update() → forwards message unchanged
    ↓
Model.openExternalEditor():
  1. GetEditorConfig() → detect editor from env vars
  2. EditorConfig.Validate() → verify binary exists
  3. CreateTempFile() → write content with smart extension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kbrdn1/LazyCurl](https://github.com/kbrdn1/LazyCurl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
