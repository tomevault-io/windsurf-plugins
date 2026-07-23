---
trigger: always_on
description: Brand Operating System (BOS) 3.0 - AI-powered brand management platform built with Next.js 16+, React 19, TypeScript, and Tailwind CSS. Features multi-model AI chat (Claude, Perplexity), brand knowledge systems, and collaborative workspaces.
---

# BOS 3.0 - Claude Development Guide

## Project Overview
Brand Operating System (BOS) 3.0 - AI-powered brand management platform built with Next.js 16+, React 19, TypeScript, and Tailwind CSS. Features multi-model AI chat (Claude, Perplexity), brand knowledge systems, and collaborative workspaces.

## Essential Commands
```bash
bun dev          # Start dev server (http://localhost:3000)
bun run build    # Production build
bun run lint     # ESLint
```

## Tech Stack
- **Framework**: Next.js 16+ App Router
- **UI**: React 19 + React Aria Components (accessibility)
- **Styling**: Tailwind CSS with UUI semantic tokens (CSS variables)
- **State**: Zustand
- **Database**: Supabase
- **AI**: Anthropic SDK (Claude), Perplexity API

## Key Directories
```
app/              # Next.js pages and API routes
components/ui/    # Design system primitives (buttons, inputs, etc.)
components/chat/  # Chat interface components
lib/ai/           # AI provider configs, auto-router, tools
lib/supabase/     # Database services
hooks/            # Custom React hooks
```

## Claude Configuration Structure

The `.claude/` directory is organized for Claude Code CLI compatibility:

```
.claude/
├── CLAUDE.md                    # This file (CLI required at root)
├── settings.json                # MCP permissions (CLI required at root)
├── settings.local.json          # Local dev permissions (gitignored)
│
├── agents/                      # Autonomous workflows (CLI required at root)
│   ├── README.md
│   └── feature-dev/
│
├── commands/                    # Slash commands (CLI required at root)
│   ├── README.md
│   └── restart.md
│
├── plugins/                     # Full capability packages (CLI required at root)
│   ├── agent-sdk-dev/
│   ├── code-review/
│   ├── commit-commands/
│   ├── feature-dev/
│   ├── hookify/
│   ├── plugin-dev/
│   ├── pr-review-toolkit/
│   └── ralph-wiggum/
│
├── skills/                      # Auto-activating knowledge (CLI required at root)
│   ├── bos-code-quality/
│   ├── brand-guidelines/
│   ├── frontend-design/
│   └── ...
│
├── brand/                       # Brand content (for creatives)
│   ├── identity/                # Brand guidelines, PDFs
│   ├── writing/                 # Writing style guides
│   └── assets/                  # Future: logos, images
│
├── data/                        # Reference data
│   └── news-sources.md
│
├── reference/                   # Documentation
│   ├── design-system.md         # BOS design system reference
│   └── mcp-setup.md             # MCP server configuration
│
└── system/                      # Auto-generated (read-only)
    └── architecture.md          # Codebase architecture
```

### Categories by Audience

| Category | Folder | Audience | Contents |
|----------|--------|----------|----------|
| **CLI Tools** | `agents/`, `commands/`, `plugins/`, `skills/` | Engineers | CLI-required paths |
| **Brand** | `brand/` | Creatives | Identity docs, writing styles, assets |
| **Data** | `data/` | Both | Reference data (news sources, etc.) |
| **Reference** | `reference/` | Both | Design system, MCP setup docs |
| **System** | `system/` | Both | Auto-generated architecture |

### Agents vs Commands vs Skills

| Type | Trigger | Purpose | Location |
|------|---------|---------|----------|
| **Agents** | Auto-activates on context | Autonomous multi-step workflows | `agents/` or `plugins/*/agents/` |
| **Commands** | User types `/command` | Single operations, user-controlled | `commands/` or `plugins/*/commands/` |
| **Skills** | Auto-activates on keywords | Context-aware knowledge injection | `skills/` or `plugins/*/skills/` |
| **Plugins** | Contains all of the above | Full-featured packages | `plugins/` |

### Plugin-Embedded Agents

Some agents live inside plugins as subagents for larger workflows:

| Plugin | Embedded Agents |
|--------|-----------------|
| `feature-dev` | code-architect, code-explorer, code-reviewer |
| `pr-review-toolkit` | code-reviewer, silent-failure-hunter, code-simplifier, comment-analyzer, pr-test-analyzer, type-design-analyzer |
| `plugin-dev` | agent-creator, skill-reviewer, plugin-validator |
| `agent-sdk-dev` | agent-sdk-verifier-ts, agent-sdk-verifier-py |
| `hookify` | conversation-analyzer |

### Where to Look First

- **Building a feature?** → Check `plugins/feature-dev/`
- **Reviewing a PR?** → Check `plugins/pr-review-toolkit/`
- **Creating a plugin?** → Check `plugins/plugin-dev/`
- **Brand questions?** → Check `brand/identity/`
- **Writing content?** → Check `brand/writing/`
- **Design system?** → Check `reference/design-system.md`

---

## Code Conventions

### TypeScript
- Strict mode enabled
- Prefer `interface` for component props
- Use Zod for runtime validation where needed

### Components
- Use React Aria Components for accessible primitives
- Co-locate component-specific types in the same file
- Prefer composition over prop drilling

### Styling with CSS Variables
All colors use semantic CSS variables from `theme.css`:
```css
/* Backgrounds */
var(--bg-primary)      /* Main background */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensesh/OS_design-directory](https://github.com/opensesh/OS_design-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
