---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENT_ARCH.md

## Task Memory

`MEMORY.md` (at the project root) is the persistent task log for this project.

**Claude Code must:**
- Read `MEMORY.md` at the start of each session to restore context on active or recent tasks.
- Update `MEMORY.md` when starting a new task, making significant decisions, or completing work.
- Move completed tasks to the "Completed Tasks" section with a brief summary.
- Record open questions or blockers that carry over between sessions.

**Do not store in `MEMORY.md`:**
- Code patterns, architecture, or file paths (already in this file or derivable from the code).
- Information already documented elsewhere in `CLAUDE.md`.

## Communication Guidelines

- Respond to chat questions in French
- All code, comments, documentation, and commit messages must be in English
- Do not use emojis in responses

## Project Overview

OllamAssist is a JetBrains IDE plugin that integrates Ollama-powered AI capabilities directly into the development workflow. The plugin provides:
- In-IDE chat with Ollama models
- RAG (Retrieval-Augmented Generation) with workspace context
- Smart code autocompletion
- AI-powered commit message generation
- Code refactoring suggestions

## Build and Development Commands

### Building the Plugin
```bash
./gradlew build
```
This runs tests, benchmark tests, and builds the plugin distribution.

### Running Tests
```bash
# Run unit tests only
./gradlew test

# Run benchmark tests
./gradlew benchmark

# Run all tests
./gradlew check
```

### Running the Plugin in Development
```bash
./gradlew runIde
```
This launches a sandboxed IntelliJ IDEA instance with the plugin installed.

### Building Plugin Distribution
```bash
./gradlew buildPlugin
```
Output is in `build/distributions/`.

### Code Quality
```bash
./gradlew sonar
```
Requires `SONAR_TOKEN` environment variable.

## Architecture Overview

### Core Components

**IntelliJ Platform Integration:**
- `OllamAssistStartup` - Project startup activity that performs async prerequisite checks
- `OllamaWindowFactory` - Registers the chat tool window on the right panel
- Services use `@Service(Service.Level.PROJECT)` or `@Service(Service.Level.APPLICATION)`
- Plugin configuration in `src/main/resources/META-INF/plugin.xml`

**Service Architecture:**
- **Application Services:** `OllamAssistSettings`, `PrerequisiteService`, `IndexRegistry` (singleton across all projects)
- **Project Services:** `OllamaService`, `DocumentIndexingPipeline`, `LuceneEmbeddingStore`, `WorkspaceContextRetriever` (per-project instances)

### RAG Implementation

The plugin implements a custom RAG system for workspace-aware AI responses:

**Document Indexing Pipeline (`DocumentIndexingPipeline`):**
- Batch processor with async queue (batch size: 10, sync: 100)
- Scheduled executor runs every 30 seconds
- Thread-safe with `ReentrantLock` and `Phaser` synchronization
- Retry mechanism (max 3 retries per file)
- Use `flush()` for immediate indexing

**Embedding Store (`LuceneEmbeddingStore`):**
- Custom Apache Lucene-backed store
- Storage location: `{project}/.ollamassist/database/knowledge_index/`
- Thread-safe with read-write locks
- Stores vector embeddings, text content, and metadata

**Context Retrieval (`ContextRetriever`):**
- Composite retriever that parallelizes three sources:
  1. **EmbeddingStoreContentRetriever** - Semantic search (top-2, min score 0.85)
  2. **WorkspaceContextRetriever** - Current file context (5000-char window)
  3. **DuckDuckGoContentRetriever** - Web search (if enabled)
- Global timeout: 2 seconds
- Uses `CompletableFuture` for parallel execution

**File Monitoring (`ProjectFileListener`):**
- Monitors file creation, modification, deletion via `VirtualFileListener`
- Debounces changes with 1-minute window
- Updates embedding store automatically
- Respects `.gitignore` patterns

### Ollama Integration

**OllamaService (Project Service):**
- Creates `OllamaStreamingChatModel` from LangChain4j library
- Model parameters: temperature=0.7, topK=50, topP=0.85, timeout=300s
- Supports separate Ollama URLs for chat/completion/embedding models
- Optional Basic Auth via `AuthenticationHelper`
- Chat memory: `MessageWindowChatMemory` (max 25 messages)

**Assistant Interface:**
- Built using LangChain4j's `AiServices` framework
- System prompt enforces "Tree of Thoughts" reasoning
- Streaming responses via `TokenStream`
- Integrates with custom `ContextRetriever`

### Event System

The plugin uses IntelliJ's `MessageBus` for loose coupling:

| Event | When Published | Purpose |
|-------|----------------|---------|
| `ModelAvailableNotifier` | Models initialized | Shows UI when ready |
| `NewUserMessageNotifier` | User sends chat message | Triggers AI response |
| `ConversationNotifier` | Settings change | Clears chat history |
| `ChatModelModifiedNotifier` | Model changed | Reloads assistant |
| `StoreNotifier` | Files indexed | Updates embedding store |
| `PrerequisteAvailableNotifier` | Ollama checked | Shows prerequisite status |

### Chat System

**Message Flow:**
1. User input in `PromptPanel`
2. `AskToChatAction` publishes `NewUserMessageNotifier`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baretto-labs/OllamAssist](https://github.com/baretto-labs/OllamAssist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
