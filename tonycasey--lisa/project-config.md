---
trigger: always_on
description: Lisa supports multiple AI coding assistants through a unified, event-driven architecture:
---

# Lisa Development Guide

## Multi-CLI Architecture

Lisa supports multiple AI coding assistants through a unified, event-driven architecture:

### Supported CLIs

| CLI Tool | Integration | Directory | Status |
|----------|-------------|-----------|--------|
| **Claude Code** | Lifecycle hooks | `.claude/` | Stable |
| **OpenCode** | Plugin system | `.opencode/` | Stable |

### Shared Resources

Both CLIs share resources from `.lisa/`:

```
.lisa/                        # Source of truth (shared)
├── skills/                   # Memory, tasks, lisa, jira, git
├── rules/                    # Coding standards
├── .env                      # Storage configuration

.claude/                      # Claude Code specific
├── settings.json             # Hook commands registered here
├── skills/
│   └── lisa/ -> ../../.lisa/skills  # Subdirectory symlink
└── rules/
    └── lisa/ -> ../../.lisa/rules   # Subdirectory symlink

.opencode/                    # OpenCode specific
├── plugin/
│   └── lisa.js               # Bundled plugin
└── skills/
    ├── memory/ -> ../../.lisa/skills/memory
    ├── tasks/ -> ../../.lisa/skills/tasks
    └── ...                   # Individual skill symlinks
```

**Note:** Lisa uses subdirectory symlinks to preserve any existing user files in `.claude/` or `.opencode/`.

### Event Mapping

Lisa events map to CLI-specific lifecycle hooks:

| Lisa Event | Claude Code | OpenCode |
|------------|-------------|----------|
| `session:start` (startup) | `SessionStart` trigger=startup | `session.created` |
| `session:start` (resume) | `SessionStart` trigger=resume | `session.updated` |
| `session:start` (compact) | `SessionStart` trigger=compact | `session.compacted` |
| `session:stop` (idle) | `Stop` | `session.idle` |
| `prompt:submit` | `UserPromptSubmit` | `message.updated` |

### CLI Selection

During `lisa init`, users can select which CLIs to support:

```bash
# Interactive (prompts for selection)
lisa init

# Non-interactive
lisa init --claude-only      # Only Claude Code
lisa init --opencode-only    # Only OpenCode
lisa init -y                 # Both (default)
```

---

## Core Library Development

### Where to Write Code

**All core library code lives in `src/lib/`**. This is the source of truth for:
- CLI commands (`cli.ts`)
- Domain interfaces and types (`domain/`)
- Infrastructure implementations (`infrastructure/`)
- Application handlers (`application/`)
- Service factories and DI (`services.ts`)

The `src/project/` directory contains **templates only** - hooks and skills that get deployed to destination projects. These are not core library code.

### Installation Model

Lisa is installed as a **global npm package** in destination projects:

```bash
# Global installation
npm install -g @tonycasey/lisa

# Initialize in a project
cd /path/to/your/project
lisa init
```

When `lisa init` runs, it:
1. Copies templates from the installed package to the project
2. Creates `.lisa/`, `.claude/`, and/or `.opencode/` directories
3. Sets up symlinks for shared resources
4. Configures storage backend (Local Docker or Zep Cloud)

The core library code (`dist/lib/`) remains in global `node_modules` and is invoked via the `lisa` CLI command.

### Clean Architecture

Lisa follows **Clean Architecture** principles with clear layer separation:

```
┌─────────────────────────────────────────────────────────┐
│                    CLI / Presentation                    │
│                      (src/lib/cli.ts)                   │
├─────────────────────────────────────────────────────────┤
│                     Application Layer                    │
│            (src/lib/application/handlers/)              │
│   SessionStartHandler, SessionStopHandler, etc.         │
├─────────────────────────────────────────────────────────┤
│                      Domain Layer                        │
│              (src/lib/domain/interfaces/)               │
│   IMemoryRepository, ITaskRepository, IMemoryItem, etc. │
├─────────────────────────────────────────────────────────┤
│                   Infrastructure Layer                   │
│              (src/lib/infrastructure/dal/)              │
│   McpMemoryRepository, Neo4jTaskRepository, ZepClient   │
└─────────────────────────────────────────────────────────┘
```

**Key principles:**
- **Domain layer has no dependencies** on infrastructure or application
- **Application layer** depends only on domain interfaces
- **Infrastructure layer** implements domain interfaces
- **Dependency inversion** via constructor injection

### Event-Driven Architecture

Lisa uses an **event-driven architecture** where CLI lifecycle events trigger handlers:

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────────┐
│  CLI Event   │ ──> │  Event Handler  │ ──> │  Domain Services │
│  (Hook)      │     │  (Application)  │     │  (Infrastructure)│
└──────────────┘     └─────────────────┘     └──────────────────┘
```

**Event flow:**
1. **Claude Code/OpenCode** triggers a lifecycle event (session start, stop, prompt submit)
2. **Hooks** (in `.claude/hooks/` or `.opencode/plugin/`) receive the event
3. **Handlers** in `src/lib/application/handlers/` process the event
4. **Services** in `src/lib/infrastructure/services/` execute domain logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TonyCasey/lisa](https://github.com/TonyCasey/lisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
