---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference

**请使用中文回复所有对话和代码注释。** All responses, explanations, and code comments should be in Chinese (Simplified).

## Project Background

**当前状态**: 这是一个处于持续开发阶段的项目，代码库正在进行重构和优化。

## Skill 使用规范

- **页面开发或 UI 修改时**：使用 `frontend-design` skill
- **代码审查时**：使用 `code-review-excellence` skill

## Project Overview

A dual-mode AI application combining:
- **Chat Scene**: Traditional conversational AI interface with streaming responses
- **AI Editing Scene**: Rich text editor (Quill) with AI-powered text manipulation features

Built with Vue 3 + TypeScript + Vite, using Moonshot AI API for LLM capabilities.

## Development Commands

```bash
# Development server
bun dev

# Build (includes TypeScript type checking)
bun build

# Preview production build
bun preview

# Linting
bun lint
bun lint:fix

# Type checking
bun typecheck

# Testing
bun test              # Unit tests (watch mode)
bun test:run          # Unit tests (single run)
bun test:coverage     # Unit tests with coverage
bun test:ui           # Vitest UI
bun test:e2e          # E2E tests
bun test:e2e:ui       # Playwright UI
bun test:all          # Run all tests
```

**Package Manager**: This project uses `bun` (version 1.2.14 specified in package.json).

**Testing**: 详见 [TESTING.md](./TESTING.md) 查看完整测试指南。

## Architecture

### Scene Management

The app operates in two distinct scenes controlled by `services/appConfig.ts`:

```typescript
SCENES.CHAT // Traditional chat interface
SCENES.AI_EDITING // Rich text editor with AI features
```

Scene switching happens through `switchScene()` which updates `currentScene.value` and manages panel visibility.

### Data Flow Architecture

**Chat Scene Flow**:
1. User input → `addUserMessage()` in `services/chat.ts`
2. Empty AI message created with `isStreaming: true`
3. `useAI.generate()` → `useApi.generateChat()` → SSE stream processing
4. Incremental updates via `handleAiPartialResponse()`
5. Completion via `handleAiCompletion()` sets `isStreaming: false`

**AI Editing Flow**:
1. User selects text → `highlightSelection()` in `components/AIEditing/util.ts`
2. Prompt applied → `handleSend()` calls `AIEditingAPI.streamChat()`
3. SSE responses processed → `renderMarkdownToQuill()`
4. Optional Monaco diff editor for before/after comparison

### Database Layer (Dexie/IndexedDB)

Three tables defined in `services/database.ts`:
- `chats`: Chat sessions (id, name, model, createdAt)
- `messages`: Chat messages (id, chatId, role, content, imageUrl, isStreaming, createdAt)
- `config`: System prompts per model (id, model, systemPrompt, createdAt)

All database operations go through `dbLayer` abstraction in `services/chat.ts`.

### API Integration

**Provider**: Moonshot AI (https://api.moonshot.cn)

**Key Endpoints**:
- `POST /v1/chat/completions` - Streaming chat (SSE format)
- `GET /v1/models` - Available models list

**SSE Processing** (`api/api.ts:processStreamResponse`):
- Reads `data: [JSON]` lines from stream
- Extracts `choices[0].delta.content` for incremental text
- `finish_reason === 'stop'` triggers completion callback
- `data: [DONE]` signals stream end

### State Management

No Vuex/Pinia - uses composables pattern:
- `services/appConfig.ts`: Global config (uses @vueuse/core's `useLocalStorage`)
- `services/chat.ts`: Chat state and operations (exported refs)
- `services/useAI.ts`: AI generation logic

All state is reactive Vue refs/computed values.

### AI Editing Component Structure

Main entry: `components/AIEditing/index.vue` (large component ~1000+ lines)

**Key modules**:
- `api.ts`: AI editing API (uses `@microsoft/fetch-event-source`)
- `util.ts`: Core editing functions (highlight, diff, render, send)
- `export.ts`: Multi-format export (PDF via html2pdf.js, DOCX via docx, MD via turndown)
- `import.ts`: DOCX import (via mammoth)
- `markdown.ts`: Markdown rendering with KaTeX math support
- `monacoConfig.ts`: Monaco editor setup for diff view
- `storage.ts`: LocalStorage persistence for editor content

**Editor Stack**:
- Quill 2.0 for rich text (with quill-table-ui plugin)
- Monaco Editor for diff comparison
- markdown-it pipeline with plugins (katex, highlightjs, anchor, link-attributes)

### Image Support

Messages can include images via `imageUrl` field (Base64 or URL). API messages construct multi-modal content:

```typescript
content: [
  { type: 'image_url', image_url: { url } },
  { type: 'text', text }
]
```

### Dark Mode

Implemented via:
- `utils/darkMode.ts`: System theme sync + localStorage persistence
- Tailwind `dark:` variants throughout components
- CSS custom properties in `App.vue` (--message-bg-*, --text-color-*, etc.)

## Important Patterns

### Path Aliasing

`@/*` maps to `src/*` (configured in vite.config.ts and tsconfig.json).

### TypeScript Configuration

- Strict mode enabled
- `noUnusedLocals` and `noUnusedParameters` disabled
- Bundler module resolution
- `allowImportingTsExtensions` for .ts imports in Vue SFCs

### Vue SFC Structure

Mix of `<script setup>` (most components) and Options API (legacy). New code should use Composition API with `<script setup lang="ts">`.

### Error Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tt-a1i/chat_edit](https://github.com/tt-a1i/chat_edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
