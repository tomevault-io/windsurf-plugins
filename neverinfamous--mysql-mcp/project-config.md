---
trigger: always_on
description: Ensure PRs adhere to these SSoT architectural rules:
---

# mysql-mcp — Copilot Code Review Context

## Enforce Core Architecture Rules
Ensure PRs adhere to these SSoT architectural rules:
- **Tool filtering** skips MySQL connections when using exclusively router, proxysql, or shell tools.
- **Code Mode** (`mysql_execute_code`) significantly reduces token usage.
- **Cache**: `METADATA_CACHE_TTL_MS` controls the cache TTL (default 30000).
- **Transports**: Supports `stdio` and streamable `http` (MCP v2 stateless architecture via `NodeStreamableHTTPServerTransport`).
- **Validation**: Ensure parameter alias validation at the MCP boundary (via the Dual-Schema Pattern).
- **Authentication**: Secure connections with Bearer Tokens or OAuth.
- **Features**: Tool filtering, token logging, and ecosystem integrations for MySQL Router, ProxySQL, and Shell.

## Review Session Context

Read `memory://briefing/mysql-mcp` from `memory-journal-mcp` before starting. This provides essential real-time context:

- **Recent journal entries** — what was just worked on by the development agent
- **GitHub status** — open issues, PRs, CI status, milestones
- **Workflow runs** — recent CI/CD results
- **Copilot review summaries** — your own recent review findings

For detailed session handoff context, search for entries tagged `session-summary`. These contain end-of-session notes from the development agent.

Log review issues using `mj_execute_code`. Agents review these findings during their next session briefings.

## Uphold Coding Standards

### Standardize File Naming

- **Files and folders**: Always kebab-case (`schema-manager.ts`, `tool-filter.ts`)
  - **Exception**: `.github` repository templates may use `snake_case`.
- **Never** PascalCase or camelCase for filenames

### Maintain Modularity

- **File size limit**: Source files stay under ~500 lines
- **Split pattern**: `foo.ts` → `foo/` directory with sub-modules + `foo/index.ts` barrel re-export
- **Logical grouping**: Split by functional cohesion, not arbitrary line counts

### Enforce Type Safety

- **Strict TypeScript** — `tsconfig.json` enforces strict mode
- **Never use `eslint-disable`** to evade standards
- **Never use `any`** — use `unknown` and narrow with type guards
- Type assertions (e.g., `as`) — always forbidden (use `satisfies` or strict type guards instead)
- **Never use `@ts-ignore` or `@ts-expect-error`** — fix the underlying type issue
- **Zod schemas** for all tool input validation at system boundaries
- **Union types over enums** — use `type Status = "active" | "inactive"` instead of `enum`

### Implement Error Handling

All tool handlers return structured error responses — never raw exceptions:

```typescript
{
  success: false,
  error: string,          // Human-readable message
  code: string,           // Module-prefixed code (e.g., "QUERY_ERROR")
  category: ErrorCategory,// Error category (validation, connection, query, etc.)
  suggestion?: string,    // Optional actionable fix for the agent
  recoverable: boolean,   // true = user can fix, false = server error
  details?: unknown,      // Optional error details
  metrics?: unknown       // Optional error metrics
}
```

> **Note**: Table-querying tools must return `{exists: false, table}` for nonexistent tables. All schema examples must reflect the comprehensive toolset and current config flags.
> **Anti-Hallucination**: Do not assume existence of tools, resources, or prompts. They must be explicitly listed in the tool-reference or registered in `server/`.

## Understand Architecture

```
scripts/                        # Instruction and infrastructure scripts
src/
├── cli.ts                      # CLI entry point (util.parseArgs)
├── index.ts                    # Library entry point
├── version.ts                  # Version export
├── __tests__/                  # Unit and E2E tests
├── adapters/                   # MySQL database adapters
├── audit/                      # Audit and token logging
├── auth/                       # OAuth authentication
├── cli/                        # CLI argument parsing modules
├── codemode/                   # Sandboxed JS execution engine
├── constants/                  # Server instructions, config
├── filtering/                  # Tool filtering (groups, meta-groups)
├── logging/                    # Structured logging
├── observability/              # Observability and metrics
├── pool/                       # Connection pool management
├── progress/                   # Progress notification helpers
├── server/                     # MCP server setup and registration
├── transports/                 # Streamable HTTP transport layer
├── types/                      # Type definitions + barrel exports
└── utils/                      # Logger, error helpers, utilities
```

## Consult Reference Files

| File                            | Purpose                             |
| ------------------------------- | ----------------------------------- |
| `README.md`                     | Primary project documentation       |
| `AGENT_README.md`               | Root AI agent specific instructions |
| `skills/AGENT_README.md`        | AI agent specific instructions      |
| `test-server/code-map.md`       | File → tool/handler mapping         |
| `test-server/tool-reference.md` | Categorized tool inventory          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neverinfamous/mysql-mcp](https://github.com/neverinfamous/mysql-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
