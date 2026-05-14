---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Server Manager is a Go web application that centralizes management of Model Context Protocol (MCP) servers across multiple AI clients. It solves the problem of manually editing various JSON files for different MCP clients by providing a single YAML configuration file and web interface.

**Key Features:**
- Single binary with embedded assets (no external dependencies)
- Cross-platform support (Linux, macOS, Windows)
- Real-time web interface with HTMX
- Dark/light theme system with manual override and system preference detection
- Automatic client config synchronization
- Server order preservation in UI and configuration
- Systemd integration for auto-start

## Development Guidelines

### Important Development Notes
- **DO NOT run `make run`** - The development server is already running in the background
- **DO NOT include Claude Code attribution in git commits** - Use semantic commit messages instead
- Use semantic git commit format: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, etc.
- Example: `feat: add syntax highlighting to config viewer` instead of generic attribution
- After completing a task, ask for confirmation then check it in the TODO list below and suggest the commit message

### Core Commands

- `make build` - Build the binary to `bin/mcp-server-manager` (single binary with embedded assets)
- `make test` - Run all Go tests
- `make test-coverage` - Run tests with coverage report (generates `coverage.out` for SonarQube)
- `make install-deps` - Download and organize Go dependencies
- `make setup` - Complete production setup (build, install systemd user service, enable, start)
- `make logs-service` - View systemd user service logs in real-time
- `make status-service` - Check systemd user service status
- `make sync-assets` - Sync web assets from web/ to internal/assets/web/ for embedding
- `make test-release` - Build local .deb package, install, and restart service for testing
- `make release VERSION=x.x.x` - Create git tag and trigger GitHub Actions release
- `make clean` - Remove build artifacts and coverage files

### Release Commands

- `make test-release` - Build and test .deb package locally before release
- `make release VERSION=v1.1.0` - Create official release with git tag and trigger GitHub Actions
- `make release VERSION=v1.1.0 MESSAGE="Custom release summary"` - Create release with custom message
- `make sync-assets` - Sync web assets to embedded location (auto-included in build/release)
- This triggers GitHub Actions to build cross-platform binaries via GoReleaser
- Produces releases for Linux, macOS, Windows (amd64 + arm64)

### Release Message Guidelines

When asked to create a release, Claude should:
1. Analyze commits since the last git tag to understand changes - **IMPORTANT**: Use `git log --format="%H%n%s%n%b%n---"` to read full commit bodies (not just one-line summaries)
2. Create a fun, engaging release summary in the style of YNAB release notes - casual, slightly humorous, user-focused
3. Summarize features/improvements in a non-technical way that users would understand and appreciate
4. Format the message with a **subject line**, **blank line**, and **body** for proper GoReleaser parsing
5. **IMPORTANT**: To avoid shell quoting issues, write the message to `/tmp/release_message.txt` and use git tag directly

**Message Format Requirements:**
- **First line**: Short, catchy subject (will appear as release title)
- **Second line**: Must be blank
- **Remaining lines**: Detailed description with features, improvements, etc.

**Recommended approach (avoids quoting issues):**
```bash
# Write message to temp file
cat > /tmp/release_message.txt << 'EOF'
Fresh updates for your MCP setup! 🎉

We've been busy little bees! 🐝 This release brings you a shiny new dark mode (because your eyes deserve better at 2 AM), smoother animations that'll make you smile, and we finally ditched that confusing global toggle that nobody understood anyway. Your MCP servers have never looked so good!
EOF

# Create and push release tag
make test && make sync-assets && git tag -a v1.4.0 -F /tmp/release_message.txt && git push origin v1.4.0
```

**Alternative (if MESSAGE parameter works):**
```bash
make release VERSION=v1.4.0 MESSAGE="Subject line

Body content..."
```

## Code Quality & Testing

### Test Coverage
- Project maintains **64.8% overall test coverage**
- Coverage tracking configured for SonarQube CI-based analysis
- Run `make test-coverage` to generate coverage reports locally
- Coverage file: `coverage.out` (auto-generated in CI, not committed)
- View coverage: `go tool cover -func=coverage.out` or `go tool cover -html=coverage.out`

### SonarQube Integration
- **Analysis Type**: CI-based analysis via GitHub Actions (`.github/workflows/sonarqube.yml`)
- **Configuration**: `sonar-project.properties`
- **Coverage Reporting**: Automatic via GitHub Actions workflow
  - Tests run with coverage: `go test -coverprofile=coverage.out ./...`
  - Coverage data sent to SonarQube Cloud on every push/PR
- **Strategic exclusions configured**:
  - Test file naming conventions (Go uses underscores)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlazic/mcp-server-manager](https://github.com/vlazic/mcp-server-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
