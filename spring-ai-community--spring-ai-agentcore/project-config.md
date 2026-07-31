---
trigger: always_on
description: Context for AI coding assistants working on this module.
---

# AGENTS.md

Context for AI coding assistants working on this module.

## Module Overview

Spring AI integration with Amazon AgentCore Code Interpreter. Executes Python, JavaScript, and TypeScript code in a secure sandbox with automatic file retrieval.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CodeInterpreterTools                        │
│                   (Tool implementation logic)                   │
├─────────────────────────────────────────────────────────────────┤
│  SESSION_ID_CONTEXT_KEY = "sessionId"                           │
│  executeCode(language, code) → String                           │
│    - Validates language (python, javascript, typescript)        │
│    - Gets sessionId from ToolCallReactiveContextHolder          │
│    - Executes code via client                                   │
│    - Stores files in FileStore                                  │
│    - Returns text-only result to LLM                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │ uses
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                  AgentCoreCodeInterpreterClient                 │
│                    (Low-level SDK wrapper)                      │
├─────────────────────────────────────────────────────────────────┤
│  startSession(name) → sessionId                                 │
│  executeCode(sessionId, language, code) → CodeExecutionResult   │
│  listFiles(sessionId, path) → List<String>                      │
│  readFiles(sessionId, paths) → List<GeneratedFile>              │
│  stopSession(sessionId)                                         │
│  executeInEphemeralSession(language, code) → CodeExecutionResult│
└───────────────────────────┬─────────────────────────────────────┘
                            │ returns
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CodeExecutionResult                         │
├─────────────────────────────────────────────────────────────────┤
│  String textOutput          // stdout/stderr combined           │
│  boolean isError            // from SDK isError flag            │
│  List<GeneratedFile> files  // images, PDFs, CSVs, etc.         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       GeneratedFile                             │
├─────────────────────────────────────────────────────────────────┤
│  String mimeType            // "image/png", "application/pdf"   │
│  byte[] data                // raw bytes (defensively copied)   │
│  String name                // filename                         │
│  isImage(), isText(), toDataUrl(), size()                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                 ArtifactStore<GeneratedFile>                    │
│            (Session-scoped file storage with TTL)               │
├─────────────────────────────────────────────────────────────────┤
│  DEFAULT_CATEGORY = "default"                                   │
│  Caffeine cache with TTL (default 5 min) and max entries        │
│  store(sessionId, file)  // store with default category         │
│  store(sessionId, category, file)  // store with category       │
│  retrieve(sessionId) → List<GeneratedFile>  // get and clear    │
│  retrieve(sessionId, category) → List<GeneratedFile>            │
│  hasArtifacts(sessionId) → boolean                              │
└─────────────────────────────────────────────────────────────────┘
```

## Key Classes

| Class | Purpose |
|-------|---------|
| `AgentCoreCodeInterpreterAutoConfiguration` | Spring Boot auto-config with `ToolCallbackProvider` |
| `AgentCoreCodeInterpreterClient` | Low-level SDK wrapper with configurable timeouts |
| `AgentCoreCodeInterpreterConfiguration` | Config properties (timeouts, TTL, identifier, description) |
| `CodeInterpreterTools` | Tool implementation with session context and optional category support |
| `CodeInterpreterArtifacts` | Helper for creating artifacts with metadata |
| `CodeExecutionResult` | Record for execution results with null-safe defaults |
| `GeneratedFile` | Record for file data with defensive copy and helper methods |
| `ExecuteCodeRequest` | Input schema record for the tool (language, code) |

## Design Decisions

1. **ToolCallbackProvider pattern** - Programmatic tool registration with configurable description
2. **File handling in ChatService** - Files appended after stream completes, outside memory flow
3. **Reactor context for session ID** - Session ID passed via `ToolCallReactiveContextHolder`, not `ToolContext`, to avoid leaking metadata to MCP tools
4. **No advisor** - Avoids files being stored in conversation memory (context overflow)
5. **Null-safe records** - `CodeExecutionResult` and `GeneratedFile` use defensive copies
6. **TTL-based cleanup** - Caffeine cache with 5-minute TTL prevents memory leaks
7. **Input validation** - Tool validates language and code parameters before execution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spring-ai-community/spring-ai-agentcore](https://github.com/spring-ai-community/spring-ai-agentcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
