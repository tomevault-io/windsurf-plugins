---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MailMind (mailmind) is an AI-powered email agent built on LangGraph. It combines large language models with modern email systems (IMAP/SMTP) to provide intelligent email management through a web interface.

The core `deepagents` package is a general-purpose agent framework with sub-agent spawning, todo list capabilities, and a mock file system. The `web_app` layer adds email-specific functionality on top of this framework.

## Common Commands

### Linting and Formatting
```bash
# Check and format all Python files
make lint

# Format files (auto-fix issues)
make format

# Format only files changed from master branch
make format_diff
```

### Testing
```bash
# Run unit tests with coverage
make test

# Run integration tests
make integration_test

# Quick agent development test
python lib/test.py
```

### Web Application
```bash
# Start both backend and frontend (from web_app/ directory)
./start.sh

# Backend only (from web_app/backend/)
python api_server.py

# Frontend only (from web_app/frontend/)
npm run dev
npm run build
npm run preview
```

## High-Level Architecture

### DeepAgents Framework (`deepagents/`)

The agent framework follows a **backend-agnostic middleware pattern**:

- **Backends** (`deepagents/backends/`): Provide file storage and execution capabilities
  - `StateBackend`: Stores files in agent state (default, in-memory)
  - `FilesystemBackend`: Stores files on disk at `root_dir`
  - `SandboxBackend`: Adds shell command execution support
  - Protocol defined in `protocol.py` with `BackendProtocol`, `SandboxBackendProtocol`

- **Middleware** (`deepagents/middleware/`): Transforms agent requests/responses
  - `SubAgentMiddleware`: Provides the `task` tool for spawning subagents
  - `FilesystemMiddleware`: Provides file tools (`ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, `execute`)
  - `SkillsMiddleware`: Loads skill files from backend for reusability
  - `MemoryMiddleware`: Loads memory/AGENTS.md files into system prompt

- **Agent Creation** (`deepagents/graph.py`): `create_deep_agent()` assembles middleware stack

### Subagent Architecture

The framework uses **subagents** for task isolation and parallelization:

1. Subagents are stateless - each invocation is independent
2. They receive only the task description, no ability to clarify requirements
3. Subagents inherit middleware from the main agent (via `default_middleware`)
4. The `task` tool description is dynamically generated based on available subagent types
5. Subagents return a single final message to the main agent

### Web Application (`web_app/`)

**Backend** (`web_app/backend/api_server.py`):
- FastAPI server with WebSocket support for real-time streaming
- Session-based architecture: each chat session gets an isolated agent instance
- Email cache persisted to `.emails_cache.json` (survives server restarts)
- Session-isolated filesystems in `agent_fs/session_{id}_{hash}/`
- API endpoints at `/api/` routes, WebSocket at `/ws/chat/{session_id}`
- Tavily search integration for web search capabilities

**Frontend** (`web_app/frontend/`):
- React 18 + TypeScript with Vite
- Tailwind CSS for styling
- WebSocket client for streaming responses
- React Context for state management
- Key pages: Chat, Emails, Contacts, Settings

### Email Tools (`lib/email_tools.py`)

Email operations are implemented as LangChain tools:

**Core Email Operations:**
- `email_dashboard`: Get comprehensive email statistics and overview
- `read_emails`: Fetch emails with filtering (folder, sender, subject, etc.)
- `send_email`: Compose and send emails
- `delete_email`: Delete emails
- `move_email`: Move emails between folders
- `flag_email`: Mark as important/unread/seen
- `search_address_book`: Search contacts by name, email, or group
- `modify_address_book`: Add/edit/delete contacts and groups
- `list_folders`: List all available mailbox folders
- `download_attachments`: Download email attachments

Tools use `imap_tools` library and support proxy configuration via `better-proxy`.

### Address Book (`lib/address_book.json`)

Contact management is persisted to `lib/address_book.json`:
- AI can search, add, edit, and delete contacts
- Contacts support multiple email addresses and group assignments
- Used by the AI for quick email composition and filtering

### Session Management

Each chat session maintains:
- Separate agent instance with isolated state
- Independent email cache (cleaned up on session deletion)
- Dedicated filesystem directory (`agent_fs/session_{id}_{hash}/`)
- Unique conversation history persisted via LangGraph checkpointers

### Knowledge Base (`knowledge/`)

The `knowledge/` directory stores persistent information that agents can access:
- Used by the MemoryMiddleware to load context into agent prompts
- Can store AGENTS.md files with long-term knowledge
- AI can read and write files here for cross-session knowledge persistence

### Environment Configuration

The web application uses `.env` file in `web_app/`:

**Model Settings:**
- `MODEL`: LLM model name (e.g., `gpt-4`, `claude-3-sonnet`)
- `OPENAI_API_KEY`: API key

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LivXue/open-email-agent](https://github.com/LivXue/open-email-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
