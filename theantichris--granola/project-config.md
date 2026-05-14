---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
 code in this repository.

## Project Overview

Granola CLI is a command-line tool for exporting notes and transcripts from the Granola
note-taking application. It provides two distinct export capabilities:

1. **Notes Export**: Connects to the Granola API, authenticates using bearer tokens,
   fetches AI-generated notes in JSON format, and converts them to clean Markdown files
2. **Transcripts Export**: Reads the local Granola cache file, extracts raw meeting transcripts
   with timestamps and speaker identification, and exports them to plain text files

## Common Commands

### Build

```bash
go build
```

### Run

```bash
# Export notes (AI-generated from API)
go run main.go notes
# Or after building:
./granola notes

# Export transcripts (raw from cache file)
go run main.go transcripts
# Or after building:
./granola transcripts
```

### Test

```bash
go test ./...
go test -v ./...  # verbose output
```

### Module Management

```bash
go mod tidy       # clean up dependencies
go mod download   # download dependencies
```

### Releases

```bash
# Create a new release (automated via GitHub Actions)
git tag v0.1.0
git push origin v0.1.0

# Test release locally (requires GoReleaser)
goreleaser release --snapshot --clean

# Check GoReleaser configuration
goreleaser check
```

### Linting

```bash
# Markdown linting (runs in GitHub Actions, installed via brew)
markdownlint-cli2 "**/*.md" "#notes" "#transcripts"

# Go linting (if golangci-lint is installed)
golangci-lint run
```

## Architecture

The project follows a modular Go CLI application structure:

- **Entry Point**: `main.go` - Uses Charmbracelet's fang for execution context
- **Command Structure**: `cmd/` directory contains Cobra command definitions
  - `cmd/root.go` - Defines the root command with configuration initialization using constructor pattern
  - `cmd/notes.go` - Implements the notes command for fetching and converting AI-generated notes from API
  - `cmd/transcripts.go` - Implements the transcripts command for reading and exporting raw transcripts from cache
- **Internal Packages**:
  - `internal/api/` - Granola API client with Supabase token authentication and document models (including ProseMirror structures)
  - `internal/cache/` - Cache file reader for extracting transcript data from local Granola cache
  - `internal/converter/` - Document to Markdown conversion with YAML frontmatter
  - `internal/prosemirror/` - ProseMirror JSON to Markdown conversion and plain text extraction
  - `internal/transcript/` - Transcript formatter for converting segments to readable text with timestamps
  - `internal/writer/` - File system writer for Markdown files with sanitization
- **Configuration**: Supports multiple configuration sources:
  - Environment variables via `.env` file (using godotenv)
  - Config file (`.granola.toml` in home directory or current directory)
  - Command-line flags:
    - Global: `--debug`, `--config`
    - Notes command: `--supabase`, `--timeout`, `--output`
    - Transcripts command: `--cache`, `--output`
  - Environment variable mapping: `SUPABASE_FILE`, `DEBUG_MODE`
- **Logging**: Uses Charmbracelet's log package for structured logging
  - Debug mode can be enabled via `--debug` flag or config
  - Logger includes timestamp and caller information
  - Log levels: Debug, Info, Warn, Error (defaults to Warn, Debug with debug flag)
  - Logger is created in Execute() and injected via dependency injection
  - **Logging Best Practices**:
    - Log errors only at the command level (cmd package) where they are handled
    - Internal packages should return errors without logging to avoid duplicates
    - Commands return errors to Cobra rather than logging them (Cobra handles display)
    - Debug/Info logging can occur at any level for progress tracking

## Key Dependencies

- **cobra**: Command-line interface framework
- **viper**: Configuration management (env vars, config files, flags)
- **afero**: Filesystem abstraction for testable file operations
- **charmbracelet/fang**: Enhanced command execution with context
- **charmbracelet/log**: Structured logging with customizable output formats
- **godotenv**: .env file support
- **net/http**: Standard library HTTP client for API communication
- **encoding/json**: Standard library JSON parsing

## Build and Release

- **GoReleaser**: Automated release management configured in `.goreleaser.yaml`
  - Builds for Linux, macOS (Darwin), and Windows
  - Creates tar.gz archives (zip for Windows)
  - Automatically generates changelog from commit messages
  - Triggered by pushing version tags (e.g., v1.0.0)

## Development Notes

- The root command is "granola" with "notes" and "transcripts" as the primary subcommands
- Commands use constructor pattern (e.g., `NewRootCmd()`, `NewNotesCmd()`, `NewTranscriptsCmd()`)
- Debug logging is available via the `--debug` flag for troubleshooting API calls
- Configuration precedence: flags > env vars > config file > defaults
- Logger is created in Execute() and passed via dependency injection (no globals)
- Releases are automated via GoReleaser when tags are pushed to GitHub
- Binary builds have CGO disabled for maximum portability


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theantichris/granola](https://github.com/theantichris/granola) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
