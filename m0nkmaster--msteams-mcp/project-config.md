---
trigger: always_on
description: This document captures project knowledge to help AI agents work effectively with this codebase.
---

# Agent Guidelines for Teams MCP

This document captures project knowledge to help AI agents work effectively with this codebase.

## Repository

- **Repository**: https://github.com/m0nkmaster/msteams-mcp
- **Install**: `npx -y msteams-mcp@latest` or clone the repo, `npm install && npm run build`, then point your MCP client to `dist/index.js`

## Project Overview

This is an MCP (Model Context Protocol) server that enables AI assistants to interact with Microsoft Teams. Rather than using the complex Microsoft Graph API, it uses Teams APIs (Substrate, chatsvc, CSA) with authentication tokens extracted from a browser session. The browser is only used for initial login - all operations use direct API calls.

## Architecture

### Directory Structure

```
src/
├── index.ts              # Entry point, runs the MCP server
├── cli.ts                # Standalone CLI (msteams bin) - full tool parity via in-memory MCP transport
├── server.ts             # MCP server (TeamsServer class) - delegates to tool registry
├── constants.ts          # Shared constants (page sizes, timeouts, thresholds)
├── tools/                # Tool handlers (modular design)
│   ├── index.ts          # Tool context and type definitions
│   ├── registry.ts       # Tool registry - maps names to handlers
│   ├── search-tools.ts   # Search and channel tools
│   ├── message-tools.ts  # Messaging, favourites, save/unsave tools
│   ├── people-tools.ts   # People search and profile tools
│   ├── meeting-tools.ts  # Calendar and meeting tools
│   ├── file-tools.ts     # Shared files tools
│   └── auth-tools.ts     # Login and status tools
├── auth/                 # Authentication and credential management
│   ├── index.ts          # Module exports
│   ├── crypto.ts         # AES-256-GCM encryption for credentials at rest
│   ├── session-store.ts  # Secure session state storage with encryption
│   ├── token-extractor.ts # Extract tokens from Playwright session state
│   ├── token-refresh.ts  # Token refresh orchestrator (HTTP-first, browser fallback)
│   └── token-refresh-http.ts # Browserless token refresh via direct OAuth2 calls
├── api/                  # API client modules (one per API surface)
│   ├── index.ts          # Module exports
│   ├── substrate-api.ts  # Search and people APIs (Substrate v2)
│   ├── chatsvc-api.ts    # Barrel file re-exporting all chatsvc sub-modules
│   ├── chatsvc-common.ts # Shared utilities (date formatting)
│   ├── chatsvc-messaging.ts # Send, edit, delete, threads, 1:1/group chat
│   ├── chatsvc-activity.ts  # Activity feed (mentions, reactions, replies)
│   ├── chatsvc-reactions.ts # Add/remove emoji reactions
│   ├── chatsvc-virtual.ts   # Saved messages, followed threads, save/unsave
│   ├── chatsvc-readstatus.ts # Consumption horizons, mark as read, unread
│   ├── csa-api.ts        # Favorites API (CSA)
│   ├── calendar-api.ts   # Calendar/meetings API
│   ├── transcript-api.ts # Meeting transcripts (Substrate WorkingSetFiles)
│   ├── files-api.ts      # Shared files (Substrate AllFiles)
│   └── profile-api.ts    # Resolve MRIs to profiles (middleTier fetchShortProfile)
├── browser/              # Playwright browser automation (login only)
│   ├── context.ts        # Persistent browser profile management
│   └── auth.ts           # Authentication detection and manual login handling
├── utils/
│   ├── parsers.ts        # Pure parsing functions (barrel; testable submodules)
│   ├── parsers-reactions.ts # Emoji reaction parsing from raw messages
│   ├── parsers.test.ts   # Unit tests for parsers
│   ├── http.ts           # HTTP client with retry, timeout, error handling
│   ├── api-config.ts     # API endpoints and header configuration
│   └── auth-guards.ts    # Reusable auth check utilities (Result types)
├── types/
│   ├── teams.ts          # Teams data interfaces
│   ├── errors.ts         # Error taxonomy with machine-readable codes
│   ├── result.ts         # Result<T, E> type for explicit error handling
│   └── api-responses.ts  # Typed interfaces for raw API response shapes
├── __fixtures__/
│   └── api-responses.ts  # Mock API responses for testing
```

### Implementation Patterns

1. **Credential Encryption**: Session state and token cache are encrypted at rest using AES-256-GCM with a machine-specific key derived from hostname and username. Files have restrictive permissions (0o600).

2. **Server Class Pattern**: `TeamsServer` class encapsulates all state (browser manager, initialisation flag), allowing multiple server instances and simpler testing.

3. **Error Taxonomy**: Errors use machine-readable codes (`ErrorCode` enum), `retryable` flags, and `suggestions` arrays to help LLMs understand failures and recover appropriately.

4. **Result Types**: API functions return `Result<T, McpError>` for type-safe error handling with explicit success/failure discrimination.

5. **HTTP Utilities**: Centralised HTTP client (`utils/http.ts`) provides automatic retry with exponential backoff, request timeouts, and rate limit tracking.

6. **Dynamic Configuration from Session**: All tenant-specific configuration is extracted from the user's session localStorage, ensuring compatibility across different Teams environments (commercial, GCC, GCC-High, DoD):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m0nkmaster/msteams-mcp](https://github.com/m0nkmaster/msteams-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
