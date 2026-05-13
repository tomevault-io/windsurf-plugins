---
trigger: always_on
description: Copy the following section to your project's `CLAUDE.md` file to enable Claude to proactively use the memory system:
---

# cc-obsidian-mem

## For Users: Add to Your Project's CLAUDE.md

Copy the following section to your project's `CLAUDE.md` file to enable Claude to proactively use the memory system:

```markdown
## Memory System (cc-obsidian-mem)

You have access to a persistent memory system via MCP tools. Use it proactively.

### Available Tools

| Tool                  | Use When                                                 |
| --------------------- | -------------------------------------------------------- |
| `mem_search`          | Looking for past decisions, errors, patterns, or context |
| `mem_read`            | Need full content of a specific note                     |
| `mem_write`           | Saving important decisions, patterns, or learnings       |
| `mem_write_knowledge` | Saving Q&A, explanations, research from conversations    |
| `mem_supersede`       | Updating/replacing outdated information                  |
| `mem_project_context` | Starting work on a project (get recent context)          |
| `mem_list_projects`   | Need to see all tracked projects                         |
| `mem_generate_canvas` | Generate Obsidian canvas visualizations for a project    |
| `mem_file_ops`        | Delete, move, or create directories in the vault         |

### When to Search Memory

**Proactively search memory (`mem_search`) when:**

- Starting work on a codebase - check for project context and recent decisions
- Encountering an error - search for similar errors and their solutions
- Making architectural decisions - look for related past decisions
- User asks "how did we..." or "why did we..." or "what was..."
- Implementing a feature similar to past work

**Example searches:**

- `mem_search query="authentication" type="decision"` - Find auth-related decisions
- `mem_search query="TypeError" type="error"` - Find past TypeScript errors
- `mem_search query="database schema"` - Find DB-related knowledge
- `mem_project_context project="my-project"` - Get full project context

### When to Save to Memory

**Save to memory (`mem_write`) when:**

- Making significant architectural or technical decisions
- Discovering important patterns or gotchas
- Solving tricky bugs (save the solution)
- Learning something project-specific that will be useful later

**Use `mem_supersede` when:**

- A previous decision is being replaced
- Updating outdated documentation or patterns
```

---

## For Contributors: Development Guide

### Version Bump Checklist

When releasing a new version, update the version number in **all four files**:

| File                                | Field                | Example              |
| ----------------------------------- | -------------------- | -------------------- |
| `plugin/package.json`               | `version`            | `"version": "1.0.4"` |
| `plugin/.claude-plugin/plugin.json` | `version`            | `"version": "1.0.4"` |
| `.claude-plugin/marketplace.json`   | `plugins[0].version` | `"version": "1.0.4"` |
| `plugin/src/mcp-server/index.ts`    | `version`            | `version: "1.0.4"`   |

### Project Structure

```
cc-obsidian-mem/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace metadata (version here!)
├── plugin/                   # The actual plugin
│   ├── .claude-plugin/
│   │   └── plugin.json       # Plugin metadata (version here!)
│   ├── package.json          # NPM package (version here!)
│   ├── hooks/
│   │   ├── hooks.json        # Hook definitions
│   │   └── scripts/          # Hook implementations
│   ├── scripts/              # Utility scripts (backfill, migrations)
│   ├── src/
│   │   ├── vault/            # Vault management, canvas generation
│   │   ├── summarizer/       # AI-powered knowledge extraction
│   │   ├── mcp-server/       # MCP server for mem_* tools
│   │   ├── session-end/      # Background session processing
│   │   ├── sqlite/           # SQLite database operations
│   │   ├── context/          # Context injection for prompts
│   │   ├── sdk/              # SDK agent integration
│   │   ├── shared/           # Shared types, config, validation, logging
│   │   ├── cli/              # Setup CLI
│   │   ├── fallback/         # JSON fallback storage
│   │   └── worker/           # Background worker service
│   └── tests/
└── CLAUDE.md                 # This file
```

### Key Files by Feature

#### Vault & Knowledge Management

- `plugin/src/vault/vault-manager.ts` - Vault CRUD, search, project structure, topic deduplication
- `plugin/src/vault/note-builder.ts` - Frontmatter building, filename generation, wikilinks
- `plugin/src/vault/canvas.ts` - Canvas generation (dashboard, timeline, graph layouts)
- `plugin/src/summarizer/summarizer.ts` - AI-powered knowledge extraction using Claude CLI
- `plugin/src/summarizer/prompts.ts` - System prompts for knowledge extraction

#### Hook Scripts

- `plugin/hooks/scripts/session-start.ts` - Initialize session tracking, inject project context
- `plugin/hooks/scripts/user-prompt-submit.ts` - Track user prompts
- `plugin/hooks/scripts/post-tool-use.ts` - Capture tool observations, exploration tracking
- `plugin/hooks/scripts/pre-compact.ts` - Trigger background summarization before compaction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Z-M-Huang/cc-obsidian-mem](https://github.com/Z-M-Huang/cc-obsidian-mem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
