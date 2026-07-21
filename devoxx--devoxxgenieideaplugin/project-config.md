---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

DevoxxGenie is a Java-based LLM Code Assistant plugin for IntelliJ IDEA that integrates with both local LLM providers (Ollama, LMStudio, GPT4All, Llama.cpp, Exo) and cloud-based LLMs (OpenAI, Anthropic, Mistral, Groq, Gemini, DeepInfra, DeepSeek, OpenRouter, Azure OpenAI, Amazon Bedrock). The plugin supports advanced features like RAG (Retrieval-Augmented Generation), MCP (Model Context Protocol) servers, web search, and agentic programming capabilities.

## Build & Development Commands

### Building
```bash
./gradlew buildPlugin              # Build plugin (creates ZIP in build/distributions/)
./gradlew clean                    # Clean build artifacts
./gradlew shadowJar                # Create shadow JAR with dependencies
```

### Testing
```bash
./gradlew test                                    # Run all tests
./gradlew test --tests ClassName                  # Run specific test class
./gradlew test --tests ClassName.methodName      # Run single test method
./gradlew verifyPlugin                            # Verify plugin (includes tests)
```

### Running & Publishing
```bash
./gradlew runIde                   # Run IntelliJ IDEA with plugin for testing
./gradlew publishPlugin            # Publish to JetBrains Marketplace (requires PUBLISH_TOKEN env var)
```

### Task Automation (using Taskfile)
```bash
task build                         # Build the plugin
task test                          # Run tests
task run-ide                       # Run IDE with plugin
task generate-changelog VERSION=0.8.0  # Generate changelog from merged PRs
task preview-changes VERSION=0.8.0     # Preview changelog without committing
```

## Core Architecture

### Multi-Module Structure
- **Root module**: Main IntelliJ plugin code (`src/main/java/com/devoxx/genie/`)
- **Core module**: Shared utilities being refactored (see `core/README.md` - issue #564)
- **Docusaurus**: Documentation website (`docusaurus/`)

### Key Architectural Components

#### 1. Prompt Execution Flow
The plugin processes user prompts through a layered architecture:

**Entry Point**:
- `UserPromptPanel` → `PromptSubmissionListener.onPromptSubmitted()` → `PromptExecutionController.handlePromptSubmission()`

**Processing Layer**:
- `PromptExecutionService.executeQuery()` - Handles token calculations, RAG, and GitDiff settings
- `ChatPromptExecutor.executePrompt()` - Dispatches to appropriate LLM provider
- `LLMProviderService.getAvailableModelProviders()` - Retrieves model from ChatModelFactory

**Execution Strategies**:
- `StreamingPromptExecutor` - Token-by-token streaming responses
- `NonStreamingPromptExecutionService` - Full response mode
- `WebSearchPromptExecutionService` - Web search augmented prompts

**Response Rendering**:
- `ChatStreamingResponsePanel` - Real-time streaming UI updates
- `ChatResponsePanel` - Final response display with code highlighting
- `ResponseHeaderPanel`, `ResponseDocumentPanel`, `MetricExecutionInfoPanel` - Modular response components

#### 2. LLM Provider System
**Factory Pattern Implementation**:
- `ChatModelFactory` (interface) - Base factory for all providers
- `ChatModelFactoryProvider` - Provider registry and lookup
- Provider-specific factories under:
  - `chatmodel/cloud/` - Cloud providers (OpenAI, Anthropic, Gemini, etc.)
  - `chatmodel/local/` - Local providers (Ollama, GPT4All, LMStudio, etc.)

**Cloud Providers**: anthropic, azureopenai, bedrock, deepinfra, deepseek, google, grok, groq, mistral, openai, openrouter

**Local Providers**: ollama, gpt4all, lmstudio, llamacpp, jan, customopenai

**Adding New Providers**:
1. Create factory class implementing `ChatModelFactory` under `chatmodel/cloud/` or `chatmodel/local/`
2. Implement `createChatModel()` and `createStreamingChatModel()` methods
3. Register in `ChatModelFactoryProvider`
4. Add provider to `ModelProvider` enum in `model/enumarations/`

#### 3. RAG (Retrieval-Augmented Generation) System
**Components**:
- `ProjectIndexerService` - Indexes project files for semantic search
- `ChromaEmbeddingService` - Stores embeddings in ChromaDB (Docker-based, v0.6.2)
- `SemanticSearchService` - Retrieves relevant code based on similarity
- Uses Ollama with Nomic Text embeddings for vector generation
- `RAGValidatorService` - Validates Docker, ChromaDB, and Ollama setup

**Validators**:
- `DockerValidator` - Checks Docker availability
- `ChromeDBValidator` - Validates ChromaDB connection
- `OllamaValidator` - Verifies Ollama and embedding model
- `NomicEmbedTextValidator` - Checks nomic-embed-text model

#### 4. MCP (Model Context Protocol) Support
**Key Services**:
- `MCPService` - Core MCP server management
- `MCPExecutionService` - Executes MCP tool calls
- `MCPListenerService` - Implements ChatModelListener for MCP integration
- `MCPCallbackLogger` - Logs MCP requests/responses for debugging

**Configuration**:
- MCP servers configured in Settings UI (`ui/settings/mcp/`)
- Supports stdio and HTTP SSE transports
- Tools are automatically exposed to LLM conversations when MCP is enabled

#### 5. Service Layer Organization
Key services under `service/`:
- `ChatService` - Manages chat conversations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devoxx/DevoxxGenieIDEAPlugin](https://github.com/devoxx/DevoxxGenieIDEAPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
