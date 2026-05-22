---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`fj` ("finance") is a command-line tool for managing business documents (invoices, offers, letters) using LaTeX and Git. It's designed for freelancers and small businesses, with a particular focus on German-speaking conventions. The tool maintains a Git-managed "fj home" directory structure and generates professional PDFs from JSON metadata and CSV billables.

Key capabilities:
- CLI-first design with optional web UI (`fj serve`)
- Git-based document versioning and archival
- LaTeX-powered PDF generation
- Structured client/rate/document management
- Travel expense tracking (web UI only)
- Support for encrypted S3 backups via git-remote-gcrypt

## Build System

This is a **Zig 0.15.1** project using the standard Zig build system.

### Common Commands

```bash
# Build the project
zig build

# Run the executable
zig build run -- [args]

# Run tests
zig build test

# Install to zig-out/bin/
zig build install
```

### Running fj commands during development

```bash
# Run with build system
zig build run -- init --generate config.json
zig build run -- serve --host=127.0.0.1 --port=3000

# Or build and run directly
zig build
./zig-out/bin/fj -h
```

### Dependencies

Third-party dependencies are managed via `build.zig.zon`:
- **zli**: CLI argument parsing (v0.1.3)
- **zap**: HTTP server framework (web UI functionality)
- **zeit**: Date/time handling (zig-0.15.1 branch)

C libraries are vendored in `src/`:
- **pdfgen**: PDF generation (`src/pdf/pdfgen.c`)
- **stb_image**: Image processing (`src/stb/`)
- **miniz**: Compression (`src/miniz/`)

## Code Architecture

### Core Entry Points

- **`src/main.zig`**: Main entry point. Parses CLI args via `zli`, dispatches to `Fj` command methods or starts the web server
- **`src/fj.zig`**: Core business logic (2500+ lines). Contains all `cmd*` methods that implement CLI commands
- **`src/cli.zig`**: CLI structure definitions using `zli` annotations

### Document Lifecycle

Documents (letters, offers, invoices) follow this workflow:

1. **`new`**: Creates working directory with LaTeX template, JSON metadata, and CSV for billables
   - Directory naming: `{type}--{year}-XXX--{client}/` (XXX = temporary ID)
   - Implemented in `cmdCreateNewDocument()` (fj.zig:1167)

2. **`compile`**: Runs `pdflatex` twice to generate PDF from JSON + CSV
   - Implemented in `cmdCompileDocument()` (fj.zig:2228)
   - Relies on external `pdflatex` command

3. **`commit`**: Assigns permanent ID, recompiles, moves to fj home, commits to Git
   - Implemented in `cmdCommitDocument()` (fj.zig:2332)
   - Uses file locking (`.id` files) for ID generation
   - Git auto-commit happens here

4. **Other operations**: `show`, `list`, `checkout`, `open`

### Key Subsystems

**Document Management** (`src/fj.zig`):
- Records: clients, rates (JSON-based, in `{fj_home}/clients/`, `{fj_home}/rates/`)
- Documents: letters, offers, invoices (LaTeX + JSON + CSV, in `{fj_home}/{type}s/`)
- ID generation uses atomic file locking
- Template expansion from `src/templates/*.tex`

**Web Server** (`src/web/`):
- `server.zig`: Zap-based HTTP server setup, authentication, routing
- `ep_*.zig`: Endpoint handlers (dashboard, document CRUD, git ops, travel expenses)
- `templates/`: HTML templates served by endpoints
- Authentication uses session cookies (`FJ_SESSION`)

**Git Integration** (`src/git.zig`):
- Wrapper around `git -C {fj_home}`
- Supports remote management, push/pull/status
- Auto-commits on document commit
- Works with encrypted remotes (git-remote-gcrypt for S3 backups)

**PDF/Image Processing**:
- `src/travelpdfs.zig`: Converts travel receipts (images) to PDFs, generates travel log
- Uses vendored C libraries (stb_image for loading, stb_image_resize for resizing, pdfgen for PDF creation)

**JSON Handling** (`src/json.zig`):
- Defines structs for Client, Rate, Letter, Offer, Invoice
- Uses std.json for parsing/stringifying

**Utilities**:
- `src/fatal.zig`: Error handling with clean exits (CLI mode) or HTTP 500 (server mode)
- `src/fsutil.zig`: Filesystem helpers (recursive copy, directory creation)
- `src/format.zig`: String formatting utilities
- `src/styleparser.zig`: LaTeX style file parser for config extraction
- `src/zeitlog.zig`: Custom logging with timezone support

### FJ_HOME Structure

The fj home directory (default `~/.fj`, overridden by `$FJ_HOME` or `-C` flag) has this layout:

```
{fj_home}/
├── .git/                    # Git repo for versioning
├── clients/
│   ├── .id                 # Next client ID (with file locking)
│   └── {shortname}.json    # Client records
├── rates/
│   ├── .id
│   └── {shortname}.json
├── letters/
│   ├── .id
│   └── letter--{year}-{id}--{client}/  # Committed letter directories
├── offers/
│   ├── .id
│   └── offer--{year}-{id}--{client}/
├── invoices/
│   ├── .id
│   └── invoice--{year}-{id}--{client}/
└── templates/
    ├── logo.png
    ├── config-defaults.sty  # LaTeX config
    ├── invoice.tex
    ├── offer.tex
    ├── letter.tex
    └── travel.tex
```

## Development Workflow

### Making Changes

1. Edit Zig source in `src/`
2. Run `zig build test` to verify tests pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [technologylab-ai/fj](https://github.com/technologylab-ai/fj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
