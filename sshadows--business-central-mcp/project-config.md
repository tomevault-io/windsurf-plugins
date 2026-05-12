---
trigger: always_on
description: This project is NOT released and in active development:
---

# BC MCP Server v2

## Development Philosophy

This project is NOT released and in active development:
- Always choose the best solution, not the quickest compromise
- Refactor aggressively when architecture is flawed
- Fix problems properly, not with workarounds
- No stubs, mocks, or skeleton implementations -- everything must be fully functional
- No backwards compatibility concerns -- make breaking changes freely

## Quick Start

### Project Location
- **v2 source**: `U:/git/bc-mcp/`
- **v1 reference** (deprecated): `C:\bc4ubuntu\Decompiled\bc-poc\`
- **Decompiled BC28**: `U:/git/bc-mcp/reference/bc28/decompiled/`
- **Decompiled BC27**: `C:\bc4ubuntu\Decompiled\` (various Microsoft.Dynamics.* directories)
- **Architecture spec**: `C:\bc4ubuntu\Decompiled\bc-poc\docs\superpowers\specs\2026-04-03-bc-mcp-v2-design.md`

### BC Test Environments

| | BC27 | BC28 |
|---|---|---|
| URL | http://cronus27/BC/?tenant=default | http://cronus28/BC/?tenant=default |
| Username | sshadows | sshadows |
| Password | 1234 | 1234 |
| Auth | NavUserPassword | NavUserPassword |
| License popup | Auto-dismissed | Auto-dismissed |
| Protocol version | 15041 | 15041 (identical) |

Both use NavUserPassword authentication (not Windows/NTLM).

### Essential Commands
```bash
cd U:/git/bc-mcp
npx tsc --noEmit                    # Type check
npx vitest run                       # Unit + protocol tests (128 tests)
npx vitest run --config vitest.integration.config.ts  # Integration tests against real BC (103 tests)
npm start                            # HTTP server on port 3000
npm run start:stdio-direct           # Direct stdio for Claude Desktop
```

### Rules
- Use Windows paths with forward slashes in bash
- NEVER use `2>nul` (creates undeletable files on Windows)
- NEVER use emojis -- Windows rendering issues
- Always run `npx tsc --noEmit` after changes
- Run integration tests after any protocol-level change
- ESM project -- use `.js` extensions in all imports

## Protocol Verification Procedure

**CRITICAL: Always verify protocol behavior against decompiled BC source, not v1 code.**

V1 had several incorrect assumptions (per-page connections, SaveValue not echoing, etc.). When implementing or debugging any BC protocol interaction:

1. **Check the decompiled BC source first** at `U:/git/bc-mcp/reference/bc28/decompiled/`
2. Use v1 (`C:\bc4ubuntu\Decompiled\bc-poc\src\`) as a secondary reference only
3. If v1 and decompiled code disagree, trust the decompiled code
4. Document which decompiled file/class confirmed the behavior

Key decompiled assemblies:
- `Microsoft.Dynamics.Framework.UI/` -- Core UI framework (controls, forms, interactions, observers)
- `Microsoft.Dynamics.Framework.UI.Web/` -- Web serialization (ResponseManager, handler types, change serializers)
- `Microsoft.Dynamics.Nav.Service.ClientService/` -- WebSocket server-side handler
- `Microsoft.Dynamics.Nav.Types/` -- BC type system, VersionCompatibility

## Architecture Overview

```
connection/ -> protocol/ -> session/ -> services/ -> operations/ -> mcp/ + api/
```

### Single Connection Per Session
BC supports multiple forms on one WebSocket connection, tracked by `formId` in each interaction and `openFormIds` in each request. Verified from decompiled `UISession.openedForms` dictionary.

The v1 "per-page connection" was a workaround for an `openFormIds` tracking bug, not a BC requirement.

### Event-Driven Protocol
BC sends handler arrays as responses. The EventDecoder transforms these into typed `BCEvent[]`. State is derived from events via `FormProjection` into per-form `FormState`, coordinated by `PageContext`.

### Invoke Queue
All invokes are serialized via a promise queue in `BCSession`. BC's protocol is stateful -- concurrent sends corrupt sequence numbers.

### Session Lifecycle
`SessionManager` (`src/session/session-manager.ts`) owns lazy session creation and dead-session recovery with exponential backoff (1s, 2s, 4s, 8s). Server entry points (`server.ts`, `stdio-server.ts`) use it instead of managing sessions directly. When a dead session is detected, all page contexts are cleared and `SessionLostError` is thrown. `LogicalModalityViolationException` (stale modal state from crashed sessions) is handled with the same retry logic. License/evaluation dialogs are auto-dismissed during session init.

Configurable via env vars: `BC_INVOKE_TIMEOUT` (default 30s), `BC_RECONNECT_MAX_RETRIES` (default 4), `BC_RECONNECT_BASE_DELAY` (default 1s), `BC_PROFILE` (BC profile id e.g. `BUSINESS MANAGER`; empty = server default — see Tell Me Search section).

## BC Protocol Patterns (Verified from Decompiled Source)

### OpenSession Handshake (Required)
Every session starts with an `OpenSession` RPC that returns `ServerSessionId`, `SessionKey`, `CompanyName`. All subsequent `Invoke` calls must include these fields plus `tenantId`, `navigationContext`, `features`, `supportedExtensions`.

Reference: `BCSessionManager.ts` (v1), `NsServiceJsonRpcHostFactory.cs` (decompiled)

### Parameter Case Sensitivity
BC uses case-INSENSITIVE parameter matching. Verified from decompiled `InteractionParameterHelper.TryGetValueIgnoreCase` which uses `StringComparison.OrdinalIgnoreCase`. Both camelCase and PascalCase work.

### Control Paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SShadowS/business-central-mcp](https://github.com/SShadowS/business-central-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
