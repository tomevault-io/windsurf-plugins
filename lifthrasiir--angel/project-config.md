---
trigger: always_on
description: - **Project:** Angel - A personalized coding agent using Go and React/TypeScript.
---

# Key Concepts and Cautions for Agent Working on this Codebase

- **Project:** Angel - A personalized coding agent using Go and React/TypeScript.
- **Goal:** Create a simple, single-user web version of `@google/gemini-cli`.
- **Default LLM Model:** `gemini-2.5-flash` (with support for other Gemini 2.5 and 3 models)
- **Go Version:** Requires Go 1.25 or later.

## Project Features

- **Flexible LLM Integration**: The application supports multiple ways to connect to LLMs. It can use the Gemini Code Assist API (via Google OAuth) for free-tier access, or connect directly to the Gemini API using a standard API key.
- **Multi-session and Workspaces**: Users can create and manage multiple chat sessions, which can be organized into distinct workspaces.
- **Temporary Sessions**: Users can create temporary sessions that are automatically deleted after 48 hours. These are indicated by an icon next to the session name and created via the `/temp` endpoint or `isTemporary` flag.
- **Branching**: Users can create new conversation branches from existing user messages and switch between different branches within a session, allowing for exploration of alternative conversation paths. This is managed by linking messages using `parent_message_id` and `chosen_next_id` in the `messages` table, and associating them with specific `branch_id`s. The `primary_branch_id` in the `sessions` table indicates the currently active conversation path.
- **Configurable System Prompt per Session**: Each chat session allows for a custom system prompt. These prompts support Go templating for dynamic generation using the `EvaluatePrompt` function, with a preview feature available in the UI.
- **Automatic Session Name Inference**: The LLM can infer and update session names based on conversation content. This process is handled by `inferAndSetSessionName`, which uses specific prompts (`GetSessionNameInferencePrompts`) to guide the LLM.
- **Thought Display**: The LLM's internal thought processes are streamed as "thought" messages to the user interface. These thoughts are grouped and can be expanded/collapsed for detailed viewing. The `Part` struct includes a `Thought` field, and these messages are broadcast via the `EventThought` SSE event.
- **File Upload and Attachment**: Users can attach files to their messages. Files are Base64 encoded during transmission, but stored as raw binary data in a refcounted blob storage system via the `blobs` table, and support preview and download functionalities via the `/{mainSessionId}/@{blobHash}` endpoint. The blob system automatically manages reference counting to prevent data loss when files are shared across multiple messages. Images support automatic resizing (togglable) to optimize file sizes.
- **Tool Usage**: The LLM can invoke both built-in tools (e.g., `list_directory`, `read_file`, `write_file`, `run_shell_command`, `write_todo`, `web_fetch`, `subagent`, `generate_image`, `search_chat`, `recall`) and external tools via the Model Context Protocol (MCP). The `web_fetch` tool can extract URLs, check for private IPs, convert GitHub blob URLs to raw URLs, and includes a fallback mechanism for direct fetching. The `write_file` tool returns a unified diff for verification. The `subagent` tool allows spawning specialized subagents with custom system prompts for specific tasks. The `generate_image` tool enables image creation and editing using subagents with image generation capabilities. The `search_chat` tool searches through chat history using keywords, returning matching messages with context excerpts. The `recall` tool retrieves unprocessed binary content for internal AI processing using SHA-512/256 hashes. Directories can be exposed or unexposed via /(un)expose commands, and their contents and per-directory directives (GEMINI.md etc.) are given as dynamic system prompts.
- **Chat History Compression**: The chat history can be compressed into a concise, structured XML snapshot by the user explicitly running the `/compress` command. This process is handled by the `CompressSession` function, which uses token thresholds to determine when to summarize and updates the message chain with the compressed content.
- **History Context Management**: Users can clear chat history or blob storage using the `/clear` and `/clearblobs` commands for context alternation and storage management.

## Go Backend

- **Authentication (`src/gemini/code_assist.go`, `src/gemini/google_login.go`)**: Manages user authentication. It supports two main methods: Google OAuth for the free-tier Code Assist API and direct API key authentication for the Gemini API.
- **Database (`src/internal/database`)**:
  - **Split-DB Architecture**: Uses a main SQLite database (`angel.db`) for workspaces and searchable metadata, with individual session databases stored as separate files (`angel-data/sessions/<mainSessionId>.db`). Each session DB contains its own `sessions`, `branches`, `messages`, `blobs`, `shell_commands`, and `session_envs` tables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lifthrasiir/angel](https://github.com/lifthrasiir/angel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
