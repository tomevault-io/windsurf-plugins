---
trigger: always_on
description: This file provides guidance to coding agents working with code in this repository.
---

# Repository Agent Notes

This file provides guidance to coding agents working with code in this repository.

## Project Overview

This is a **VSCode Language Server Extension** for **Beancount** (plain text accounting language). It provides rich language features including syntax highlighting, code completion, error detection, hover information, navigation, and code intelligence for Beancount files. The extension runs fully in browsers (VSCode Web, github.dev) without requiring Python or bean installation.

**Beancount Syntax Reference**: https://beancount.github.io/docs/beancount_language_syntax.html

## Architecture

### Monorepo Structure

The project uses a pnpm monorepo with Turbo for task orchestration:

- **packages/lsp-client**: VSCode extension client (UI/UX, configuration, client-side logic)
- **packages/lsp-server**: Core language server implementation with tree-sitter parsing
- **packages/shared**: Shared types, constants, and utilities
- **packages/storage**: Data persistence layer
- **packages/tree-sitter-beancount**: Custom tree-sitter parser written in Rust with WASM output

### Key Technical Decisions

- **Tree-sitter Integration**: Uses custom tree-sitter grammar (`tree-sitter-beancount`) for robust parsing of Beancount syntax
- **Dual Build Targets**: Supports both Node.js (desktop VSCode) and browser (VSCode Web) environments
- **Language Server Protocol**: Implements comprehensive LSP features (syntax highlighting, completion, diagnostics, hover, navigation, etc.)
- **AI/LLM Integration**: Includes language model tools for Beancount queries

## Development Commands

### Root Level Commands

```bash
# Install dependencies and initialize git hooks
pnpm install
git submodule update --init

# Build all packages
pnpm build

# Development watch mode
pnpm dev

# Run type checking, linting, and formatting checks
pnpm check

# Run tests across all packages
pnpm test

# Format code using dprint
pnpm format

# Build for npm publication
pnpm build:npm
```

### Package-Specific Commands

Each package has its own scripts. Key ones include:

**lsp-server**:

```bash
cd packages/lsp-server
pnpm build        # Build with VSCode client assets
pnpm build:npm    # Build for npm publication
pnpm watch        # Watch mode for development
pnpm test         # Run tests with Vitest
pnpm test:watch   # Run tests in watch mode
```

**lsp-client**:

```bash
cd packages/lsp-client
pnpm run vsix     # Package extension into .vsix file
```

### Pre-commit Hooks

The project uses `simple-git-hooks` with `nano-staged`:

- Automatically formats code with dprint
- Runs ESLint fixes
- Executes `pnpm check` before commit

## Code Organization

### LSP Server Architecture

- **Tree-sitter queries**: Located in `packages/lsp-server/src/common/language/queries/`
- **Symbol index**: Defined in `packages/lsp-server/src/common/features/symbol-index.ts`
- **Feature interface**: All LSP features must implement the `Feature` interface in `packages/lsp-server/src/common/features/types.ts`
- **Existing features**: Located in `packages/lsp-server/src/common/features/` folder

### LSP Client Architecture

- **Custom messages**: Defined in `packages/shared/src/messages.ts`
- **LLM tools**: Must implement `ToolImpl` interface in `packages/lsp-client/src/common/llm/tools/tool.ts`
- **Communication**: Client communicates with server through LSP or JSON RPC with custom messages

## Important Patterns and Rules

### Text Document Import (Critical)

When working with text documents in the LSP server:

```typescript
// DO NOT use this:
import { TextDocument } from 'vscode-languageserver';

// DO use this instead:
import { TextDocument } from 'vscode-languageserver-textdocument';
```

### Feature Implementation Guidelines

1. **Parsing and calculations should be done server-side**
2. **Reuse existing parsing and symbol index processes** when possible
3. **Improve existing implementations** rather than creating new ones
4. **When returning capabilities, always return something** on its handler

### Package Manager

- **Use pnpm instead of npm** for all package management operations

### Language and Comments

- **Use English in code and comments**

### Completion System Architecture

The completion system uses a **placeholder reparse** technique for context detection:

1. **Universal try strategy**: Instead of pre-determining contexts, define completion scenarios and try them sequentially
2. **Syntax tree validation**: The tree-sitter parser automatically rejects invalid placeholders
3. **Smart trigger handling**: Automatically removes trigger characters from placeholders if already typed
4. **Error detection**: Immediately rejects trees with syntax errors to avoid wasted processing

**Key completion scenarios** (in priority order):

1. Account completions (`Assets:Bank`) - works in postings, directives (note, balance, close, pad, document)
2. Tag completions (`#tag`) - works in tags_links, pushtag, poptag, transaction lines
3. Currency completions (`CNY`) - works in price_annotation, amount after numbers
4. Link completions (`^link`) - works in tags_links, transaction lines
5. Metadata key-value (`somekey: "value"`) - works in key_value contexts

### Default Tolerance Algorithm


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fengkx/beancount-lsp](https://github.com/fengkx/beancount-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
