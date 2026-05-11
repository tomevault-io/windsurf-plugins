---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

This is a VS Code extension that integrates LiteLLM into GitHub Copilot Chat, allowing users to access 100+ LLMs (OpenAI, Anthropic, Google, AWS, Azure, etc.) through a unified API. The extension implements VS Code's Language Model Chat Provider API to enable streaming chat completions with tool calling, multimodal input (images, PDFs), and thinking/reasoning support.

## Build and Development Commands

```bash
# Install dependencies
bun install

# Compile TypeScript to JavaScript
bun run compile

# Watch mode for development (auto-recompile on changes)
bun run watch

# Run linter
bun run lint

# Format code with Prettier
bun run format

# Run all tests
bun test

# Bump version (updates package.json and CHANGELOG.md)
bun run bump-version
```

## Development Workflow

### Testing the Extension

Press `F5` to launch the Extension Development Host with the extension loaded.

### Running Tests

Tests use the `@vscode/test-electron` framework. Run `bun test` to execute all tests, which compiles the project and runs the test suite.

### Code Style

- **Linting**: ESLint with TypeScript rules
- **Formatting**: Prettier with tabs (width 2), semicolons, 120 character line width
- **Pre-commit hooks**: Husky runs Prettier on staged files via lint-staged

### Git Commit and PR Conventions

- **DO NOT** add "Co-Authored-By:" or similar attribution lines to commit messages
- **DO NOT** add "Generated with" or similar markers to pull request descriptions
- Keep commit messages and PR descriptions clean and focused on the actual changes

## Architecture

### Core Components

**`src/extension.ts`**: Extension activation and lifecycle
- Registers the LiteLLM chat provider with vendor ID `"litellm"`
- Implements the `litellm.manage` command for configuration UI
- Manages status bar indicator showing connection state with 4 states:
  - Not configured: `$(warning) LiteLLM`
  - Loading: `$(loading~spin) LiteLLM`
  - Connected: `$(check) LiteLLM (N)` where N is model count
  - Error: `$(error) LiteLLM` with error details in tooltip
- Creates "LiteLLM" output channel for diagnostic logging
- Implements `litellm.testConnection` command to verify server connectivity
- Implements `litellm.showDiagnostics` command to display configuration and connection status
- Stores credentials securely in VS Code's SecretStorage (keys: `litellm.baseUrl`, `litellm.apiKey`)
- Persists connection status in `globalState` across sessions

**`src/provider.ts`**: Main provider implementation (`LiteLLMChatModelProvider`)
- Implements VS Code's `LanguageModelChatProvider` interface
- Fetches available models from LiteLLM's `/v1/model/info` endpoint (with fallback to `/v1/models`)
- Handles streaming chat completions via `/v1/chat/completions`
- Converts VS Code message format to OpenAI-compatible format
- Parses streaming SSE responses and emits parts (text, tool calls, thinking/reasoning)
- Manages tool call buffering and deduplication
- Tracks prompt caching support per model from `/v1/model/info`
- Captures extended model metadata (supports_response_schema, supports_reasoning, supports_pdf_input, supported_openai_params)
- Broad model options pass-through: all `modelOptions` and `modelParameters` keys forwarded to LiteLLM except provider-owned fields (`model`, `messages`, `stream`, `stream_options`, `tools`, `tool_choice`)
- Requests `stream_options: { include_usage: true }` by default and logs token usage from the final streaming chunk
- Token estimation: text-only for the hard rejection path (no multimodal guesses); `provideTokenCount()` uses rough heuristics for images/PDFs for VS Code UI hints
- Provides status callback mechanism to update extension about fetch results
- Logs all operations to output channel for debugging
- Shows notifications for:
  - Missing configuration (one-time per session)
  - Empty model list from server
  - Connection errors with actionable buttons

**`src/utils.ts`**: Conversion and validation utilities
- `convertMessages()`: Transforms VS Code messages to OpenAI format, including multimodal content:
  - `LanguageModelTextPart` → text content (string or text block)
  - `LanguageModelDataPart` with image MIME (png/jpeg/gif/webp) → `image_url` content block with base64 data URL
  - `LanguageModelDataPart` with `application/pdf` → `file` content block with base64 data URL
  - `LanguageModelDataPart` with text/JSON MIME → decoded as text
  - `LanguageModelPromptTsxPart` → text extraction
  - Unknown binary MIME types → logged and skipped
  - Preserves interleaved ordering of text/image/file parts
  - User messages with non-text parts use array content format; text-only messages use string format
- `convertTools()`: Transforms VS Code tool definitions to OpenAI function definitions
  - `ToolMode.Required` with single tool → named function choice
  - `ToolMode.Required` with multiple tools → `tool_choice: "required"` (no longer throws)
- `sanitizeSchema()`: Sanitizes JSON schemas while preserving provider-compatible features:
  - Preserves composite schemas (`anyOf`/`oneOf`/`allOf`) — recursively sanitizes branches instead of collapsing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vivswan/litellm-vscode-chat](https://github.com/Vivswan/litellm-vscode-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
