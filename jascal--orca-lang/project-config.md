---
trigger: always_on
description: This guide is for engineers integrating Orca into an agent framework (Claude tool use, OpenAI function calling, AutoGen, OpenClaw, or a custom tool loop). It covers installation, the canonical generation loop, working with strings instead of files, multi-machine workflows, and extending the runtimes.
---

# Orca — Agent Integration Guide

This guide is for engineers integrating Orca into an agent framework (Claude tool use, OpenAI function calling, AutoGen, OpenClaw, or a custom tool loop). It covers installation, the canonical generation loop, working with strings instead of files, multi-machine workflows, and extending the runtimes.

---

## What Orca Provides

Orca is a **state machine language** designed as an LLM code generation target. It separates program topology (state machine structure, verified by static analysis) from computation (action functions, implemented in your language of choice).

The agent-facing interface is a set of tools:

| Tool | What it does |
|------|-------------|
| `parse_machine` | Parse `.orca.md` source → structured JSON (states, events, transitions, guards, actions, context) |
| `verify_machine` | Structurally verify a machine → errors and warnings with codes and suggestions |
| `compile_machine` | Compile to XState v5 TypeScript config or Mermaid `stateDiagram-v2` |
| `generate_machine` | Generate an Orca machine from a natural language spec (requires LLM) |
| `generate_multi_machine` | Generate a coordinated multi-machine system from a natural language spec (requires LLM) |
| `generate_actions` | Generate action scaffold code in TypeScript, Python, or Go |
| `refine_machine` | Fix verification errors using an LLM, looping until valid or `max_iterations` reached |

All tools accept `source: string` — raw `.orca.md` content. No files required.

---

## Installation

### MCP Server (recommended for modern agent frameworks)

```bash
npm install -g @orcalang/orca-mcp-server

# Or run directly with npx
npx @orcalang/orca-mcp-server
```

Configure in your MCP host (e.g. Claude Code `settings.json` or project `.mcp.json`):

```json
{
  "mcpServers": {
    "orca": {
      "command": "npx",
      "args": ["-y", "@orcalang/orca-mcp-server"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

`${VAR}` syntax passes your shell's `ANTHROPIC_API_KEY` environment variable into the MCP server process. Make sure it is set before starting Claude Code.

The server speaks the [Model Context Protocol](https://modelcontextprotocol.io) over stdio. It exposes all 7 tools with full JSON schemas. The server requires **Node.js 20+** and exits with a clear error on older versions.

### CLI (for agent frameworks that use subprocesses)

```bash
npm install -g orca
```

Discover tools programmatically:

```bash
orca --tools --json   # returns JSON array of tool definitions with input schemas
```

Use stdin to avoid writing temp files:

```bash
echo "$orca_source" | orca verify --stdin
echo "$orca_source" | orca /parse-machine --stdin
echo "$orca_source" | orca compile xstate --stdin
```

All commands support `--json` for structured output:

```bash
orca verify --json machine.orca.md
orca /verify-orca machine.orca.md        # always returns JSON
```

### TypeScript library

```bash
npm install @orcalang/orca-lang
```

```typescript
import { verifySkill, compileSkill, parseSkill } from 'orca-lang/skills';

const result = await verifySkill({ source: orcaSource });
```

### Python runtime

```bash
pip install orca-runtime-python
```

### Go runtime

```bash
go get github.com/jascal/orca-lang/packages/runtime-go
```

---

## The Generation Loop

The canonical pattern for an agent generating and deploying a state machine:

```
1. generate_machine(spec)  →  orca source
2. verify_machine(source)  →  errors?
3.   if errors: refine_machine(source, errors)  →  corrected source  →  go to 2
4. compile_machine(source, 'xstate')  →  TypeScript config
5. generate_actions(source, lang)     →  scaffold code
6. Hand off config + scaffold to the developer
```

In pseudocode (tool-call style):

```python
# Step 1: Generate
result = generate_machine(spec="A payment processor with retry logic, up to 3 attempts")
source = result["orca"]

# Step 2-3: Verify and refine
for i in range(3):
    verification = verify_machine(source=source)
    if verification["status"] == "valid":
        break
    result = refine_machine(source=source, errors=verification["errors"])
    source = result["corrected"]

# Step 4: Compile
xstate = compile_machine(source=source, target="xstate")

# Step 5: Scaffold
scaffolds = generate_actions(source=source, lang="typescript")
```

`generate_machine` and `refine_machine` already implement this loop internally (up to `max_iterations`). You can call them directly and check the returned `status`:

- `"success"` — machine is valid, use `orca`
- `"requires_refinement"` — still has errors after all iterations; inspect `errors` and decide whether to retry or hand off for manual review
- `"error"` — LLM or parse error; inspect `error` message

---

## Handling LLM Auth

| Tool | Calls LLM | Required Key |
|------|-----------|--------------|
| `parse_machine` | No | None |
| `verify_machine` | No | None |
| `compile_machine` | No | None |
| `generate_machine` | Yes | See providers below |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jascal/orca-lang](https://github.com/jascal/orca-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
