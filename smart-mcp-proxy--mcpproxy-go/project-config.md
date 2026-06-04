---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## Autonomous Operation Constraints
When operating to complete a task, adhere strictly to the following constraints to ensure zero-interruption execution:

### Must-Do (Defaults & Assumptions)
- **Zero Interruption Policy**: If a decision is needed and no explicit instruction exists, you MUST make an informed, safe assumption based on idiomatic Go best practices and document it in the PR/commit. Do NOT ask for human clarification mid-task.
- **Test-Driven Progress**: You must write a failing Go test (`_test.go`) for every sub-task before implementing the feature.
- **Graceful Fallbacks**: If an API or dependency lacks documentation, use mock interfaces or a simplified implementation rather than blocking the task.
- **Continuous Logging**: Document every step completed in an `execution_log.md` within the current working directory to maintain state.

### Must-Nots
- **Do NOT ask for plan approval**: Once a plan/spec is generated, begin execution immediately.
- **Do NOT stop for code style choices**: Run `gofmt` or `goimports` and strictly follow standard Go conventions.

### Escalation Triggers (Stop Conditions)
Only halt execution and ask a human IF:
1. You need to perform destructive data operations or delete core proxy logic that cannot be mocked.
2. A required environment variable is missing from `.env` and cannot be mocked for the scope of the task.
3. You are stuck in an error loop for the same `go test` failing after 5 consecutive attempts.



## Project Overview

MCPProxy is a Go-based desktop application that acts as a smart proxy for AI agents using the Model Context Protocol (MCP). It provides intelligent tool discovery, massive token savings, and built-in security quarantine against malicious MCP servers.

## Editions (Personal & Server)

MCPProxy is built in two editions from the same codebase using Go build tags:

| Edition | Build Command | Binary | Distribution |
|---------|--------------|--------|-------------|
| **Personal** (default) | `go build ./cmd/mcpproxy` | `mcpproxy` | macOS DMG, Windows installer, Linux tar.gz |
| **Server** | `go build -tags server ./cmd/mcpproxy` | `mcpproxy-server` | Docker image, .deb package, Linux tar.gz |

> Every feature decision should ask: "Does this make the personal edition so good that developers tell their teammates about it?"

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `cmd/mcpproxy/edition.go` | Default edition = "personal" |
| `cmd/mcpproxy/edition_teams.go` | Build-tagged override for server edition |
| `cmd/mcpproxy/teams_register.go` | Server feature registration entry point |
| `internal/teams/` | Server-only code (all files have `//go:build server`) |
| `internal/teams/auth/` | OAuth authentication, session management, JWT tokens, middleware |
| `internal/teams/users/` | User/session models, BBolt store, user server management |
| `internal/teams/workspace/` | Per-user workspace manager for personal upstream servers |
| `internal/teams/multiuser/` | Multi-user router, tool filtering, activity isolation |
| `internal/teams/api/` | Server REST API endpoints (user, admin, auth) |
| `native/macos/MCPProxy/` | Swift macOS tray app (SwiftUI, macOS 13+) |
| `native/macos/MCPProxyUITest/` | Swift MCP server for UI testing (accessibility + screenshots) |
| `native/windows/` | Future C# tray app (placeholder) |

### Edition Detection

The binary self-identifies its edition:
- `mcpproxy version` → `MCPProxy v0.21.0 (personal) darwin/arm64`
- `/api/v1/status` → `{"edition": "personal", ...}`

## Server Multi-User Authentication (Spec 024)

Server edition supports OAuth-based multi-user authentication with Google, GitHub, or Microsoft identity providers.

### Server Configuration

```json
{
  "teams": {
    "enabled": true,
    "admin_emails": ["admin@company.com"],
    "oauth": {
      "provider": "google",
      "client_id": "xxx.apps.googleusercontent.com",
      "client_secret": "GOCSPX-xxx",
      "tenant_id": "",
      "allowed_domains": ["company.com"]
    },
    "session_ttl": "24h",
    "bearer_token_ttl": "24h",
    "workspace_idle_timeout": "30m",
    "max_user_servers": 20
  }
}
```

### Server API Endpoints

| Endpoint | Auth | Description |
|----------|------|-------------|
| `GET /api/v1/auth/login` | Public | Initiate OAuth login flow |
| `GET /api/v1/auth/callback` | Public | OAuth callback (creates session) |
| `GET /api/v1/auth/me` | Session/JWT | Get current user profile |
| `POST /api/v1/auth/token` | Session | Generate JWT bearer token for MCP |
| `POST /api/v1/auth/logout` | Session | Invalidate session |
| `GET /api/v1/user/servers` | Session/JWT | List user's servers (personal + shared) |
| `POST /api/v1/user/servers` | Session/JWT | Add personal upstream server |
| `GET /api/v1/user/activity` | Session/JWT | User's activity log |
| `GET /api/v1/user/diagnostics` | Session/JWT | Server health for user's servers |
| `GET /api/v1/admin/users` | Admin | List all users |
| `POST /api/v1/admin/users/{id}/disable` | Admin | Disable a user |
| `GET /api/v1/admin/activity` | Admin | All users' activity logs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smart-mcp-proxy/mcpproxy-go](https://github.com/smart-mcp-proxy/mcpproxy-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
