---
trigger: always_on
description: This project represents a Java version of OpenClaw. OpenClaw is an open-source, personal AI assistant designed to run on your own devices. It acts as a control plane (Gateway) for an assistant that can interact across multiple communication channels.
---

# JavaClaw - Java Edition of OpenClaw

This project represents a Java version of OpenClaw. OpenClaw is an open-source, personal AI assistant designed to run on your own devices. It acts as a control plane (Gateway) for an assistant that can interact across multiple communication channels.

### Key Capabilities of OpenClaw
- **Multi-Channel Integration:** Supports platforms like WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, and more.
- **Background Daemon:** Runs as a background service to handle tasks and messages autonomously.
- **Extensible Skills:** Modular capabilities that can be added to enhance the assistant's functionality.
- **Local Control:** Designed for privacy and control, running on your own hardware.

---

## Technology Stack
- **Java 25**, Spring Boot 4.0.3, Spring Modulith 2.0.3
- **Job Scheduling:** JobRunr 8.5.0 — background jobs, dashboard on `:8081`
- **LLM Integration:** Spring AI 2.0.0-SNAPSHOT (OpenAI, Anthropic, Ollama)
- **MCP:** Spring AI MCP Client (Model Context Protocol)
- **Agent Framework:** Spring AI Agent Utils (Anthropic agent framework)
- **Database:** H2 (embedded)
- **Templating:** Pebble 4.1.1
- **Discord:** JDA 6.1.1 (Gateway / WebSocket)
- **Telegram:** Telegrambots 9.4.0 (long-polling)

---

## Module Structure

```
root
├── base/               ← Core: agent, tasks, tools, channels, config
├── app/                ← Spring Boot entry point, onboarding UI, web routes, chat channel
├── providers/
│   ├── anthropic/      ← Anthropic (Claude) provider + Claude Code OAuth support
│   ├── openai/         ← OpenAI (GPT) provider
│   ├── ollama/         ← Ollama local provider (no API key required)
│   └── google/         ← Google Gen AI (Gemini) provider
└── plugins/
    ├── telegram/       ← Telegram long-poll channel
    ├── brave/          ← Brave web search tool
    ├── discord/        ← Discord Gateway channel
    └── playwright/     ← Playwright browser tool
```

`app` depends on `base` + all `providers/` + all `plugins/`. `ChatChannel` lives inside `app/`.

Each **provider** implements `AgentOnboardingProvider` (in `base`) and is auto-discovered by Spring. Each **plugin** is an optional Spring Boot auto-configuration module that contributes tools or channels.

---

## Java Implementation Strategy (JavaClaw)

### Task Management
- Tasks are stored as Markdown files in the `workspace/tasks` folder.
- Path format for normal tasks: `yyyy-MM-dd/<HHmmss>-<name>.md`.
- Path format for recurring tasks: `recurring/<name>.md`.
- **States:** `todo`, `in_progress`, `completed`, `awaiting_human_input` — stored in YAML frontmatter (not in the filename).
- Task file format: YAML frontmatter with `task`, `createdAt`, `status`, `description` fields.
- The task `id` is the absolute file path, set by `FileSystemTaskRepository` on first save.

### Core Task Flow
```
User/Agent → TaskManager.create()
  → Task written as .md file via TaskRepository
  → JobRunr enqueues TaskHandler.executeTask(taskId)
  → TaskHandler: marks in_progress → prompts Agent → marks completed/awaiting_human_input
  → On error: resets status back to todo (retried up to 3 times by JobRunr)
```

### Core Components
- **`TaskManager`** (`base/src/main/java/ai/javaclaw/tasks/TaskManager.java`): Creates, schedules (immediate, specific-time, or cron), and manages recurring tasks. Saves via `TaskRepository`, then enqueues to JobRunr by task ID.
- **`Task`** (`base/.../tasks/Task.java`): Immutable value class. Fields: `id` (absolute file path), `name`, `createdAt`, `status`, `description`. Mutation via `withStatus()` / `withFeedback()`.
- **`TaskRepository`** (`base/.../tasks/TaskRepository.java`): Interface for task persistence — `save(Task)`, `getTaskById(String)`, `getTasks(LocalDate, Status)`, `save(RecurringTask)`, `getRecurringTaskById(String)`.
- **`FileSystemTaskRepository`** (`base/.../tasks/FileSystemTaskRepository.java`): Implements `TaskRepository`; handles all file I/O, YAML frontmatter serialization/deserialization, directory management, and filename sanitization.
- **`TaskHandler`** (`base/.../tasks/TaskHandler.java`): `@Job(retries=3)`-annotated; executes task via agent prompt; returns `TaskResult(Status newStatus, String feedback)` record (nested inside `TaskHandler`). Resets to `todo` on exception.
- **`RecurringTask`** (`base/.../tasks/RecurringTask.java`): Immutable value class for recurring task templates stored in `workspace/tasks/recurring/`.
- **`RecurringTaskHandler`** (`base/.../tasks/RecurringTaskHandler.java`): JobRunr worker spawning normal tasks from recurring templates on cron schedule.
- **`TaskNotFoundException`** (`base/.../tasks/TaskNotFoundException.java`): Thrown when a task file cannot be found or read by its ID.

### Workspace
- `workspace/AGENT.md`: System instructions + user-specific information (editable during onboarding).
- `workspace/AGENT-ORIGINAL.md`: Template / backup of original AGENT.md.
- `workspace/INFO.md`: Environment context auto-injected into every prompt.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jobrunr/JavaClaw](https://github.com/jobrunr/JavaClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
