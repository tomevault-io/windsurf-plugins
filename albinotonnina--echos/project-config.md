---
trigger: always_on
description: **NEVER make code changes directly on `main`. ALWAYS work in a git worktree.**
---

# Copilot Instructions for EchOS

## ⛔ STOP — READ THIS FIRST BEFORE TOUCHING ANY FILE ⛔

### Git Worktrees Are MANDATORY. No Exceptions.

**NEVER make code changes directly on `main`. ALWAYS work in a git worktree.**

Before writing a single line of code, creating a file, or running any build command:

```bash
# Replace FEATURE_NAME with your actual feature name (e.g. 'auth-fixes')
# 1. Create a worktree (from the main repo root)
git worktree add ../echos-FEATURE_NAME -b feature/FEATURE_NAME

# 2. Move into it — ALL work happens here
cd ../echos-FEATURE_NAME

# 3. When done and merged, clean up
git worktree remove ../echos-FEATURE_NAME
```

Rules:
- Worktrees live as **siblings** of the main repo: `../echos-FEATURE_NAME`
- Branch naming: `feature/FEATURE_NAME`, `fix/FEATURE_NAME`, `chore/FEATURE_NAME`
- If you are already inside a sibling worktree directory (e.g. `../echos-FEATURE_NAME`) and `git worktree list` shows it as a worktree, proceed
- If you are in the original repo directory (e.g. `echos/`, often on `main`) and have not created a worktree yet — **stop and create one now**
- This rule applies to every task: features, bug fixes, typo corrections, documentation edits — everything

**Skipping worktrees is not a shortcut. It is a mistake.**

---

## Project Overview

EchOS is a secure, self-hosted, agent-driven personal knowledge management system. It uses an LLM agent with tools (not rigid command routing) to interact naturally across Telegram, Web, and CLI interfaces.

**Key Principle**: Security-first. Every feature must consider security implications.

## Tech Stack (Do Not Change Without Discussion)

- **Runtime**: Node.js 20+ with TypeScript (strict mode, ESM)
- **Package Manager**: pnpm workspaces (monorepo)
- **Agent Framework**: pi-mono (pi-agent-core + pi-ai)
- **Telegram**: grammY
- **Queue**: BullMQ with Redis
- **Vector DB**: LanceDB (embedded, no server)
- **Metadata DB**: SQLite (better-sqlite3) with FTS5
- **AI**: Anthropic Claude API + OpenAI (embeddings, Whisper)
- **Web Server**: Fastify
- **Logging**: Pino

## Monorepo Structure

```
echos/
├── packages/
│   ├── shared/       # Types, utils, security, config, logging, errors
│   ├── core/         # Agent, tools, storage, search, plugin system
│   ├── telegram/     # Telegram bot interface (grammY)
│   ├── web/          # Web UI interface (Fastify + pi-web-ui)
│   ├── cli/          # CLI binary (pnpm echos) — standalone terminal interface
│   └── scheduler/    # Background jobs (BullMQ) and cron tasks
├── plugins/
│   ├── article/          # Web article extraction plugin
│   ├── content-creation/ # Content creation and drafting plugin
│   ├── digest/           # Digest and summary generation plugin
│   ├── image/            # Image processing plugin
│   ├── journal/          # Journaling and reflective writing plugin
│   ├── resurface/        # Resurfacing / spaced-repetition plugin
│   ├── twitter/          # Twitter/X ingestion plugin
│   └── youtube/          # YouTube transcript extraction plugin
├── docker/           # Docker configuration
├── scripts/          # Deploy, backup, setup scripts
└── data/             # Runtime data (gitignored)
```

## Patterns

### Plugin System

Content processors are plugins, not core code. Each plugin implements `EchosPlugin`:

```typescript
import type { EchosPlugin, PluginContext } from '@echos/core';

const myPlugin: EchosPlugin = {
  name: 'my-processor',
  description: 'Processes some content type',
  version: '0.1.0',
  setup(context: PluginContext) {
    // Return AgentTool[] to register with the agent
    return [createMyTool(context)];
  },
};
export default myPlugin;
```

Plugins receive a `PluginContext` with access to storage, embeddings, logger, and config.
Register plugins via `PluginRegistry` in the entry point.

**CRITICAL — Adding a new plugin checklist (ALWAYS do ALL of these):**

1. Create `plugins/<name>/package.json` with the plugin package
2. Add `plugins/<name>/package.json` to `pnpm-workspace.yaml` (if not glob-matched)
3. Run `pnpm sync-plugins` — this auto-generates `tsconfig.paths.json` (TypeScript path aliases) and ensures root `package.json` has the `"workspace:*"` dependency entry
4. In the `deps` stage in `docker/Dockerfile`, add `COPY plugins/<name>/package.json plugins/<name>/`
5. In the `production` stage in `docker/Dockerfile`, add `COPY --from=deps /app/plugins/<name>/package.json plugins/<name>/`
6. Register the plugin in the daemon entry point

**Tool `execute` signatures must always include explicit types:**

The `execute` function in `AgentTool` must always have an explicitly typed first parameter to avoid `TS7006` implicit `any` errors:

```typescript
execute: async (_toolCallId: string, params: Params) => {
  // ...
}
```

Never write `async (_toolCallId, params: Params)` without the `: string` annotation.

### Tool Definitions

Core tools use TypeBox schemas for pi-agent-core compatibility:

```typescript
import { Type } from '@mariozechner/pi-ai';

const CreateNoteTool = {
  name: 'create_note',
  description: 'Create a new knowledge note',
  parameters: Type.Object({
    title: Type.String(),
    content: Type.String(),
    tags: Type.Optional(Type.Array(Type.String())),
  }),
  handler: async (params) => { /* ... */ },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albinotonnina/echos](https://github.com/albinotonnina/echos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
