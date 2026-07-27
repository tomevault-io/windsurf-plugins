---
trigger: always_on
description: **THIS PROJECT USES SVELTE 5 WITH RUNES. YOU MUST ALWAYS:**
---

# tinykit - Claude Code Documentation

## ⚠️ CRITICAL: SVELTE 5 ONLY

**THIS PROJECT USES SVELTE 5 WITH RUNES. YOU MUST ALWAYS:**

1. **Use `$state()` for reactive variables** - NOT `let foo = 'bar'`

   ```svelte
   ❌ WRONG: let count = 0
   ✅ RIGHT: let count = $state(0)
   ```

2. **Use `$props()` for component props** - NOT `export let`

   ```svelte
   ❌ WRONG: export let title = ''
   ✅ RIGHT: let { title = '' } = $props()
   ```

3. **Use `watch` from runed for watching prop changes** - NOT `$effect()`

   ```svelte
   ❌ WRONG: $effect(() => { if (target_field) doSomething() })
   ✅ RIGHT: watch(() => target_field, (value) => { if (value) doSomething() })
   ```

   - Import: `import { watch } from "runed"`
   - Use `watch` when reacting to specific prop/state changes
   - Use `$effect()` only for setup/cleanup side effects (e.g., event listeners)

4. **Use `$derived()` for computed values** - NOT `$:` assignments

   ```svelte
   ❌ WRONG: $: doubled = count * 2
   ✅ RIGHT: let doubled = $derived(count * 2)
   ```

5. **Use `onclick={handler}` NOT `on:click={handler}`**

   ```svelte
   ❌ WRONG: <button on:click={handler}>Click</button>
   ✅ RIGHT: <button onclick={handler}>Click</button>
   ```

6. **BEFORE writing ANY Svelte component:**
   - Use `mcp__svelte__svelte-autofixer` tool to validate your code
   - Pass `desired_svelte_version: 5` and `async: false` (unless using await in markup)
   - Fix ALL issues the autofixer reports before proceeding

**No exceptions. Svelte 4 syntax will break this codebase.**

## Project Overview

tinykit is an open-source, self-hosted AI development platform—think Lovable, but running on your own server alongside the apps it builds. It provides database, editor, and hosting all at `/tinykit`, enabling you to ship production CRUD apps in an afternoon.

**Core Value Proposition:**
Let the AI handle the boring 80% (routes, auth, CRUD operations, basic UI), then you take over for the last mile—the details that actually matter.

## Tech Stack

### Core Technologies

- **Frontend Framework**: SvelteKit 2.x with TypeScript
- **Database & Storage**: Pocketbase 0.23.8
- **Styling**: Tailwind CSS 3.4.x
- **Icons**: Lucide (via lucide-svelte)
- **Editor**: CodeMirror 6 with Emmet support
- **AI Integration**: Vercel AI SDK (@ai-sdk/openai, @ai-sdk/anthropic, @ai-sdk/google)
- **Version Control**: Snapshots stored in Pocketbase (code-only)

### Architecture

- Self-hosted, single-server deployment
- No external services required (except LLM API)
- **Two Pocketbase collections:**
  - `_tk_projects` - All project data
  - `_tk_settings` - App configuration (LLM keys, etc.)
- Pocketbase proxied through SvelteKit at `/_pb/` (same port)
- Live preview with in-browser Svelte compilation
- Server-side build system for production HTML

### `_tk_projects` Collection Schema

```
- id (auto)
- name (text)
- domain (text, unique, required)
- frontend_code (text, max 10MB)
- backend_code (text, max 10MB)
- custom_instructions (text, max 1MB)
- design (json) - CSS variables array
- content (json) - Content fields array
- snapshots (json) - Time travel history
- agent_chat (json) - Conversation history
- data (json) - App data tables
- settings (json) - Project settings (vibe_zone_enabled, etc.)
- published_html (file) - Built production HTML
- created/updated (auto)
```

### `_tk_settings` Collection Schema

```
- id (text, primary key) - Setting key (e.g., "llm")
- value (json) - Setting value
```

### Domain-Based Routing

Each project is associated with a domain. Users can point multiple domains to a single tinykit server:

```
calculator.myserver.com/                → Serves calculator production app
calculator.myserver.com/tinykit/studio  → Edit calculator app
calculator.myserver.com/tinykit/dashboard → See ALL apps

blog.myserver.com/                      → Serves blog production app
blog.myserver.com/tinykit/studio        → Edit blog app
```

**Route Resolution:**

- `/` - Serve production app for current domain (from `published_html` file)
- `/tinykit/studio` - Edit the project for current domain
- `/tinykit/studio?id=X` - Edit specific project by ID
- `/tinykit/dashboard` - List all projects
- `/tinykit/new?domain=X` - Create new project for domain
- `/tinykit/settings` - LLM configuration
- `/setup` - First-time setup wizard

### Current UI Layout

```
┌───────────────────────────────────────────────────────────┐
│ tinykit · My Project                    [Vibe] [Deploy]   │
├────────────────────────────┬──────────────────────────────┤
│ [Agent] [Code] [Content]   │ Preview                      │
│ [Design] [Data] [History]  │                              │
├────────────────────────────┤                              │
│                            │                              │
│  Left Pane Content         │  Live Preview                │
│  (tabs switchable)         │                              │
│                            │                              │
└────────────────────────────┴──────────────────────────────┘
```

- **Left Pane**: 6 tabs
  - **Agent** (Cmd+1): AI chat interface for building apps
  - **Code** (Cmd+2): CodeMirror editor for manual code editing
  - **Content** (Cmd+3): CMS-like content fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinykit-studio/tinykit](https://github.com/tinykit-studio/tinykit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
