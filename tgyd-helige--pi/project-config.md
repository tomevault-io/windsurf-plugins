---
trigger: always_on
description: This monorepo contains Pi extensions (`packages/pi-*`), a shared library (`packages/shared`), and integration tests (`tests/`). Most packages are **Pi extensions** — TypeScript modules that hook into the Pi agent runtime.
---

# Pi Extensions Monorepo — Agent Guidelines

## Project Overview

This monorepo contains Pi extensions (`packages/pi-*`), a shared library (`packages/shared`), and integration tests (`tests/`). Most packages are **Pi extensions** — TypeScript modules that hook into the Pi agent runtime.

---

## Pi Extension Fundamentals

> Full docs: https://pi.dev/docs/latest/extensions

### Entry Point

Every extension exports a default factory receiving `ExtensionAPI`:

```typescript
import type { ExtensionAPI } from '@earendil-works/pi-coding-agent';

export default function (pi: ExtensionAPI) {
  pi.on('session_start', async (event, ctx) => { /* ... */ });
  pi.registerTool({ /* ... */ });
  pi.registerCommand('name', { /* ... */ });
}
```

### Lifecycle

Events grouped by phase:

**Session-level:**
1. `project_trust`
2. `session_start`
3. `resources_discover`

**Agent-level (per user prompt):**
4. `input`
5. `before_agent_start`
6. `agent_start`

**Per-turn loop** (repeats until agent is done):
- `turn_start`
- `context`
- `before_provider_request`
- tool calls (may be multiple per turn)
- `turn_end`

**Agent completion:**
7. `agent_end`

**Shutdown:**
8. `session_shutdown`

Session transitions (`/new`, `/resume`, `/fork`) emit `session_before_switch`/`session_before_fork`, then `session_shutdown`, then `session_start` on the new session.

Note: `agent_end` fires after each user prompt completes; `session_shutdown` only fires on session exit or transition — not between prompts in the same session.

### Key APIs

| API | Purpose |
|-----|---------|
| `pi.on(event, handler)` | Subscribe to lifecycle events |
| `pi.registerTool({...})` | Register a tool callable by the LLM |
| `pi.registerCommand(name, {handler, ...})` | Register a `/slash` command |
| `pi.registerShortcut(...)` | Keyboard shortcuts |
| `pi.registerProvider(...)` | Model providers |
| `pi.appendEntry(key, data)` | Persist state in session (not sent to LLM) |

### Tool Registration

```typescript
pi.registerTool({
  name: 'my_tool',
  label: 'My Tool',
  description: 'What this tool does',
  parameters: Type.Object({ /* typebox schema */ }),
  // Use StringEnum from @earendil-works/pi-ai for string enums (Google compatibility)
  promptSnippet: 'One-line for Available tools list',
  promptGuidelines: ['Use my_tool when...'],
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    return { content: [{ type: 'text', text: 'result' }], details: {} };
  },
});

// On expected error (config missing, invalid input, etc.):
// return { isError: true, content: [{ type: 'text', text: 'sanitized message' }], details: {} };
```

**Tool output must be bounded** — tool results flow into `pi-coding-agent`, which truncates at `DEFAULT_MAX_BYTES` (50KB) and `DEFAULT_MAX_LINES` (2000). Extensions should proactively keep outputs below these limits rather than rely on runtime truncation.

Truncation priority:
1. If the package already has a local `truncateText` or similar helper, use it (keeps the package internally consistent)
2. Otherwise import `truncateHead`/`truncateTail` from `@earendil-works/pi-coding-agent` (v0.74+)
3. Do not pull in external deps just for truncation

What to truncate:
- Large strings, large arrays, and external response bodies in both `content` and `details`; preserve small structured metadata as-is
- Error messages — a stack trace can easily exceed limits

### ExtensionContext (`ctx`)

Every handler receives `ctx` (type: `ExtensionContext` from `@earendil-works/pi-coding-agent`):

| Field | Purpose |
|-------|---------|
| `ctx.ui` | User interaction: confirm, select, input, notify, setStatus |
| `ctx.cwd` | Current working directory |
| `ctx.mode` | `'tui'` \| `'rpc'` \| `'json'` \| `'print'` |
| `ctx.hasUI` | Boolean — guard dialog methods with this |
| `ctx.signal` | AbortSignal for cancellation — long-running tool calls must propagate this to fetch/child processes; the runtime cancels via this signal |
| `ctx.modelRegistry` | Model access and lookup |
| `ctx.sessionManager` | Read-only session state |
| `ctx.getContextUsage()` | Token usage info |
| `ctx.compact()` | Trigger compaction |
| `ctx.shutdown()` | Graceful exit |

---

## Pi Packages

> Full docs: https://pi.dev/docs/latest/packages

Packages bundle extensions + skills + prompts + themes. Manifest in `package.json`:

```json
{
  "pi": {
    "extensions": ["./dist/index.js"],
    "skills": ["./skills"]
  }
}
```

Only include `skills` when the package actually ships skill files. See the Adding a New Package checklist below for the full setup.

Core Pi packages (`@earendil-works/pi-ai`, `pi-agent-core`, `pi-coding-agent`, `pi-tui`, `typebox`) must go in `peerDependencies` — never bundle these. Version ranges should follow the existing convention in this repo (e.g. `">=0.74.0"`); check a sibling package for the current pattern before adding.

---

## Preview Images

Pi extension packages should ship a package-root `preview.png` (for example, `packages/pi-foo/preview.png`) and reference it from `package.json` via `pi.image` when the package is intended to appear in extension/package listings.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TGYD-helige/pi](https://github.com/TGYD-helige/pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
