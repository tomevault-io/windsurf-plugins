---
trigger: always_on
description: > **A README for AI coding agents** - This document provides context and instructions for AI coding agents working on snag. For human contributors, see [README.md](README.md).
---

# AGENTS.md

> **A README for AI coding agents** - This document provides context and instructions for AI coding agents working on snag. For human contributors, see [README.md](README.md).

## Project Overview

`snag` is a CLI tool that intelligently fetches web page content using Chrome/Chromium via the Chrome DevTools Protocol (CDP). Built for AI agents to consume web content efficiently.

**Key Features:**

- Auto-detect and connect to existing Chrome instances
- Launch headless or visible browser modes
- Handle authenticated sessions gracefully
- Multiple output formats: Markdown (default), HTML, Text, PDF, PNG
- Multiple URL support: Process multiple URLs in a single command
- Tab management: List, select, and fetch from existing browser tabs
- Pattern matching: Select tabs by index, exact URL, substring, or regex
- Output directory support: Auto-generate filenames with timestamps
- Single binary distribution, no runtime dependencies

**Technology Stack:**

- Language: Go 1.25.3
- CLI Framework: github.com/spf13/cobra v1.10.1
- Browser Control: github.com/go-rod/rod v0.116.2 (Chrome DevTools Protocol)
- HTML to Markdown: github.com/JohannesKaufmann/html-to-markdown/v2 v2.4.0
- HTML to Text: github.com/k3a/html2text v1.2.1

## Setup Commands

```bash
# Clone repository
git clone https://github.com/p3bot/snag.git
cd snag

# Install dependencies
go mod download

# Build binary
go build -o snag

# Run snag
./snag --version
./snag --help
```

## Build and Test Commands

```bash
# Build for current platform
go build -o snag

# Build with version info
go build -ldflags "-X main.version=1.0.0" -o snag

# Run all tests (integration tests with real browser)
go test -v

# Run specific test
go test -v -run TestFetchPage

# Run with coverage
go test -v -cover

# Test basic content fetching
snag https://example.com                # Fetch page as Markdown
snag --format html https://example.com  # Fetch as HTML
snag --format text https://example.com  # Fetch as plain text
snag --format pdf https://example.com   # Save as PDF (auto-generates filename)
snag --format png https://example.com   # Save as PNG screenshot (auto-generates filename)
snag -o output.md https://example.com   # Save to file

# Test multiple URL fetching
snag https://example.com https://google.com               # Fetch multiple URLs
snag -d output/ https://example.com https://google.com    # Save multiple with auto-generated names
snag -o combined.md https://example.com https://google.com # Combine into single file

# Test tab management features (Phase 2)
snag --open-browser                     # Open persistent browser (with DevTools enabled)
snag --list-tabs                        # List all open tabs
snag https://example.com                # Fetch URL (creates new tab)
snag --list-tabs                        # List tabs again (should show example.com)

# Tab selection by index (1-based)
snag --tab 1                            # Fetch from first tab
snag -t 2                               # Fetch from second tab (short form)

# Tab selection by pattern
snag -t "example.com"                   # Exact URL match (case-insensitive)
snag -t "example"                       # Substring/contains match
snag -t "https://.*\.com"               # Regex pattern match

# Tab features with output options
snag -t 1 --format html                 # Fetch tab 1 as HTML
snag -t "github" -o repo.md             # Fetch tab matching "github", save to file
snag -t 1 --wait-for ".content"         # Wait for selector in existing tab

# Cross-platform builds
GOOS=darwin GOARCH=arm64 go build -o snag-darwin-arm64
GOOS=darwin GOARCH=amd64 go build -o snag-darwin-amd64
GOOS=linux GOARCH=amd64 go build -o snag-linux-amd64
GOOS=linux GOARCH=arm64 go build -o snag-linux-arm64

# Code quality checks
go vet ./...
gofmt -l .

# Clean build artifacts
rm -f snag snag-*
```

## Code Style Guidelines

**Go Conventions:**

- Follow standard Go formatting: use `gofmt` or `goimports`
- Use Go 1.25.3+ features and idioms
- Keep functions focused and small
- Use descriptive variable names

**Project-Specific Patterns:**

- Flat project structure at root (main.go, browser.go, fetch.go, formats.go, handlers.go, output.go, logger.go, errors.go, validate.go)
- Custom logger for CLI output (logger.go)
- Sentinel errors for internal logic (errors.go)
- Exit codes: 0 (success), 1 (any error), 130 (SIGINT), 143 (SIGTERM)
- Output routing (critical for piping):
  - stdout: Content only (HTML/Markdown/Text or binary formats)
  - stderr: All logs, warnings, errors, progress indicators

**Naming Conventions:**

- Exported constants use PascalCase: `FormatMarkdown`, `FormatHTML`
- Sentinel errors: `ErrBrowserNotFound`, `ErrPageLoadTimeout`, etc.
- Functions: Use descriptive verbs - `validateURL()`, `fetchPage()`, `convertToMarkdown()`

**Error Handling:**

- Use sentinel errors defined in errors.go
- Wrap errors with context: `fmt.Errorf("failed to navigate to %s: %w", url, err)`
- Clear, actionable error messages via logger
- Never panic for expected errors

**Logging:**

- Use custom Logger with 4 levels (quiet, normal, verbose, debug)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p3bot/snag](https://github.com/p3bot/snag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
