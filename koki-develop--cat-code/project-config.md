---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

- **Build**: `bun run build` - Uses the custom build script in `scripts/build.ts`
- **Lint**: `bun run lint` - Runs Biome linter checks
- **Format**: `bun run fmt` - Formats code using Biome with unsafe fixes
- **Type check**: `bun run typecheck` - Runs TypeScript compiler without emitting files
- **Test locally**: `bun run ./dist/index.js` - Run the built CLI locally
- **Test safe mode**: `bun run ./dist/index.js --safe` - Run in safe mode (no file modifications)
- **Run tests**: `bun test` - Runs all test files (*.spec.ts)
- **Run single test**: `bun test src/lib/util.spec.ts` - Run specific test file
- **Test cat**: `bun test src/lib/cat.spec.ts` - Run cat vocabulary and emotion detection tests
- **Development**: `bun run src/index.tsx` - Run the app directly without building
- **Install globally**: `bun link` - Install as global `cat-code` command for testing
- **Pre-commit hooks**: Automatically runs linting and formatting via Husky

## Project Architecture

This is a terminal-based chat UI CLI tool that simulates a conversation with a cat. The architecture consists of:

### Core Components
- **CLI Entry Point** (`src/index.tsx`): Commander.js setup that renders the Ink app
- **Chat Application** (`src/App.tsx`): Main state management component for messages and loading state
- **UI Components** (`src/components/`): Modular React components for terminal UI
  - `ChatHistory.tsx`: Message list rendering using Ink's Static component
  - `MessageItem.tsx`: Individual message display component with multi-line indent support, user messages prefixed with "> ", cat messages with "⏺"
  - `InputField.tsx`: Custom text input with advanced terminal-style keybindings using `useInput` hook
  - `Spinner.tsx`: Loading indicator during cat responses
  - `types.ts`: Shared TypeScript interfaces
- **Business Logic** (`src/lib/`): Core functionality and utilities
  - `cat.ts`: Cat class with emotion-based response generation and vocabulary system
  - `util.ts`: Reusable utility functions (e.g., indent for text formatting)
- **Build System** (`scripts/build.ts`): Custom Bun build script that creates executable CLI binary
- **File Editor System** (`src/lib/fileEditor.ts`): Handles random file editing with cat sounds
- **Git Integration** (`src/lib/git.ts`): Manages git repository detection and file tracking

### Technology Stack
- **Runtime**: Bun (primary development runtime)
- **CLI Framework**: Commander.js for argument parsing
- **Terminal UI**: Ink (React for CLIs) with built-in hooks
  - `ink-spinner` for loading states
- **Build Tool**: Custom Bun build script that creates executable CLI binary
- **Linting/Formatting**: Biome (replaces ESLint + Prettier)
- **Package Manager**: Bun with lockfile management
- **Git Operations**: simple-git for repository interactions
- **File Pattern Matching**: glob for non-git file discovery
- **Terminal Styling**: chalk for ANSI color support

### InputField Keybindings
The custom InputField component supports comprehensive terminal-style keybindings:
- **Cursor Movement**: Arrow keys, Ctrl+b/f (left/right), Ctrl+n/p (up/down), Ctrl+a/e (home/end)
- **Text Editing**: Backspace/Delete (delete before cursor), Ctrl+d (delete at cursor), Ctrl+w (delete word), Ctrl+u (delete to line start), Ctrl+k (delete to line end), Ctrl+l (clear input)
- **Multi-line Support**: Full cursor navigation and editing across multiple lines
- **Ink Quirk**: Both `key.backspace` and `key.delete` events represent the physical backspace key

### Chat UI Design Patterns
- **Message Persistence**: Uses Ink's `<Static>` component to render chat history that doesn't re-render
- **Loading States**: Displays cyan spinner with "Thinking..." during cat response delay
- **Input Management**: Disables cursor and prevents duplicate submissions during loading
- **Cat Behavior**: Cat class provides contextual responses based on detected emotions, with random delay (300-1300ms) for realistic interaction
- **Message Styling**: User messages prefixed with "> " in gray, cat messages prefixed with "⏺ " in cyan
- **Multi-line Support**: Both user and cat messages use indent utility for proper 2+ line formatting
- **Exit Control**: Double Ctrl+C required within 3 seconds to exit, single Ctrl+C clears input and shows exit warning
- **Clean Exit**: Exit warning is hidden before termination using setTimeout to wait for React re-render

### Build Process
The build process creates a standalone executable CLI tool:
1. Removes existing `dist/` directory
2. Uses Bun.build() to bundle TypeScript/TSX to JavaScript
3. Adds Node.js shebang for CLI execution
4. Makes output executable with chmod +x

### Component Architecture
The codebase follows a modular component structure:
- **State Management**: Centralized in `App.tsx` using React hooks with Date.now() for unique message IDs
- **Component Separation**: Each UI piece is a separate component with clear responsibilities
- **Type Safety**: Shared types in `components/types.ts` for consistency
- **Prop Interface**: Components receive specific props rather than accessing global state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koki-develop/cat-code](https://github.com/koki-develop/cat-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
