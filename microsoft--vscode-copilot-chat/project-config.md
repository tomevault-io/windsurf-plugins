---
trigger: always_on
description: This is the **GitHub Copilot Chat** extension for Visual Studio Code - a VS Code extension that provides conversational AI assistance, a coding agent with many tools, inline editing capabilities, and advanced AI-powered features for VS Code.
---

# GitHub Copilot Chat Extension - Copilot Instructions

## Project Overview

This is the **GitHub Copilot Chat** extension for Visual Studio Code - a VS Code extension that provides conversational AI assistance, a coding agent with many tools, inline editing capabilities, and advanced AI-powered features for VS Code.

### Key Features
- **Chat Interface**: Conversational AI assistance with chat participants, variables, and slash commands
- **Inline Chat**: AI-powered editing directly in the editor with `Ctrl+I`
- **Agent Mode**: Multi-step autonomous coding tasks
- **Edit Mode**: Natural language to code
- **Inline Suggestions**: Next edit suggestions and inline completions
- **Language Model Integration**: Support for multiple AI models (GPT-4, Claude, Gemini, etc.)
- **Context-Aware**: Workspace understanding, semantic search, and code analysis

### Tech Stack
- **TypeScript**: Primary language (follows VS Code coding standards)
- **TSX**: Prompts are built using the @vscode/prompt-tsx library
- **Node.js**: Runtime for extension host and language server features
- **WebAssembly**: For performance-critical parsing and tokenization
- **VS Code Extension API**: Extensive use of proposed APIs for chat, language models, and editing
- **ESBuild**: Bundling and compilation
- **Vitest**: Unit testing framework
- **Python**: For notebooks integration and ML evaluation scripts

## Validating changes

You MUST check compilation output before running ANY script or declaring work complete!

1. **ALWAYS** check the `start-watch-tasks` watch task output for compilation errors
2. **NEVER** use the `compile` task as a way to check if everything is working properly
3. **FIX** all compilation errors before moving forward

### TypeScript compilation steps
- Monitor the `start-watch-tasks` task outputs for real-time compilation errors as you make changes
- This task runs `npm: watch:tsc-extension`,`npm: watch:tsc-extension-web`, `npm: watch:tsc-simulation-workbench`, and `npm: watch:esbuild` to incrementally compile the project
- Start the task if it's not already running in the background

## Project Architecture

### Top-Level Directory Structure

#### Core Source Code (`src/`)
- **`src/extension/`**: Main extension implementation, organized by feature
- **`src/platform/`**: Shared platform services and utilities
- **`src/util/`**: Common utilities, VS Code API abstractions, and service infrastructure

#### Build & Configuration
- **`.esbuild.ts`**: Build configuration for bundling extension, web worker, and simulation workbench
- **`tsconfig.json`**: TypeScript configuration extending base config with React JSX settings
- **`vite.config.ts`**: Test configuration for Vitest unit tests
- **`package.json`**: Extension manifest with VS Code contributions, dependencies, and scripts

#### Testing & Simulation
- **`test/`**: Comprehensive test suite including unit, integration, and simulation tests
- **`script/simulate.sh`**: Test runner for scenario-based testing
- **`notebooks/`**: Jupyter notebooks for performance analysis and ML experiments

#### Assets & Documentation
- **`assets/`**: Icons, fonts, and visual resources
- **`CONTRIBUTING.md`**: Architecture documentation and development guide

### Key Source Directories

#### `src/extension/` - Feature Implementation

**Core Chat & Conversation Features:**
- **`conversation/`**: Chat participants, agents, and conversation flow orchestration
- **`inlineChat/`**: Inline editing features (`Ctrl+I`) and hints system
- **`inlineEdits/`**: Advanced inline editing capabilities with streaming edits

**Context & Intelligence:**
- **`context/`**: Context resolution for code understanding and workspace analysis
- **`contextKeys/`**: VS Code context key management for UI state
- **`intents/`**: Chat participant/slash command implementations
- **`prompts/`**: Prompt engineering and template system
- **`prompt/`**: Common prompt utilities
- **`typescriptContext/`**: TypeScript-specific context and analysis

**Search & Discovery:**
- **`search/`**: General search functionality within the extension
- **`workspaceChunkSearch/`**: Chunked workspace search for large codebases
- **`workspaceSemanticSearch/`**: Semantic search across workspace content
- **`workspaceRecorder/`**: Recording and tracking workspace interactions

**Authentication & Configuration:**
- **`authentication/`**: GitHub authentication and token management
- **`configuration/`**: Settings and configuration management
- **`byok/`**: Bring Your Own Key (BYOK) functionality for custom API keys

**AI Integration & Endpoints:**
- **`endpoint/`**: AI service endpoints and model selection
- **`tools/`**: Language model tools and integrations
- **`api/`**: Core API abstractions and interfaces
- **`mcp/`**: Model Context Protocol integration

**Development & Testing:**
- **`testing/`**: Test generation and execution features
- **`test/`**: Extension-specific test utilities and helpers

**User Interface & Experience:**
- **`commands/`**: Service for working with VS Code commands
- **`codeBlocks/`**: Streaming code block processing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-copilot-chat](https://github.com/microsoft/vscode-copilot-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
