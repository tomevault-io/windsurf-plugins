---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the Navigator project.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the Navigator project.

## Project Overview

Navigator is a Go-based web server for multi-tenant web applications. It provides framework independence, intelligent request routing, dynamic process management, authentication, static file serving, managed external processes, and support for deployment patterns like Microsoft Azure's Deployment Stamps and Fly.io's preferred instance routing.

## Production Status

**IMPORTANT**: Navigator is production-ready and actively developed.

- **Single Implementation**: All development targets `cmd/navigator/`
- **Production Proven**: Battle-tested serving 75+ customers across 8 countries
- **Legacy Retired**: The legacy and refactored split has been unified into a single codebase

## Current Implementation Status

✅ **Framework Independence**: Support for Rails, Django, Node.js, and other web frameworks
✅ **Modular Architecture**: Well-organized internal packages in `cmd/navigator/`
✅ **YAML Configuration Support**: Modern YAML-based configuration
✅ **Managed Processes**: External process management (Redis, Sidekiq, workers, etc.)
✅ **Dynamic Port Allocation**: Finds available ports instead of sequential assignment
✅ **PID File Management**: Automatic cleanup of stale PID files with /tmp/navigator.pid
✅ **Graceful Shutdown**: Proper SIGTERM/SIGINT handling
✅ **Static File Serving**: Direct filesystem serving with try_files behavior
✅ **Authentication**: htpasswd support with multiple hash formats
✅ **Configuration Reload**: Live reload via SIGHUP signal without restart
✅ **Machine Idle Management**: Fly.io machine auto-suspend/stop after idle timeout
✅ **Intelligent Fly-Replay**: Smart routing with automatic fallback for large requests
✅ **WebSocket Support**: Full WebSocket connection support with standalone servers
✅ **High Reliability**: Automatic retry with exponential backoff for proxy failures
✅ **Simple Configuration**: Flexible variable substitution system for multi-tenant apps
✅ **Structured Logging**: Source-identified output with configurable JSON format for all processes
✅ **Lifecycle Hooks**: Server and tenant hooks for custom integration and automation
✅ **CGI Script Support**: Execute standalone scripts with user switching and config reload
✅ **Comprehensive Documentation**: Complete documentation site at https://rubys.github.io/navigator/

## Architecture

### Code Organization

Navigator has a single, production-ready implementation:

- **`cmd/navigator/`** - Production version with modular, well-organized codebase
  - Uses shared `internal/` packages for maintainability
  - All development happens here
  - Battle-tested in production

### Modular Package Structure

The refactored navigator uses focused internal packages for maintainability:
- **internal/server/**: HTTP handling, routing, static files, proxying
- **internal/process/**: Web app and managed process lifecycle
- **internal/config/**: Configuration loading and validation
- **internal/auth/**: Authentication (htpasswd)
- **internal/proxy/**: Reverse proxy and Fly-Replay logic
- **internal/idle/**: Fly.io machine idle management
- **internal/errors/**: Domain-specific error constructors
- **internal/logging/**: Structured logging helpers
- **internal/utils/**: Common utilities (duration parsing, environment, etc.)

**Design principles**:
- **Focused modules**: Each package has a single, clear responsibility
- **Easy deployment**: Single binary with minimal dependencies
- **Clear dependencies**: Only essential external Go packages
- **Maintainability**: Well-organized code with good separation of concerns
- **Testability**: Each module can be tested independently

### Key Components

1. **Configuration Loading** (`LoadConfig`, `ParseYAML`, `UpdateConfig`)
   - YAML configuration format (nginx format removed)
   - Supports template variable substitution for tenant configuration
   - Live configuration reload via SIGHUP signal

2. **Process Management** (`AppManager`, `ProcessManager`)
   - **Web Apps**: On-demand startup with dynamic port allocation
   - **Framework Configuration**: Configurable runtime and server executables
   - **Managed Processes**: External process lifecycle management
   - **PID File Handling**: Automatic cleanup of stale processes
   - **Graceful Shutdown**: Clean termination of all processes

3. **HTTP Handler** (`CreateHandler`)
   - **Rewrite Rules**: URL rewriting with redirect, last, and fly-replay flags
   - **Authentication**: Pattern-based auth exclusions with htpasswd
   - **Static Files**: Direct filesystem serving with caching
   - **Try Files**: File resolution for public content with multiple extensions
   - **Web App Proxy**: Reverse proxy to web applications with method exclusions
   - **Standalone Servers**: Proxy support for external services (Action Cable, etc.)
   - **Suspend Tracking**: Request tracking for idle machine suspension
   - **Proxy Retry**: Automatic retry logic with exponential backoff

4. **Static File Serving** (`serveStaticFile`, `tryFiles`)
   - **Performance**: Bypasses web framework for static content
   - **Try Files**: Attempts multiple file extensions before web app fallback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubys/navigator](https://github.com/rubys/navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
