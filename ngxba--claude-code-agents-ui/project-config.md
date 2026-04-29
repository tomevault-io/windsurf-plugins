---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

agents-ui is a Nuxt 3-based visual dashboard for managing Claude Code agents, commands, skills, workflows, and plugins. It provides a GUI layer on top of the `~/.claude` directory, allowing users to create, edit, and organize their Claude Code configuration without touching markdown files directly.

## Development Commands

```bash
# Development
bun run dev          # Start dev server at http://localhost:3030
npm run dev          # Alternative with npm

# Build & Production
bun run build        # Build for production
bun run preview      # Preview production build

# Type Checking
bun run typecheck    # Run TypeScript type checking
```

## Architecture

### Frontend (Nuxt 3 + Vue 3)

**Pages** (`app/pages/`):
- `/agents` - List and manage agents
- `/agents/[slug]` - Edit individual agent
- `/commands` - List and manage commands
- `/skills` - List and manage skills
- `/workflows` - Visual workflow builder
- `/graph` - Relationship visualization
- `/explore` - Browse templates and marketplace
- `/cli` - Full terminal emulator with context monitoring
- `/settings` - Global settings

**Composables** (`app/composables/`):
The app uses a centralized CRUD pattern via `useCrud.ts` which provides standard `fetchAll`, `fetchOne`, `create`, `update`, `remove` operations. Domain-specific composables wrap this:
- `useAgents.ts` - Agent CRUD operations
- `useCommands.ts` - Command CRUD operations
- `useSkills.ts` - Skill CRUD operations
- `useWorkflows.ts` - Workflow CRUD operations
- `useStudioChat.ts` - Agent Studio chat with SSE streaming
- `useGithubImports.ts` - Import skills from GitHub repos
- `useMarketplace.ts` - Browse and install plugins from marketplace
- `useTerminal.ts` - Terminal emulator with xterm.js integration and WebSocket streaming
- `useContextMonitor.ts` - Real-time context metrics tracking (tokens, cost, files, tools)
- `useCliExecution.ts` - CLI session management with agent-aware and standalone modes

**Chat/Studio System**:
The Agent Studio (`/agents/[slug]` page with test panel) uses SSE (Server-Sent Events) for streaming responses. The backend (`server/api/chat.post.ts`) uses the `@anthropic-ai/claude-agent-sdk` to query agents and stream results back as `text_delta`, `thinking_delta`, `tool_progress` events.

### Backend (Nuxt Server API)

**File System Layer** (`server/utils/`):
- `claudeDir.ts` - Resolves `~/.claude` path (respects `CLAUDE_DIR` env var)
- `frontmatter.ts` - Parse/serialize YAML frontmatter + markdown body
- `relationships.ts` - Extract relationships between agents/commands/skills by scanning frontmatter (`agent:` field) and body text for references
- `github.ts` - Clone, scan, and import skills from GitHub repos
- `marketplace.ts` - Fetch and install plugins from marketplace sources

**API Routes** (`server/api/`):
All CRUD routes follow REST conventions:
- `GET /api/agents` - List all
- `GET /api/agents/[slug]` - Get one
- `POST /api/agents` - Create
- `PUT /api/agents/[slug]` - Update
- `DELETE /api/agents/[slug]` - Delete

Special endpoints:
- `POST /api/chat` - SSE endpoint for Agent Studio, uses `@anthropic-ai/claude-agent-sdk` to execute agents with streaming
- `GET /api/relationships` - Build graph data by extracting relationships
- `GET /api/agents/[slug]/skills` - List skills assigned to an agent
- `POST /api/github/import` - Import skills from GitHub
- `POST /api/marketplace/install` - Install plugin from marketplace

### Data Model

**Agents** (`~/.claude/agents/*.md`):
```yaml
---
name: Agent Name
description: What the agent does
model: sonnet | opus | haiku
color: "#hex"
memory: user | project | none
---

Agent instructions go here...
```

**Commands** (`~/.claude/commands/**/*.md`):
```yaml
---
name: command-name
description: What the command does
argument-hint: "[optional args]"
allowed-tools: [Read, Write, Edit]
agent: agent-slug  # optional, link to agent
---

Command prompt goes here...
```

**Skills** (`~/.claude/skills/[name]/SKILL.md`):
```yaml
---
name: skill-name
description: What the skill does
context: when | always
agent: agent-slug  # optional, link to agent
---

Skill prompt goes here...
```

**Workflows** (`~/.claude/workflows/*.json`):
```json
{
  "name": "Workflow Name",
  "description": "Description",
  "steps": [
    { "id": "step-1", "agentSlug": "agent-name", "label": "Step label" }
  ],
  "createdAt": "ISO timestamp"
}
```

### Key Patterns

**Relationship Detection**:
The system automatically detects relationships between agents/commands/skills by:
1. Checking `agent:` frontmatter field (links command/skill → agent)
2. Scanning body text for `subagent_type: "agent-name"` patterns
3. Scanning for `/command-name` references in agent bodies
4. Matching direct mentions of agent slugs in text

**Studio Chat System**:
- Frontend uses `useStudioChat()` which streams SSE events
- Backend uses `@anthropic-ai/claude-agent-sdk`'s `query()` function
- System prompt is either the default "Agent Manager" prompt or the selected agent's instructions
- Session IDs enable conversation continuation across multiple messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ngxba/claude-code-agents-ui](https://github.com/Ngxba/claude-code-agents-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
