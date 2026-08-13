---
trigger: always_on
description: Guidance for AI coding agents working on AI Happy Design v2.
---

# AGENTS.md

Guidance for AI coding agents working on AI Happy Design v2.

## Project Goal

A single Go binary + Figma plugin that gives LLMs full Figma canvas access through:
- CLI for direct operations (schema-validated, design-linted)
- WebSocket relay to Figma plugin
- Built-in design intelligence (catalog + design guide)
- Schema system with auto-correction and fuzzy matching

## System Components

### 1) Go Binary
- Entry: `cmd/ai-happy-design/main.go`
- Modes: `mcp` (schema-backed stdio server), `ws` (relay only), `command`, `batch`, `tools`, `schema`, `validate`, `guide`

### 2) Relay Layer
- Server: `internal/ws/server.go`
- Client: `internal/ws/client.go`
- Legacy command routing: `internal/ws/command_routing.go`

### 3) Schema + Validation Layer
- Schema types: `internal/schema/types.go`
- Schema registry: `internal/schema/registry.go`
- Command schemas: `internal/schema/*_schemas.go`
- Validator: `internal/validate/validator.go` (fuzzy matching, named colors, auto-fix)
- Design lint: `internal/designlint/lint.go` (text sizing, contrast, spacing, scoring)
- **LLM catalog (SOURCE OF TRUTH)**: `internal/tools/catalog_llm.go`
- Describe tool: `internal/tools/describe.go`

### 4) Plugin Runtime
- Entry: `plugin/src/main.ts`
- Domain handlers: `plugin/src/handlers/*.ts`
- UI relay client: `plugin/src/ws/client.ts`
- UI: `plugin/src/ui/*`

## Figma Plugin Build Target (CRITICAL)

Figma's plugin sandbox uses QuickJS/WASM. Build target MUST be `es6`.

### Unsupported syntax (causes "Unexpected token" errors):
- `?.` optional chaining
- `??` nullish coalescing
- `{...obj}` object spread
- `for await...of` async iteration
- `?.()` optional call
- `??=`, `||=`, `&&=` logical assignment
- Class fields and private class features

### Required build config:

**esbuild.config.mjs**: `target: 'es6'`

**tsconfig.json**: `"target": "ES6"`, `"lib": ["ES2015", "ES2017"]`

### Post-build verification:
```bash
grep -c '\?\.' dist/code.js    # 0
grep -c '\?\?' dist/code.js    # 0
grep -c '\.\.\.' dist/code.js  # 0
```

### Plugin UI images:
Figma blocks `data:` URIs on `<img>` tags. Use `<div>` with CSS `background-image` instead.

## Protocol Contracts (Do Not Break)

### Response envelope
All responses wrapped: `{"type":"message","channel":"<ch>","message":{"id":"<id>","result":{...}}}`.
Errors: `{"type":"message","channel":"<ch>","message":{"id":"<id>","error":"..."}}`.
Never send bare `{id,error}`.

### Dynamic page access
Use `await figma.getNodeByIdAsync(...)`. Avoid deprecated sync getters.

### Image fill flow
- `set_image_fill_from_url`: try `figma.createImageAsync(url)` → fallback `fetch(url)` → bytes → `figma.createImage(bytes)` → set IMAGE fill
- `set_image_fill`: decode base64/data URL → `figma.createImage(bytes)` → set IMAGE fill

## Channel Resolution Order
1. Positional argument
2. `--channel` flag
3. `AHD_CHANNEL` env var
4. Relay preferred/active channel

## Design Intelligence — Central Source of Truth

### The Rule

**`internal/tools/catalog_llm.go` is the SINGLE source of truth for ALL design rules.** Nothing else should define design rules. Everything else references the catalog.

### What lives in catalog_llm.go:
- Design thinking (CSS-to-Figma, visual hierarchy, design decisions, layer organization)
- Design patterns (coordinates, grid, auto-layout, cards, typography, balance, scaling, aspect ratio, frame positioning)
- Playbook (12-step process)
- Workflow (batch vs single command)

### Discovery endpoints:

| CLI Command | Returns |
|-------------|---------|
| `ahd-figma schema` | List all commands with descriptions |
| `ahd-figma schema <command> --json` | Exact JSON schema for a command |
| `ahd-figma validate` | Dry-run validation (schema + design lint) |
| `ahd-figma guide` | Design intelligence (visual hierarchy, composition, effects) |
| `ahd-figma schema --all` | Full command reference (for llms-full.txt) |
| MCP `tools/list` / `resources/list` | Schema-backed tool and resource discovery |
| MCP `ahd_describe` | LLM catalog or guide content |

### When updating design rules:

1. Edit ONLY `internal/tools/catalog_llm.go`
2. Run `go build ./...` to verify compilation
3. Rebuild binary: `make build && cp bin/ahd-figma ~/bin/ && cp bin/ai-happy-design ~/bin/`
4. Restart relay if running
5. **Do NOT duplicate rules** into SKILL.md, AGENTS.md, or reference files — they all point to the CLI

### What references the catalog (but does NOT define rules):
- **Claude skill** (`~/.claude/skills/ai-happy-design/SKILL.md`) — workflow + "call design_guide for rules"
- **Skill reference files** (`references/design-patterns.md`) — quick offline fallback only
- **README.md** — user-facing overview, links to CLI commands
- **This file (AGENTS.md)** — architecture + development practices

## Build/Test Commands

```bash
make build                              # Go binary
go test ./...                           # Go tests (schema, validate, designlint, tools)
go build ./...                          # Verify compilation
cd plugin && npm run check && cd ..     # Plugin typecheck + build + syntax verification
ahd-figma schema text.create      # Verify schema system
ahd-figma validate                # Verify validation pipeline
```

## Development Practices (Learned)

### When modifying the catalog:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerveband/ai-happy-design](https://github.com/nerveband/ai-happy-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
