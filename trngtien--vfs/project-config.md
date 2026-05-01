---
trigger: always_on
description: Code search strategy: vfs for navigation, Grep/Read for understanding. Combine both to avoid hallucination.
---


# Code Search Strategy: Navigate with vfs, Understand with Grep/Read

> vfs is a **navigation tool** (find where things are), not an **understanding tool** (know how things work).
> Signatures without bodies create false confidence. Always read implementation before claiming to understand behavior.

## How vfs Works

vfs parses source files via AST and returns **exported signatures with bodies stripped**. It supports Go, JS, TS, Python, Rust, Java, C#, Dart, Kotlin, Swift, Ruby, Solidity, HCL, Dockerfile, Protobuf, SQL, and YAML.

**What vfs gives you:** `internal/services/fare.go:42: func CalculateFare(req *FareRequest) (*FareResponse, error)`
**What vfs hides:** The 50 lines of implementation inside that function.

This makes vfs excellent for **locating** definitions, but dangerous for **understanding** behavior.

## Step 1: Classify Your Intent

Before searching, determine what you need:

| Intent | Description | Primary tool | Depth required |
|--------|-------------|-------------|----------------|
| **Locate** | "Which file defines X?" | vfs | Signature only — no Read needed |
| **Understand** | "How does X work?" | vfs → Read body + context | Full implementation + dependencies |
| **Modify** | "Change how X behaves" | vfs → Read body + callers | Full implementation + Grep for usages |
| **Debug** | "Why does X fail?" | Grep + Read | Bodies, callers, error paths — vfs alone is useless here |

## Step 2: Search (vfs for navigation)

Use vfs as the **first step** to locate definitions — not as the final answer.

### Access Priority: MCP First, CLI Fallback

MCP runs on the host outside the sandbox and bypasses binary restrictions.

```
Step,Name,Action,Condition,Result
1,MCP,"CallMcpTool(server: user-user-vfs, toolName: search, ...)",success,Use MCP for all vfs operations
1,MCP,"CallMcpTool(server: user-user-vfs, toolName: search, ...)",failure/error,Go to step 2
2,CLI fallback,"run `command -v vfs` in Shell",found,"Use `vfs <path> -f <pattern>` via Shell"
2,CLI fallback,"run `command -v vfs` in Shell",not found,Go to step 3
3,Grep/Read fallback,"Fall back to Grep/Read silently",,Notify user once per session if needed; Do NOT block progress
```

### MCP Calls (preferred)

Server name: **`user-user-vfs`**

> **CRITICAL: MCP calls MUST use absolute paths.** MCP runs on the host and does NOT share the agent's working directory. Always use the full workspace path from `<user_info>`.

```
# Find definitions by name
CallMcpTool(server: "user-user-vfs", toolName: "search", arguments: { "paths": ["/absolute/path/to/workspace"], "pattern": "HandleLogin" })

# List all exports from a directory
CallMcpTool(server: "user-user-vfs", toolName: "extract", arguments: { "paths": ["/absolute/path/to/workspace/internal/handlers"] })
```

### CLI Calls (fallback only)

```bash
vfs <path> -f <pattern>           # filter signatures (case-insensitive)
vfs .                             # all exported sigs in current project
vfs ./internal ./pkg              # scan specific directories
```

## Step 3: Read with Sufficient Depth (Anti-Hallucination)

> **NEVER assume you understand a function's behavior from its signature alone.**
> A signature is an address, not a description. You must read the body before making claims.

After vfs locates a signature, determine how much to read based on your intent:

### Locate intent — signature is enough
```
vfs search → found fare.go:42: func CalculateFare(...)
Answer: "CalculateFare is defined in internal/services/fare.go at line 42."
Done. No Read needed.
```

### Understand intent — read the full function + surrounding context
```
vfs search → found fare.go:42: func CalculateFare(...)

Read: fare.go L1-20    (imports + package-level vars — reveals dependencies)
Read: fare.go L42-90   (the full function body — reveals actual behavior)
```

**Why read imports/package-level context:** A function that imports `"encoding/csv"` behaves very differently from one that imports `"net/http"`. The signature won't tell you this.

**Minimum read range:** For any function body, read at least:
- The complete function (not just the first 10 lines — logic often lives at the end)
- Package-level variables and init() if they exist (first 20-30 lines of file)
- Types referenced in the signature if they're in the same package

### Modify intent — read body + find all callers
```
vfs search → found fare.go:42: func CalculateFare(...)

Read: fare.go L1-20    (imports)
Read: fare.go L42-90   (full body)
Grep: "CalculateFare" across the codebase  (find all callers before changing)
```

### Debug intent — skip vfs, start with Grep/Read
When debugging, you need to follow execution flow through bodies. vfs strips the information you need most. Start with Grep for error messages, log strings, or the failing function name, then Read the relevant bodies.

## When to Skip vfs Entirely

Use Grep/Read directly when:

1. **You already know the exact file and line** — just Read it.
2. **Searching inside function bodies** — string literals, config keys, error messages, log strings.
3. **Non-code files** — JSON, CSS, Markdown, `.env`.
4. **The user gave you a file path** — e.g. "look at line 50 of client.go".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrNgTien/vfs](https://github.com/TrNgTien/vfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
