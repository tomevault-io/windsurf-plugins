---
trigger: always_on
description: **reachat** is a React UI library for building chat/LLM experiences. It provides customizable, composable components for building chat interfaces with support for markdown rendering, rich text input with mentions and slash commands, file uploads, session management, and theming via Tailwind CSS.
---

# CLAUDE.md - AI Assistant Guide for reachat

## Project Overview

**reachat** is a React UI library for building chat/LLM experiences. It provides customizable, composable components for building chat interfaces with support for markdown rendering, rich text input with mentions and slash commands, file uploads, session management, and theming via Tailwind CSS.

- **Repository**: reaviz/reachat
- **License**: Apache-2.0
- **Package Manager**: pnpm (v9.5.0)
- **Documentation**: https://reachat.dev
- **Storybook**: https://storybook.reachat.dev

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 18+ | UI framework |
| TypeScript | 4.9.5 | Type safety |
| Tailwind CSS | 4.x | Styling |
| Vite | 5.x | Build tool & dev server |
| Storybook | 8.x | Component development |
| Vitest | 1.x | Testing |
| reablocks | 9.x | Base UI components |
| Tiptap | 3.x | Rich text editor framework |
| Floating UI | 0.27.x | Popup positioning |
| motion | 12.x | Animations |
| Zod | 3.x / 4.x | Runtime prop validation |

## Directory Structure

```
reachat/
├── src/                    # Source code
│   ├── index.ts           # Main entry point - exports all public APIs
│   ├── types.ts           # Core TypeScript interfaces
│   ├── theme.ts           # Theme system definitions
│   ├── Chat.tsx           # Root Chat component
│   ├── ChatContext.ts     # React context for chat state
│   ├── AppBar/            # App bar component
│   ├── ChatBubble/        # Chat bubble component
│   ├── ChatInput/         # Input field components
│   ├── ChatSuggestions/   # Suggestion chips component
│   ├── ComponentCatalog/  # Dynamic component rendering system
│   ├── Markdown/          # Markdown rendering (code, tables, etc.)
│   ├── MessageStatus/     # Loading/status indicators
│   ├── SessionMessages/   # Message display components
│   ├── SessionsList/      # Session list/grouping components
│   ├── utils/             # Utility functions
│   └── assets/            # SVG icons
├── stories/               # Storybook stories and examples
├── .storybook/            # Storybook configuration
├── dist/                  # Build output (generated)
└── scripts/               # Build scripts
```

## Quick Commands

```bash
# Install dependencies
npm install

# Start Storybook development server (port 9009)
npm start

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Build library for production
npm run build

# Lint code
npm run lint

# Fix lint issues
npm run lint:fix

# Format code with Prettier
npm run prettier

# Build Storybook for deployment
npm run build-storybook
```

## Core Concepts

### Component Architecture

The library uses a **composable slot-based architecture**. The main `Chat` component wraps children and provides context:

```tsx
<Chat sessions={sessions} activeSessionId={activeId}>
  <SessionMessagePanel>
    <SessionMessages />
    <ChatInput />
  </SessionMessagePanel>
</Chat>
```

### Key Data Types

```typescript
// Core data structures in src/types.ts
interface Session {
  id: string;
  title?: string;
  createdAt?: Date;
  updatedAt?: Date;
  conversations: Conversation[];
}

interface Conversation {
  id: string;
  createdAt: Date;
  question: string;
  response?: string;
  sources?: ConversationSource[];
  files?: ConversationFile[];
}
```

### View Types

Three view modes are supported:
- `console` - Full screen with sessions sidebar
- `companion` - Compact/mobile view
- `chat` - Chat only, no sessions list

### Theme System

The theme is defined in `src/theme.ts` using a typed object with Tailwind classes:

```typescript
const chatTheme: ChatTheme = {
  base: 'dark:text-white text-gray-500',
  console: 'flex w-full gap-4 h-full',
  // ... nested theme objects for each component
};
```

Components use the theme via `useComponentTheme` from reablocks:
```typescript
const theme = useComponentTheme<ChatTheme>('chat', customTheme);
```

### Rich Text Input Features

The library includes an advanced rich text input system built on **Tiptap v3** with support for mentions and slash commands.

#### RichTextInput Component

Located in `src/ChatInput/RichTextInput.tsx`, this component provides:

- **Auto-expanding textarea** with configurable min/max heights
- **Mentions support** - Trigger with `@` to mention users, files, or custom entities
- **Slash commands** - Trigger with `/` for quick actions
- **Custom keyboard handling** - Shift+Enter for multi-line, Enter to submit
- **Floating suggestions** - Smart popup positioning using Floating UI
- **Keyboard navigation** - Arrow keys, Enter/Tab to select, Escape to close

**Exposed Methods via Ref:**
```typescript
interface RichTextInputRef {
  focus: () => void;
  getValue: () => string;
  setValue: (value: string) => void;
  insertText: (text: string) => void;
}
```

**Usage Example:**
```tsx
<ChatInput
  mentions={{
    trigger: '@',
    items: [
      { id: '1', label: 'John Doe', description: 'Product Manager' },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reaviz/reachat](https://github.com/reaviz/reachat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
