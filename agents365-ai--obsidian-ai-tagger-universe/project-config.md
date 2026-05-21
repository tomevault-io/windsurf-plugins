---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Skills

### obsidian-plugin-dev
**Use for Obsidian API and plugin development.** Contains:
- Plugin API reference and TypeScript definitions
- UI components (Modal, Setting, ItemView, etc.)
- CSS variables for theming
- Best practices for plugin development

### frontend-design
**Use for UI/UX design and styling.** Creates distinctive, production-grade interfaces with high design quality. Use when building modals, views, or improving visual components.

## Language Requirements

**All code comments and commit messages MUST be in English.**

- Write all code comments in English only
- Write all git commit messages in English only
- Variable names, function names, and documentation should be in English
- The only exception is i18n translation strings (which contain Chinese translations)

## Build Commands

```bash
# Development build with watch mode and inline sourcemaps
npm run dev

# Production build (type-checks, then bundles)
npm run build

# Version bump (updates manifest.json and versions.json)
npm run version
```

The build process uses esbuild to bundle `src/main.ts` into `main.js`. Production builds disable sourcemaps; dev builds enable inline sourcemaps.

## Architecture Overview

### Core Plugin Structure

**Entry Point**: `src/main.ts` (`AITaggerPlugin` class)
- Main plugin class extending Obsidian's `Plugin`
- Manages lifecycle: settings loading, LLM service initialization, command registration
- Handles tag operations: `analyzeAndTagNote()`, `showTagNetwork()`, batch processing
- Central coordinator between services, UI, and Obsidian API

### Service Layer Architecture

**LLM Services** (`src/services/`)
- **Base abstractions**: `LLMService` interface defines contract for all providers
- **Two service types**:
  - `LocalLLMService`: Ollama, LM Studio, LocalAI, OpenAI-compatible endpoints
  - `CloudLLMService`: Cloud providers (OpenAI, Claude, Gemini, Groq, etc.)
- **Adapter pattern** (`src/services/adapters/`): Each cloud provider has its own adapter (e.g., `claudeAdapter.ts`, `geminiAdapter.ts`) handling API-specific formatting
- **Prompt engineering** (`src/services/prompts/tagPrompts.ts`): XML-structured prompts optimized for Claude/GPT with clear task/requirements/output sections

**Key service flow**:
1. Plugin calls `llmService.analyzeTags(content, candidateTags, mode, maxTags, language)`
2. Service builds prompt via `buildTagPrompt()` with mode-specific instructions
3. For cloud: Adapter formats request → calls API → parses response
4. Returns `LLMResponse` with `suggestedTags` and `matchedExistingTags`

### Tagging Modes

Four distinct modes in `TaggingMode` enum:
- **GenerateNew**: AI creates entirely new tags from content
- **PredefinedTags**: AI selects only from existing vault/file tags
- **Hybrid**: Combines both (generates new + matches existing)
- **Custom**: User-defined prompt with custom instructions

Mode selection affects prompt structure and tag merging logic in `analyzeAndTagNote()`.

### Settings & Configuration

**Settings schema** (`src/core/settings.ts`):
- `AITaggerSettings` interface with 20+ configuration options
- Key settings: `serviceType`, `taggingMode`, `cloudServiceType`, `interfaceLanguage`
- Settings UI split into modular sections (`src/ui/settings/`):
  - `LLMSettingsSection`: Service provider configuration
  - `TaggingSettingsSection`: Mode selection, tag limits, exclusions
  - `InterfaceSettingsSection`: Language selection (zh-cn/en)

**Settings persistence**: Loaded in `loadSettings()`, saved via `saveSettings()`, triggers service reinitialization.

### Internationalization (i18n)

**Translation system** (`src/i18n/`):
- Supported languages: English (`en.ts`) and Simplified Chinese (`zh-cn.ts`)
- Type-safe translations via `Translations` interface
- Access translations: `this.t.settings.someKey` or `plugin.t.messages.someMessage`
- Language switch requires Obsidian restart to update all UI elements

**Adding new i18n strings**:
1. Add to `Translations` interface in `types.ts`
2. Implement in both `en.ts` and `zh-cn.ts`
3. Reference via `t.section.key` in code

### Tag Utilities & Operations

**Core utilities** (`src/utils/tagUtils.ts`):
- `TagUtils.formatTags()`: Sanitizes tags (removes prefixes, enforces kebab-case)
- `TagUtils.updateNoteTags()`: Modifies frontmatter YAML, handles merge vs replace
- `TagUtils.getAllTags()`: Extracts all tags from vault frontmatter
- `TagUtils.getTagsFromFile()`: Reads predefined tags from markdown file

**Tag formatting rules**:
- Remove `#` prefix and malformed prefixes (`tag:`, `matchedExistingTags-`, etc.)
- Convert to kebab-case (spaces/special chars → hyphens)
- Preserve `/` for nested tags (e.g., `science/biology`)

**Tag operations** (`src/utils/tagOperations.ts`):
- Batch processing with progress notifications
- Handles file reading, content analysis, frontmatter updates

### Tag Network Visualization

**Implementation** (`src/ui/views/TagNetworkView.ts`):
- Custom Obsidian `ItemView` for graph visualization
- Dynamically loads D3.js v7 from CDN
- Network data built by `TagNetworkManager` (`src/utils/tagNetworkUtils.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agents365-ai/obsidian-ai-tagger-universe](https://github.com/Agents365-ai/obsidian-ai-tagger-universe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
