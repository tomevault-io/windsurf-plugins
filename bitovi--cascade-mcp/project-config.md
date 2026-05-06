---
trigger: always_on
description: - With all api changes or file changes, keep the documentation in `server/readme.md` up to date.
---

# Jira MCP Auth Bridge - AI Agent Instructions


## Development Standards

- With all api changes or file changes, keep the documentation in `server/readme.md` up to date.

### Security Guidelines - CRITICAL

**NEVER log or expose sensitive credentials:**
- ❌ NEVER log RSA private keys (`GOOGLE_SERVICE_ACCOUNT_PRIVATE_KEY`)
- ❌ NEVER log unencrypted Google service account JSON
- ❌ NEVER log decrypted service account credentials in console output
- ❌ NEVER include private keys or unencrypted credentials in error messages
- ✅ ALWAYS use encrypted credentials (`X-Google-Token` header with `RSA-ENCRYPTED:` prefix)
- ✅ ALWAYS decrypt credentials only when needed and never log the decrypted result
- ✅ ONLY log metadata (like client_email or project_id) if absolutely necessary for debugging

### Code Organization for MCP Tools

**Folder Structure:**
- Complex MCP tools should have their own folder under `server/providers/{provider}/tools/{tool-name}/`
  - Use own folder for: Multi-step workflows, tools with sampling hooks, tools with helper modules
  - Simple single-step tools can remain as single files
- Each tool folder contains:
  - `index.ts` - Exports the tool registration function
  - `{tool-name}.ts` - Main tool implementation file
  - `{helper-name}.ts` - Semi-specific helper modules (parsers, validators, formatters, etc.)

**Main Tool File Structure (`{tool-name}.ts`):**
1. **Top section**: Imports and type definitions
2. **Middle section**: Tool registration function (orchestration only - calls helper functions)
3. **Bottom section**: Step helper functions in execution order

**Helper Function Guidelines:**
- **Semi-specific helpers** (parsers, validators, domain logic) → Separate module files
  - Example: `shell-story-parser.ts` with `parseShellStoriesFromAdf()` and `addCompletionMarkerToShellStory()` functions
  - Definition: Could be used by different workflows, minimal dependencies on external state/parameters
  - Benefits: Testable, reusable, maintainable
  - Export types/interfaces used across modules
  - Export functions for testing: `export function parseShellStoriesFromAdf(...)`, `export function addCompletionMarkerToShellStory(...)`
- **Broad workflow steps** → Exported functions at bottom of main file, in execution order
  - Example: `fetchEpicAndExtractShellStories()`, `findNextUnwrittenStory()`, `validateDependencies()`
  - These orchestrate the tool's main workflow steps
  - Keep main handler clean by delegating to these functions
  - Export for testing: `export async function fetchEpicAndExtractShellStories(...)`
  - Functions should throw descriptive errors (don't return error objects)

**Utility Function Placement:**
- **General utilities** (date formatting, string manipulation) → `server/utils/` (to be created as needed)
- **Provider-specific utilities** (Jira/Atlassian API helpers, Figma helpers, Google helpers) → `server/providers/{provider}/` directory
  - Atlassian object helpers (issue key parsing, ADF manipulation, Jira-specific utilities) → `server/providers/atlassian/`
  - Functions that only operate on Atlassian data structures belong in the Atlassian provider folder
- **Tool-specific but reusable** (parsers, validators) → Separate module in tool folder

**Example Structure:**
```
server/providers/combined/tools/write-next-story/
├── index.ts                    # export { registerWriteNextStoryTool }
├── write-next-story.ts         # Main tool + workflow step functions at bottom
└── shell-story-parser.ts       # Semi-specific helper (parser logic)

server/providers/atlassian/
├── atlassian-api-client.ts     # API client
├── atlassian-helpers.ts        # Jira API helpers
└── jira-issue-helpers.ts       # Issue key parsing, Jira object utilities
```

## Architecture Overview

This is an **OAuth 2.0 bridge server** that enables MCP (Model Context Protocol) clients like VS Code Copilot to access Jira through secure authentication. The system has three main components:

### Dual Interface Pattern (MCP + REST API)

Tools are exposed via **both** MCP protocol and REST API without code duplication:

**Pattern:**
- `server/providers/{provider}/tools/{tool}/core-logic.ts` - Shared business logic (`executeWriteShellStories`, etc.)
- `server/providers/{provider}/tools/{tool}/{tool}.ts` - MCP tool wrapper (uses OAuth context)
- `server/api/{tool}.ts` - REST API wrapper (uses PAT headers)

**New tools should support both interfaces.** Copy existing patterns from `write-shell-stories` or `analyze-feature-scope`.

### Authentication: OAuth vs PAT

| Method | Used By | Auth Header | Token Source |
|--------|---------|-------------|--------------|
| **OAuth** | MCP clients (VS Code Copilot, Claude Desktop) | `Authorization: Bearer <JWT>` | Our PKCE OAuth flow |
| **PAT** | REST API, scripts, server-to-server | `X-Atlassian-Token`, `X-Figma-Token` | User's personal access tokens |

**PKCE Bridge Pattern (MCP OAuth):**
1. MCP client initiates PKCE OAuth with our server (`server/pkce/`)
2. We initiate traditional OAuth with providers (Atlassian, Figma)
3. Provider tokens embedded in JWT returned to MCP client
4. MCP client sends JWT → we extract provider tokens → call provider APIs

### Atlassian API Client Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitovi/cascade-mcp](https://github.com/bitovi/cascade-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
