---
trigger: always_on
description: Claude Code intelligent agent manager - TypeScript HTTP server with jQuery + Tailwind.css UI. Features Ralph Loop iterative development pattern and roadmap-based automation.
---

# Claudito - Project Context

Claude Code intelligent agent manager - TypeScript HTTP server with jQuery + Tailwind.css UI. Features Ralph Loop iterative development pattern and roadmap-based automation.

## Project Structure

```
src/
  index.ts          # Entry point (config + server instantiation only)
  config/           # Configuration loading
  server/           # Express server + WebSocket integration
  routes/           # API route handlers
  services/         # Business logic (ProjectService, RoadmapParser, InstructionGenerator, ClaudeOptimizationService)
  repositories/     # Data persistence (Project, Conversation, Settings)
  agents/           # Agent management (Agent interface, ClaudeBinary, AnthropicSdkAgent, OpencodeAgent, AgentManager)
  websocket/        # WebSocket server for real-time updates
  utils/            # Logger, error handling, retry utilities
public/
  vendor/           # Third-party assets (jQuery, Tailwind - NO CDN)
  js/               # Frontend JavaScript with WebSocket client
  css/              # Custom styles
test/
  unit/             # Unit tests
doc/
  ROADMAP.md        # Project milestones
  MERMAID_EXAMPLES.md # Mermaid.js diagram examples and reference
```

## Data Storage Structure

Global data in `$HOME/.claudito/`:
```
projects/
  index.json                      # [{ id, name, path }] - project registry
settings.json                     # Global settings + agentPromptTemplate
```

Project-specific data in `{project-root}/.claudito/`:
```
status.json                       # ProjectStatus object
conversations/
  {conversationId}.json           # Conversation with messages
```

## Key Interfaces

- **Infrastructure**: `ConfigLoader`, `HttpServer`, `ProjectWebSocketServer`, `EventManager` (in-memory event bus), `Logger` (with circular buffer)
- **Data**: `ProjectRepository` (status.json per project), `ConversationRepository` (per project/item), `SettingsRepository` (global settings + agentPromptTemplate)
- **Services**: `ProjectService`, `FilesystemService`, `GitService` (simple-git), `GitHubCLIService` (gh CLI wrapper), `RoadmapParser`, `RoadmapGenerator`, `InstructionGenerator`, `ClaudeOptimizationService` (edits files directly via Edit tool), `DataWipeService` (factory reset — wipes all Claudito data), `RunConfigurationService` (CRUD for run configs), `RunConfigImportService` (detects project files and suggests configs), `RunProcessManager` (node-pty process lifecycle with auto-restart), `InventifyService` (project idea generator using one-off agent + Ralph Loop)
- **Docker**: `DockerService` (CLI wrapper), `DockerCommandRunner` (testable command execution), `DockerProcessSpawner` (implements `ProcessSpawner` for Docker exec), `ContainerManager` (per-project container lifecycle, returns `EnsureContainerResult` with restart detection), `ImageManager` (image CRUD + variants)
- **Agents**: `Agent` (provider-agnostic interface), `ClaudeBinary` (Claude CLI implementation), `AnthropicSdkAgent` (Vercel AI SDK implementation, chat-only), `AgentManager` (multi-agent lifecycle: interactive + one-off, profile-aware factory)

## API Endpoints

All project routes prefixed with `/api/projects/:id`. Standard REST verbs (GET/POST/PUT/DELETE).

**Global**: `GET /api/health` (includes `shellEnabled`), `GET /api/agents/status`, `GET|PUT /api/settings`, `GET /api/settings/models`, `POST /api/settings/wipe-all-data`

**Integrations** (`/api/integrations`): `GET github/status`, `GET github/repos(?owner=&language=&limit=)`, `GET github/repos/search(?query=&language=&sort=&limit=)`, `POST github/clone` (body: repo, targetDir, branch?, projectName?), `GET github/issues(?repo=&state=&label=&assignee=&milestone=&limit=)`, `GET github/issues/:num(?repo=)`, `POST github/issues` (body: repo, title, body?, labels?, assignees?, milestone?), `POST github/issues/:num/close(?repo=)`, `POST github/issues/:num/comment(?repo=)` (body: body), `GET github/labels(?repo=)`, `GET github/milestones(?repo=)`, `GET github/collaborators(?repo=)`, `POST github/pr` (body: repo, title, body, base?, draft?), `GET github/pulls(?repo=&state=&limit=)`, `GET github/pulls/:num(?repo=)`

**Filesystem** (`/api/fs`): `drives`, `browse?path=`, `browse-with-files?path=`, `read?path=`, `PUT write`, `DELETE delete`

**Projects**: CRUD on `/api/projects` + `/:id`

**Roadmap** (`/:id/roadmap`): GET (content+parsed), `POST generate`, PUT (modify), `POST respond`, `PUT next-item`, `POST task` (add task), DELETE `task|milestone|phase`

**Agent** (`/:id/agent`): `POST interactive` (start session), `POST send`, `POST answer` (AskUserQuestion tool_result), `POST stop`, GET `status|context|loop|queue`, DELETE `queue(/:index)`

**One-Off Agents** (`/:id/agent/oneoff/:oneOffId`): `POST send|stop`, GET `status|context`

**Conversations** (`/:id`): GET `conversation|conversations(?limit=N)`, `PUT conversations/:conversationId` (rename)

**Config** (`/:id`): GET/PUT `claude-files|permissions|model`, GET `optimizations|debug`

**Git** (`/:id/git`): `POST generate-pr-description` (auto-generate PR title/body from conversation + diff), `GET user-name`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [comfortablynumb/claudito](https://github.com/comfortablynumb/claudito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
