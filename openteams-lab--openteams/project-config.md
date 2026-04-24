---
trigger: always_on
description: **OpenTeams** is a multi-agent conversation platform where multiple AI agents (Claude Code, Gemini CLI, Codex, QWen Coder, etc.) can collaborate in shared chat sessions like a real team. Key features:
---

# Repository Guidelines

## Project Overview

**OpenTeams** is a multi-agent conversation platform where multiple AI agents (Claude Code, Gemini CLI, Codex, QWen Coder, etc.) can collaborate in shared chat sessions like a real team. Key features:
- **Multi-agent chat sessions** with real-time streaming
- **Context synchronization** across agents
- **Agent execution tracking** with diffs and logs
- **Session archive/restore** functionality
- **Permission-based access control**
- **Skill registry** for extensible agent capabilities

### Supported AI Agents
- Claude Code (`@anthropic-ai/claude-code`)
- Gemini CLI (`@google/gemini-cli`)
- Codex (`@openai/codex`)
- QWen Coder (`@qwen-code/qwen-code`)
- Amp
- More coming soon

## Project Structure & Module Organization

### Backend (Rust Workspace)
- `crates/db/`: SQLx database models and migrations
  - Core chat models: `chat_session.rs`, `chat_agent.rs`, `chat_session_agent.rs`, `chat_message.rs`, `chat_run.rs`, `chat_permission.rs`, `chat_artifact.rs`
  - Chat extensions: `chat_skill.rs`, `chat_agent_skill.rs`, `chat_work_item.rs`
  - Analytics: `analytics.rs`
  - Workspace: `workspace.rs`, `workspace_repo.rs`, `repo.rs`, `project.rs`, `project_repo.rs`
  - Legacy models: `task.rs`, `session.rs`, `image.rs`, `execution_process.rs`, `coding_agent_turn.rs`
  - 80+ migrations in `migrations/`
- `crates/server/`: API server and binaries
  - Chat routes: `src/routes/chat/` (sessions.rs, agents.rs, messages.rs, runs.rs, skills.rs, work_items.rs, presets.rs)
  - Type generation: `src/bin/generate_types.rs`
- `crates/services/`: Business logic (40+ services)
  - `chat.rs`: Message parsing, mentions, attachments
  - `chat_runner.rs`: Agent execution orchestration, WebSocket streaming
  - `skill_registry.rs`: Skill discovery and management
  - `native_skills.rs`: Built-in skill implementations
  - `analytics.rs`: Usage analytics tracking
  - `config/`: Configuration management with version migrations
  - `git_host/`: GitHub, Azure DevOps integrations
  - `events/`: Event streaming and patches
- `crates/executors/`, `crates/utils/`, `crates/deployment/`, `crates/local-deployment/`, `crates/git/`, `crates/review/`, `crates/remote/`: Supporting crates

### Frontend
- `frontend/`: Main React + TypeScript application (Vite, Tailwind)
  - `src/pages/ui-new/`: **New design** pages
    - ChatSessions.tsx, Workspaces.tsx, WorkspacesLanding.tsx
    - ProjectKanban.tsx, VSCodeWorkspacePage.tsx, MigratePage.tsx
    - ElectricTestPage.tsx
  - `src/pages/`: **Legacy design** pages (Projects.tsx, ProjectTasks.tsx - wrapped in LegacyDesignScope)
  - `src/components/ui-new/primitives/`: 50+ new design components
    - Chat components: ChatBoxBase, CreateChatBox, SessionChatBox
    - Form components: InputField, Dropdown, MultiSelectDropdown, SearchableDropdown
    - Navigation: AppBar, ViewNavTabs, Toolbar, CommandBar
    - Display: Label, Badge, StatusDot, PriorityIcon, PrBadge
    - And many more...
  - `src/components/ui-new/primitives/conversation/`: 17 specialized message renderers
    - ChatUserMessage, ChatAssistantMessage, ChatSystemMessage, ChatThinkingMessage
    - ChatMarkdown, ChatToolSummary, ChatAggregatedDiffEntries, ChatApprovalCard
  - `src/components/ui/`: Legacy design system components
  - `src/lib/api.ts`: API client with chatApi methods
  - `src/styles/`: CSS for both design systems
    - `new/index.css` + `tailwind.new.config.js` (scoped to `.new-design`)
    - `legacy/index.css` + `tailwind.legacy.config.js` (scoped to `.legacy-design`)
- `remote-frontend/`: Lightweight remote deployment frontend
- `shared/`: Generated TypeScript types from Rust (`shared/types.ts` - auto-generated)
- `npx/openteams-npx/`: NPM CLI package (`openteams`) for cross-platform installer
- `scripts/`: Development helpers (port management, DB preparation, desktop packaging)
- `docs/`: Documentation (Mintlify setup)
- `src-tauri/`: Tauri desktop application configuration
- `assets/`, `dev_assets_seed/`, `dev_assets/`: Packaged and local dev assets

## Chat System Architecture

### Database Schema
The chat system uses 10 core entities:
- **ChatSession**: Conversation containers with title, status (active/archived), team_protocol
- **ChatAgent**: Agent definitions with name, runner_type, system_prompt, tools_enabled
- **ChatSessionAgent**: Join table linking agents to sessions; tracks state (idle/running/waiting_approval/dead), workspace_path, allowed_skill_ids
- **ChatMessage**: Messages with sender_type (user/agent/system), mentions, metadata
- **ChatRun**: Agent execution runs per session_agent; includes run_index, run_dir, log/output paths
- **ChatPermission**: Access control with capability, scope, TTL
- **ChatArtifact**: Files/artifacts pinned to sessions
- **ChatSkill**: Skill registry entries with triggers, enabled status, source
- **ChatAgentSkill**: Many-to-many relation between agents and skills
- **ChatWorkItem**: Work items tracked within sessions

### API Routes (`/chat`)
```
/chat
├── /sessions (list, create)
│   ├── /{session_id}
│   │   ├── / (get, update, delete)
│   │   ├── /archive (POST), /restore (POST)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openteams-lab/openteams](https://github.com/openteams-lab/openteams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
