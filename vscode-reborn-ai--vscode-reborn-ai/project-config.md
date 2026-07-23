---
trigger: always_on
description: This file provides specific guidelines for GitHub Copilot to ensure consistent, maintainable code generation for the VSCode Reborn AI extension.
---

# GitHub Copilot Instructions for VSCode Reborn AI Extension

This file provides specific guidelines for GitHub Copilot to ensure consistent, maintainable code generation for the VSCode Reborn AI extension.

## Project Overview

VSCode Reborn AI is a VS Code extension that provides AI-powered coding assistance through a webview interface. The extension uses a dual-process architecture with a backend (Node.js/VS Code Extension API) and frontend (React-based webview).

## Architecture & Communication Patterns

### Dual-Process Architecture
- **Backend Process**: Node.js extension host (`src/main.ts`, `src/entry-point.ts`)
- **Frontend Process**: React webview (`src/renderer/`)
- **Communication**: Message-based IPC between backend and frontend via `webview.postMessage()`

### Message Communication Pattern
**Always follow the established message pattern**:

1. **Backend Message Types**: Defined in `src/renderer/types-messages.ts` with `BackendMessageType` enum
2. **Frontend Message Types**: Defined in `src/renderer/types-messages.ts` with `FrontendMessageType` enum
3. **Message Handlers**: Backend handles messages in `src/main.ts`, frontend in `src/renderer/message-handler.ts`
4. **Messenger Classes**: Use `src/send-to-frontend.ts` (backend) and `src/renderer/send-to-backend.ts` (frontend)

**Example Message Flow**:
```typescript
// Frontend sending to backend
sendMessageToBackend(BackendMessageType.addFreeTextQuestion, {
  conversation,
  question,
  includeEditorSelection
} as AddFreeTextQuestionMessage);

// Backend responding to frontend
this.frontendMessenger.sendAddMessage(chatMessage, conversationId);
```

### State Management Architecture
- **Redux Toolkit**: All frontend state managed through Redux store (`src/renderer/store/`)
- **Three Store Slices**:
  - `app.ts`: Extension settings, API status, models, translations
  - `conversation.ts`: Chat conversations and messages
  - `action.ts`: Smart actions (README generation, gitignore creation)
- **Typed Hooks**: Always use `useAppDispatch` and `useAppSelector` from `src/renderer/hooks.ts`

## File Structure & Naming Conventions

### Core Structure
```
src/
├── entry-point.ts          # Extension activation entry point
├── main.ts                 # Main backend controller (ChatGptViewProvider)
├── helpers.ts              # Backend utility functions
├── openai-api-provider.ts  # AI API abstraction layer
├── smart-action-runner.ts  # Complex AI-powered actions
├── send-to-frontend.ts     # Backend → Frontend messaging
├── renderer/               # React frontend code
│   ├── index.tsx          # React app entry point
│   ├── layout.tsx         # Main layout component
│   ├── hooks.ts           # Typed Redux hooks
│   ├── store.ts           # Redux store configuration
│   ├── types.ts           # Frontend type definitions
│   ├── types-messages.ts  # IPC message type definitions
│   ├── components/        # React components
│   ├── store/            # Redux slices
│   └── views/            # Page-level components
```

### File Naming
- **Components**: PascalCase (e.g., `ChatMessage.tsx`, `ModelSelect.tsx`)
- **Utilities**: kebab-case (e.g., `send-to-backend.ts`, `smart-action-runner.ts`)
- **Views**: lowercase (e.g., `chat.tsx`, `api.tsx`)

## Component & Code Patterns

### React Component Structure
```typescript
// Always use this pattern for components
import React from "react";
import { useAppDispatch, useAppSelector } from "../hooks";
import { RootState } from "../store";

export default function ComponentName({
  prop1,
  prop2,
}: {
  prop1: string;
  prop2: Conversation;
}) {
  const dispatch = useAppDispatch();
  const someState = useAppSelector((state: RootState) => state.app.someValue);

  // Component logic here

  return (
    <div>
      {/* JSX here */}
    </div>
  );
}
```

### Backend Extension Pattern
- **Main Provider**: Extend `vscode.WebviewViewProvider` (see `src/main.ts`)
- **Message Handling**: Use switch statements with message types
- **Error Handling**: Always wrap async operations in try-catch
- **Configuration Access**: Use `vscode.workspace.getConfiguration('chatgpt')`

### Smart Actions Pattern
Smart actions are complex AI-powered operations that follow this pattern:
```typescript
class MyAction extends Action {
  public async run(
    systemContext: string,
    controller: AbortController
  ): Promise<void> {
    // 1. Validate workspace/requirements
    // 2. Gather necessary data
    // 3. Create AI prompt
    // 4. Stream AI response to file
    // 5. Handle completion/errors
  }
}
```

## API & Data Management

### OpenAI API Integration
- **Provider Class**: `ApiProvider` in `src/openai-api-provider.ts`
- **Model Support**: Supports OpenAI, Azure OpenAI, and OpenAI-compatible APIs
- **Streaming**: Uses `ai` SDK for streaming responses
- **Token Counting**: Uses `js-tiktoken` for accurate token estimation

### Configuration Management
- **Extension Settings**: Defined in `package.json` contributes.configuration
- **Type Safety**: Settings interface in `src/renderer/types.ts` must match `package.json`
- **Defaults**: Use `DEFAULT_EXTENSION_SETTINGS` constant for fallbacks

### Model Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vscode-reborn-ai/vscode-reborn-ai](https://github.com/vscode-reborn-ai/vscode-reborn-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
