---
trigger: always_on
description: VS Code extension: MCP server exposing DAP (Debug Adapter Protocol) for
---

# Debugssy - LLM Context

VS Code extension: MCP server exposing DAP (Debug Adapter Protocol) for
AI-assisted debugging. Flow: AI Assistant → MCP Protocol → Debugssy → VS Code
Debug API → Debugger

## Architecture

```
extension.ts → MCPServer.ts → [ToolRouter, PromptHandler, CompletionProvider, ResourceProvider]
                    ↓
              security/ (McpRequestValidator, ExpressionValidator)
                    ↓
              tools/ (Breakpoints, Inspection, DebugControl)
                    ↓
              dap/Client.ts (DAP message interception, state tracking)
```

## File Map

| File                                  | Purpose                                                                 | Modify When                              |
| ------------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------- |
| `extension.ts`                        | Entry point, lifecycle, commands, MCP auto-discovery (VS Code + Cursor) | Adding VS Code commands or MCP discovery |
| `types/cursor.d.ts`                   | Type augmentation for Cursor's `vscode.cursor.mcp` API                  | Cursor MCP API changes                   |
| `MCPServer.ts`                        | HTTP server, MCP SDK, session mgmt                                      | Adding MCP capabilities                  |
| `Config.ts`                           | Zod-validated settings                                                  | Adding config options                    |
| `constants.ts`                        | All magic numbers                                                       | Adding limits/defaults                   |
| `routing/ToolRouter.ts`               | Tool routing, validation                                                | Adding/modifying tools                   |
| `routing/PromptHandler.ts`            | MCP prompts                                                             | Adding prompts                           |
| `routing/CompletionProvider.ts`       | Autocomplete                                                            | Adding completions                       |
| `routing/schemas/*.ts`                | Tool JSON schemas                                                       | Adding tool parameters                   |
| `routing/types/toolArguments.ts`      | Zod validators + TS types                                               | Adding tool arg types                    |
| `security/ExpressionValidator.ts`     | Expression safety                                                       | Adding language support                  |
| `security/expression/validators/*.ts` | Per-language validators                                                 | Language-specific rules                  |
| `tools/Breakpoints.ts`                | Breakpoint ops                                                          | Breakpoint features                      |
| `tools/Inspection.ts`                 | Variable/stack inspection                                               | Inspection features                      |
| `tools/DebugControl.ts`               | Start/stop/step                                                         | Execution control                        |
| `dap/Client.ts`                       | DAP interception, state                                                 | DAP event handling                       |
| `errors/index.ts`                     | Custom error types with codes                                           | Adding new error types                   |

## File Dependencies

When modifying, also update:

- `routing/schemas/*.ts` → `routing/types/toolArguments.ts` (add Zod validator)
- `routing/schemas/*.ts` → `routing/schemas/index.ts` (export)
- `tools/*.ts` → `routing/ToolRouter.ts` (register handler)
- `Config.ts` → `package.json` contributes.configuration (schema match)
- `constants.ts` → `Config.ts` (uses constants for validation)
- `security/expression/validators/*.ts` → `ExpressionValidator.ts` (import +
  routing)

## Key Concepts

**Automation Modes:**

- `assisted`: User controls execution (F5, step), AI sets breakpoints + inspects
- `full`: AI controls everything including start/stop/continue

**Tool availability:** `ToolRouter.getToolSchemas()` returns different tools
based on `automationLevel`.

**Expression validation levels:** strict > moderate > permissive > disabled

- Risk levels: critical (system ops) > high (mutations) > medium (unknown
  funcs) > low (getters)

**Expression validation checks (in order):**

1. Comment injection (block `/* */`, line `//`, `#`)
2. Critical operations (fs, process, network - language-specific)
3. Prototype chain (`__proto__`, `.constructor`, `setPrototypeOf`)
4. Global access (`globalThis`, `window`, `global`, `self`)
5. String obfuscation (`fromCharCode`, `atob`, `Buffer.from`)
6. Meta-programming (`Proxy`, `Reflect`, `defineProperty`)
7. Language-specific validation (mutations, eval, etc.)
8. Generic pattern validation (assignments, increments, etc.)

**DAP states:** `not_started` | `running` | `paused` | `terminated`

**MCP auto-discovery:** The extension registers its HTTP server with the host

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmaynez/debugssy](https://github.com/gmaynez/debugssy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
