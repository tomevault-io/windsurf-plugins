---
trigger: always_on
description: Obsidian Gemini Scribe is an Obsidian plugin that integrates Google's Gemini AI models for AI-driven assistance within Obsidian. It provides context-aware chat, document summarization, text rewriting, and IDE-style completions.
---

# Repository Guidelines

## Project Overview

Obsidian Gemini Scribe is an Obsidian plugin that integrates Google's Gemini AI models for AI-driven assistance within Obsidian. It provides context-aware chat, document summarization, text rewriting, and IDE-style completions.

## Project Structure & Module Organization

- `src/` contains TypeScript plugin code; `src/main.ts` is the entry point with domain folders such as `agent/`, `api/`, `tools/`, `ui/`, and `services/`, plus shared utilities in `utils/`.
- `docs/` hosts user and operator guides; `prompts/` ships default agent prompts; `test-scripts/` holds manual integration runners.
- Unit tests live in the `test/` directory mirroring `src/` structure as `*.test.ts`; generated artifacts (`main.js`, `manifest.json`, `styles.css`) stay in the repo root for Obsidian.

## Commands

### Development

```bash
npm install          # Install dependencies
npm run dev          # Development build with watch mode
npm run build        # Production build (generates refs, runs TypeScript check, then bundles)
npm run generate-refs # Regenerate help references from docs/ (runs automatically in build/dev)
npm test             # Run Vitest tests
npm run format       # Format code with Prettier
npm run format-check # Check formatting without changes
npm run lint         # Lint with ESLint (eslint-plugin-obsidianmd recommended preset)
npm run lint:fix     # Auto-fix ESLint violations where possible
npm run install:test-vault # Copy built artifacts into test vault (override path with TEST_VAULT_PLUGIN_DIR)
```

**IMPORTANT**: Always run `npm install` first if you encounter TypeScript errors or missing module errors during build. The build requires all dependencies in `node_modules` to be present. If you get errors about missing modules like `obsidian`, `@google/genai`, `handlebars`, or `tslib`, run `npm install` before attempting `npm run build` again.

### Testing

- Run single test: `npm test -- path/to/test.ts`
- Manual integration: `node test-scripts/test-sdk-tools.mjs` (and siblings) validate agent toolchains before shipping

### Versioning & Releases

For the full release process, use the **release-process** skill.

## Architecture

### Core Pattern: Factory + Decorator

```
src/main.ts → ModelClientFactory.createFromPlugin() → GeminiClient | OllamaClient → RetryDecorator → ModelApi
```

The plugin uses a factory pattern (`ModelClientFactory` in `src/api/factory.ts`) to create model API clients, wrapped with a retry decorator (`RetryDecorator`) for resilience. The factory branches on `settings.provider` to instantiate either a `GeminiClient` or an `OllamaClient`. All API implementations follow the `ModelApi` interface. The factory supports different use cases (chat, summary, completions, rewrite) and provides retry logic with exponential backoff for handling transient API failures. Each provider lives in its own package under `src/api/providers/{gemini,ollama}/`.

### Key Components

1. **API Layer** (`src/api/`): Factory pattern (`ModelClientFactory`) for creating provider-appropriate model API clients, decorator pattern (`RetryDecorator`) for resilience, and interface abstraction (`ModelApi`) for consistent API interactions. Provider-specific code is encapsulated under `src/api/providers/{gemini,ollama}/`.
2. **Feature Modules**: Separate modules for chat, completions (`completions.ts`), summary (`summary.ts`), and rewrite (`rewrite.ts`)
3. **Context System** (`src/files/file-context.ts`): Builds linked note trees for context-aware AI interactions
4. **History** (`src/history/`): Markdown-based conversation history with Handlebars templates for agent sessions, stored in `[state-folder]/Agent-Sessions/` (legacy note-centric chat history from v3.x remains in `[state-folder]/History/`)
5. **Custom Prompts** (`src/prompts/`): User-defined prompt templates stored in `[state-folder]/Prompts/`
6. **Agent Mode** (`src/agent/`, `src/tools/`): AI agent with tool calling capabilities
   - Session management with persistent history
   - Tool registry and execution engine
   - Vault operations tools with permission system
   - Google Search integration (separate from function calling)
   - Web fetch tool using Google's URL Context API
   - Session-level permission system for bypassing confirmations
   - Tool loop detection to prevent infinite execution cycles
   - MCP server integration with stdio and HTTP/SSE transports (`src/mcp/`)
   - Agent skills system for extensible AI capabilities (see below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allenhutchison/obsidian-gemini](https://github.com/allenhutchison/obsidian-gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
