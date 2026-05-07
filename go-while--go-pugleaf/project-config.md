---
trigger: always_on
description: **go-pugleaf** is a modern NNTP server and web gateway for Usenet/NetNews built in Go. It provides a complete newsgroup platform with full RFC 3977 compliant NNTP server implementation, modern web interface for browsing, efficient article fetching and threading, SQLite-based storage with per-group databases, spam flagging/moderation tools, and bridge integrations for Fediverse (ActivityPub) and Matrix protocols.
---

# Go-Pugleaf Copilot Instructions

## Project Overview

**go-pugleaf** is a modern NNTP server and web gateway for Usenet/NetNews built in Go. It provides a complete newsgroup platform with full RFC 3977 compliant NNTP server implementation, modern web interface for browsing, efficient article fetching and threading, SQLite-based storage with per-group databases, spam flagging/moderation tools, and bridge integrations for Fediverse (ActivityPub) and Matrix protocols.

### High-Level Repository Information
- **Type**: Production newsgroup server system (medium-to-large codebase)
- **Size**: 149 Go files across 13 internal packages, 21 command applications
- **Languages**: Go 1.25.0+ (primary), HTML/CSS/JavaScript (web frontend), Shell scripts (build system)
- **Frameworks**: Gin web framework, custom NNTP protocol implementation, SQLite3 database
- **Target Runtime**: Linux/Unix systems (Windows support not tested)
- **Dependencies**: Minimal external dependencies - SQLite3, Gin, Go crypto libraries

## Build, Test, and Development Commands

### Essential Prerequisites
```bash
# Always ensure Go 1.25.0+ is available
go version  # Must be 1.25.0 or higher

# Always run module commands before building
go mod tidy && go mod verify
```

### Build Commands (Always run in repository root)
```bash
# Build all applications (13 binaries) - takes ~15-30 seconds
./build_ALL.sh

# Build individual applications - takes ~1-2 seconds each
# 15 individual build scripts available:
./build_webserver.sh      # Main web interface
./build_fetcher.sh        # Article fetcher
./build_nntp-server.sh    # NNTP server
./build_expire-news.sh    # Article expiration tool
./build_merge-active.sh   # Merge active files utility
./build_merge-descriptions.sh  # Merge newsgroup descriptions
./build_nntpmgr.sh        # NNTP management tool
./build_analyze.sh        # NNTP analysis tool
./build_history-rebuild.sh     # History rebuild utility
./build_fix-references.sh      # Fix article references
./build_fix-thread-activity.sh # Fix thread activity
./build_import_flat-files.sh   # Import flat files
./build_recover-db.sh          # Database recovery
./build_rslight_importer.sh    # RSLight importer
./build_TestMsgIdItemCache.sh  # Message ID cache tester

# Note: Some cmd applications don't have build scripts:
# benchmark_hash, extract_hierarchies, history-demo, parsedates, test-nntp, usermgr

# Build output goes to build/ directory
ls -la build/  # List all built binaries
```

### Testing
```bash
# Run all available tests (very limited test coverage currently)
go test ./... -v

# Run specific package tests (currently only nntp package has tests)
go test ./internal/nntp/... -v

# Note: Most packages have no test files - this is expected
# Some nntp tests skip due to missing test data files - this is normal
```

### Linting (if golangci-lint is available)
```bash
# Project includes comprehensive .golangci.yml with security focus
golangci-lint run

# Security-focused linting config emphasizes:
# - gosec (security vulnerabilities)
# - errcheck (unchecked errors) 
# - sqlclosecheck (SQL connection handling)
```

### Running Applications
```bash
# Web server (main application)
./build/webserver -nntphostname your.domain.com
# Opens web interface on http://localhost:11980
# Requires "web" folder with templates to be present

# NNTP fetcher for downloading articles
./build/pugleaf-fetcher -group alt.test -test-conn

# NNTP server for serving articles
./build/pugleaf-nntp-server -port 1119

# Get help for any application
./build/webserver -help
./build/pugleaf-fetcher -help
```

### Important Build Notes
- **Always** run builds from repository root directory
- The `build/` directory is auto-created and contains all executables
- Build scripts use `-race` flag for race condition detection
- Version injection uses `appVersion.txt` file
- Clean builds remove `build/*` before building
- Build failures typically indicate missing dependencies

## Project Layout and Architecture

### Core Directory Structure
```
/cmd/                    # 21 command applications (main executables)
  ├── web/              # Main web server (primary app)
  ├── nntp-fetcher/     # Article fetching from NNTP providers  
  ├── nntp-server/      # NNTP server implementation
  ├── expire-news/      # Article expiration and cleanup
  ├── merge-active/     # Merge active files utility
  ├── merge-descriptions/ # Merge newsgroup descriptions utility
  ├── nntpmgr/          # NNTP management tool
  ├── nntp-analyze/     # NNTP analysis tool
  ├── history-rebuild/  # History rebuild utility
  ├── fix-references/   # Fix article references
  ├── fix-thread-activity/ # Fix thread activity
  ├── import-flat-files/ # Import flat files
  ├── recover-db/       # Database recovery
  ├── rslight-importer/ # RSLight importer
  ├── test-MsgIdItemCache/ # Message ID cache tester

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-while/go-pugleaf](https://github.com/go-while/go-pugleaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
