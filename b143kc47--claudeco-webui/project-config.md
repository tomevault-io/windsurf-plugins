---
trigger: always_on
description: Enables Claude to perform structured, step-by-step reasoning:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Claude Code Web UI

A comprehensive web-based interface for the Claude CLI tool that provides streaming responses, integrated development tools, session management, and multi-device support.

## Code Quality

This project uses automated quality checks to ensure consistent code standards:

- **Lefthook**: Git hooks manager that runs `make check` before every commit
- **Quality Commands**: Use `make check` to run all quality checks manually  
- **CI/CD**: GitHub Actions runs the same quality checks on every push

The pre-commit hook prevents commits with formatting, linting, or test failures.

### Setup for New Contributors

1. **Install Lefthook**: 
   ```bash
   # macOS
   brew install lefthook
   
   # Or download from https://github.com/evilmartians/lefthook/releases
   ```

2. **Install hooks**:
   ```bash
   lefthook install
   ```

3. **Verify setup**:
   ```bash
   lefthook run pre-commit
   ```

The `.lefthook.yml` configuration is tracked in the repository, ensuring consistent quality checks across all contributors.

## Architecture

This project consists of three main components:

### Backend (Deno)

- **Location**: `backend/`
- **Port**: 8080 (configurable via CLI argument or PORT environment variable)
- **Technology**: Deno with TypeScript + Hono framework
- **Purpose**: Executes `claude` commands and streams JSON responses to frontend

**Key Features**:

- Command line interface with `--port`, `--help`, `--version` options
- Startup validation to check Claude CLI availability
- Executes `claude --output-format stream-json --verbose -p <message>`
- Streams raw Claude JSON responses without modification
- Sets working directory to project root for claude command execution
- Provides CORS headers for frontend communication
- Single binary distribution support
- Session continuity support using Claude Code SDK's resume functionality

**API Endpoints** (25+ endpoints organized by function):

### Core Chat & AI
- `POST /api/chat` - Streaming chat interface with Claude
  - Request: `{ message: string, sessionId?: string, requestId: string, allowedTools?: string[], workingDirectory?: string, thinkingBudget?: number }`
  - Supports thinking mode with configurable token budgets
- `POST /api/abort/:requestId` - Abort ongoing request

### Project Management
- `GET /api/projects` - List available project directories
- `GET /api/projects/:encodedProjectName/histories` - Get project history
- `GET /api/projects/:encodedProjectName/histories/:sessionId` - Get specific conversation

### Session & History
- `GET /api/sessions/:sessionId` - Retrieve saved session
- `POST /api/sessions/:sessionId` - Save session state
- `DELETE /api/sessions/:sessionId` - Delete session
- `GET /api/conversations` - List all conversations
- `GET /api/histories` - Get conversation histories

### Git Operations
- `GET /api/git/status` - Repository status
- `POST /api/git/stage` - Stage files
- `POST /api/git/unstage` - Unstage files
- `POST /api/git/commit` - Create commit
- `POST /api/git/push` - Push changes
- `POST /api/git/pull` - Pull changes
- `GET /api/git/branches` - List branches
- `POST /api/git/checkout` - Switch branches
- `GET /api/git/log` - Commit history
- `GET /api/git/diff` - View changes

### Terminal Integration
- `POST /api/terminal/execute` - Execute commands
- `GET /api/terminal/shells` - List active shells
- `POST /api/terminal/abort/:shellId` - Abort shell command
- `GET /api/terminal/info` - Terminal information
- `POST /api/terminal/validate-path` - Validate file paths

### File Management
- `GET /api/files/list` - Browse directory contents

### MCP Server Management
- `GET /api/mcp/smithery` - Browse available MCP servers
- `POST /api/mcp/install` - Install MCP server
- `DELETE /api/mcp/uninstall` - Remove MCP server
- `GET /api/mcp/config` - Get MCP configuration
- `POST /api/mcp/config` - Update MCP configuration

### Authentication & Devices
- `POST /api/auth/register` - Register new device
- `POST /api/auth/approve` - Approve device access
- `POST /api/auth/reject` - Reject device
- `GET /api/auth/devices` - List connected devices
- `DELETE /api/auth/devices/:deviceId` - Revoke device access

### Usage & Billing
- `GET /api/billing` - Billing information
- `GET /api/usage` - Usage statistics and analytics

### Network & System
- `GET /api/network/urls` - Get connection URLs
- `/*` - Static file serving (single binary mode)

### Frontend (React)

- **Location**: `frontend/`
- **Port**: 3000 (configurable via `--port` CLI argument to `npm run dev`)
- **Technology**: Vite + React + SWC + TypeScript + TailwindCSS + React Router
- **Purpose**: Provides project selection and chat interface with streaming responses

**Key Features**:

- **Project Directory Selection**: Choose working directory before starting chat sessions
- **Routing System**: Separate pages for project selection and chat interface
- Real-time streaming response display with modular message processing
- Parses different Claude JSON message types (system, assistant, result, tool messages)
- TailwindCSS utility-first styling for responsive design

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [B143KC47/claudeCO-webui](https://github.com/B143KC47/claudeCO-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
