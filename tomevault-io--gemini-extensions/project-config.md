---
trigger: always_on
description: > This file provides guidance to all Coding Agents such as Claude Code (claude.ai/code), Codex when working with code in this repository.
---

## markdown-flow-ui

> This file provides guidance to all Coding Agents such as Claude Code (claude.ai/code), Codex when working with code in this repository.

# AGENTS.md

This file provides guidance to all Coding Agents such as Claude Code (claude.ai/code), Codex when working with code in this repository.

## Quick Start

### Most Common Tasks

| Task                     | Command                                | Location       |
| ------------------------ | -------------------------------------- | -------------- |
| Start development server | `npm run dev`                          | Root directory |
| Build library            | `npm run build`                        | Root directory |
| Run Storybook            | `npm run storybook`                    | Root directory |
| Run tests                | `npm test`                             | Root directory |
| Run linting              | `npm run lint`                         | Root directory |
| Format code              | `npm run format`                       | Root directory |
| Check code quality       | `npm run lint && npm run format:check` | Root directory |

### Essential Files and Directories

```bash
# Core library components
src/components/           # Main UI components
src/lib/                 # Utility functions
src/index.ts            # Main export file

# Documentation and examples
.storybook/             # Storybook configuration
src/**/*.stories.ts     # Component stories
README.md              # Project documentation

# Configuration
package.json           # Dependencies and scripts
tsconfig.json         # TypeScript configuration
eslint.config.mjs     # ESLint configuration
.prettierrc           # Prettier formatting rules
```

## Critical Warnings ⚠️

### MUST DO Before Any Commit

1. **Run lint and format checks**: `npm run lint && npm run format:check` (MANDATORY)
2. **Test your changes**: Run `npm test` and verify Storybook examples work
3. **Build the library**: Run `npm run build` to ensure no build errors
4. **Use English for all code**: Comments, variables, commit messages
5. **Follow Conventional Commits**: `type: description` (lowercase type, imperative mood)
6. **Update Storybook stories**: Add/update stories for new or modified components

### Common Pitfalls to Avoid

- **Never hardcode user-facing strings** - Use props and make components configurable
- **Don't skip linting/formatting** - Pre-commit hooks will catch these issues
- **Don't commit secrets** - No API keys or sensitive data in code
- **Don't use non-English in code** - English only (except for user-facing content examples)
- **Don't break existing APIs** - Maintain backward compatibility for public interfaces
- **Don't forget TypeScript types** - All public APIs must be properly typed

## Project Overview

markdown-flow-ui is a React UI library for rendering markdown with interactive flow components, typewriter effects, and plugin support. It provides components for creating dynamic, interactive markdown experiences with features like:

- Real-time markdown rendering with syntax highlighting
- Typewriter effect animations
- Interactive flow components with single-select and multi-select support
- Plugin system for custom components
- Server-Sent Events (SSE) support for streaming content
- Mermaid diagram rendering
- Mathematical expressions with KaTeX
- Internationalization (i18n) support for UI components

## Architecture

The project follows a component-based architecture with these main parts:

- **Core Components (`src/components/`)**: Main UI components for markdown rendering
- **Utility Functions (`src/lib/`)**: Helper functions and utilities
- **Plugin System**: Extensible architecture for custom markdown components
- **Storybook Integration**: Documentation and examples for all components

### Main Components

#### ContentRender (`src/components/ContentRender/`)

- **Purpose**: Core markdown rendering component with typewriter effects
- **Key Features**:
  - Markdown-to-HTML conversion with syntax highlighting
  - Typewriter animation support
  - Plugin system for custom components
  - Stream processing capabilities
  - Interactive variable system with single-select and multi-select support
  - Internationalization support for UI elements
- **Key Files**:
  - `ContentRender.tsx`: Main component implementation
  - `useTypewriter.ts`: Typewriter effect logic
  - `plugins/`: Custom component plugins (MermaidChart, CustomVariable)
  - `utils/`: Processing utilities

##### Interactive Variable System

The CustomVariable plugin supports various interaction modes:

**Single-Select Mode (using `|` separator):**

```markdown
Choose your role: ?[%{{role}}Developer|Designer|Manager]
```

**Multi-Select Mode (using `||` separator):**

```markdown
Select skills: ?[%{{skills}}React||Vue||Angular||Node.js]
```

**Mixed Mode (Multi-select + Text Input):**

```markdown
Choose technologies: ?[%{{tech}}Frontend||Backend||Mobile||...Other technologies]
```

**Props for Multi-Select Support:**

- `confirmButtonText`: Text for the confirm button (supports i18n)
- `selectedValues`: Array of selected values in callback
- `isMultiSelect`: Automatically detected from syntax


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/gemini-extensions](https://github.com/tomevault-io/gemini-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
