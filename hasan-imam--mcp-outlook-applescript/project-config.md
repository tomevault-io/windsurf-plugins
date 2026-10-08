---
trigger: always_on
description: MCP server for Microsoft Outlook on Mac. 49 tools for mail, calendar, contacts, tasks, and notes via AppleScript. TypeScript with strict mode, Zod validation, and security-hardened input escaping.
---

# mcp-outlook-applescript

MCP server for Microsoft Outlook on Mac. 49 tools for mail, calendar, contacts, tasks, and notes via AppleScript. TypeScript with strict mode, Zod validation, and security-hardened input escaping.

## Commands

| Command | Purpose |
|---------|---------|
| `npm run build` | Compile TypeScript to `dist/` and set shebang |
| `npm run typecheck` | Type-check without emitting |
| `npm test` | Run 121 unit tests (vitest, no Outlook needed) |
| `bash scripts/audit.sh` | 28 static quality checks (build, security, package, functional) |
| `node scripts/doc-quality.mjs` | Score all 49 tool descriptions against 6-criterion rubric |
| `node scripts/smoke-test.mjs` | Live integration tests (requires Outlook running) |

## Testing

| When | Run | What it checks |
|------|-----|----------------|
| Any code change | `npm run build && npm test && bash scripts/audit.sh` | Types, 121 unit tests, 28 static checks (security, package, bug-fix preservation) |
| Before release, or after changing AppleScript/repository/parser | `node scripts/smoke-test.mjs` (Outlook must be running) | Live integration: 45/49 tools via JSON-RPC against real Outlook |
| After changing tool descriptions or Zod schemas | Give an MCP client a task without naming tools — it must discover and call them from descriptions | Tool discoverability, schema usability, response format |

## Architecture

```
src/                                 # 34 TypeScript files
├── index.ts                         # MCP server entry point, tool dispatch
├── applescript/                     # AppleScript backend
│   ├── executor.ts                  #   osascript runner, escapeForAppleScript()
│   ├── scripts.ts                   #   AppleScript templates (pure functions)
│   ├── parser.ts                    #   delimiter-based output parser
│   ├── repository.ts                #   IRepository implementation
│   ├── account-repository.ts        #   account enumeration
│   ├── account-scripts.ts           #   account-related AppleScript
│   ├── calendar-manager.ts          #   event update/delete/respond
│   ├── calendar-writer.ts           #   event creation
│   ├── content-readers.ts           #   single-item content readers
│   ├── mail-sender.ts               #   send email with attachments
│   └── index.ts                     #   barrel exports
├── approval/                        # Two-step prepare/confirm for destructive ops
│   ├── token-manager.ts             #   single-use token store (5-min TTL)
│   ├── hash.ts                      #   content hashing for change detection
│   └── types.ts                     #   ApprovalToken, OperationType, TargetType
├── database/
│   └── repository.ts                # IRepository + IWriteableRepository interfaces
├── parsers/
│   └── html-stripper.ts             # HTML to plain text
├── tools/                           # Zod input schemas + tool logic
│   ├── mail.ts                      #   mail read tools + send_email
│   ├── calendar.ts                  #   calendar tools
│   ├── contacts.ts                  #   contact tools
│   ├── tasks.ts                     #   task tools
│   ├── notes.ts                     #   note tools
│   ├── mailbox-organization.ts      #   destructive ops + low-risk mutations
│   └── index.ts                     #   barrel exports
├── types/                           # Domain types + pagination envelope
│   └── pagination.ts                #   PaginatedResult<T>, paginate()
└── utils/
    ├── dates.ts                     # Apple epoch (2001) <-> ISO conversion
    └── errors.ts                    # ErrorCode enum, OutlookMcpError hierarchy
```

### Data flow

1. `src/index.ts` registers 49 tools on the MCP server with Zod schemas
2. Each handler calls a tool class (`MailTools`, `CalendarTools`, etc.) which calls `IRepository`
3. `IRepository` (in `applescript/repository.ts`) calls `scripts.ts` to build an AppleScript string
4. `executor.ts` runs it via `osascript` (stdin, not shell) and returns raw output
5. `parser.ts` splits output on `{{RECORD}}` / `{{FIELD}}` / `{{=}}` delimiters into typed rows
6. Tool class maps rows to domain types and wraps in `paginate()` for list/search results

### Lazy initialization

The server does not connect to Outlook on startup. The first tool call triggers `initializeAppleScriptBackend()`, which checks `isOutlookRunning()` and creates all repositories and tool instances.

## Key conventions

### Zod strict schemas

Every tool uses `z.strictObject()` with `.describe()` on every field. No `as any`, no `args as {...}` casts. Schemas live in `src/tools/<category>.ts` and are re-exported through `src/tools/index.ts`.

### Input escaping

All user-provided strings interpolated into AppleScript templates must pass through `escapeForAppleScript()` in `executor.ts`. This escapes backslashes, double quotes, and converts newlines to AppleScript `linefeed`/`return` concatenation. The audit script verifies this for `sendEmail()` specifically (addresses, replyTo, attachment paths, contentId).

### Pagination envelope


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hasan-imam/mcp-outlook-applescript](https://github.com/hasan-imam/mcp-outlook-applescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
