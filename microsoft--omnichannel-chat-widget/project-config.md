---
trigger: always_on
description: **This workspace may contain up to 6 related repositories.** Not all teams have all repos. Always be aware of which repository you're in when making changes.
---

# omnichannel-chat-widget - Claude Code Instructions

## Repository Ecosystem

**This workspace may contain up to 6 related repositories.** Not all teams have all repos. Always be aware of which repository you're in when making changes.

| Repository | Type | Purpose | Typical Location |
|------------|------|---------|------------------|
| **CRM.Omnichannel** | Monorepo (Backend) | 20+ microservices for Omnichannel platform | `<workspace-root>/CRM.Omnichannel/` |
| **ConversationControl** | Frontend (Agent UI) | Agent experience and conversation management UI | `<workspace-root>/CRM.OmniChannel.ConversationControl/` |
| **LiveChatWidget** | Frontend (Customer) | Customer-facing chat widget | `<workspace-root>/CRM.OmniChannel.LiveChatWidget/` |
| **omnichannel-chat-sdk** | Public SDK | TypeScript SDK for chat integration | `<workspace-root>/omnichannel-chat-sdk/` |
| **omnichannel-chat-widget** | Public Components | React component library | `<workspace-root>/omnichannel-chat-widget/` |
| **omnichannel-amsclient** | Shared Library | File upload/download client | `<workspace-root>/omnichannel-amsclient/` |

---

## Quick Context
- **Purpose:** Shared React component library for building chat widgets
- **Type:** TypeScript/React Library (npm package)
- **Tech Stack:** TypeScript, React 17+, Rollup, Jest, Storybook
- **Distribution:** npm registry (@microsoft/omnichannel-chat-widget)
- **Consumers:** CRM.OmniChannel.LiveChatWidget, external customers

## Architecture Overview

**What is omnichannel-chat-widget?**

This is a React component library providing pre-built, customizable UI components for chat experiences. It includes message bubbles, input boxes, file attachments, typing indicators, and more. Components are theme-able and accessible (ARIA support).

**Key Features:**
- Pre-built React chat components
- Theme customization support
- Accessibility (ARIA attributes, keyboard navigation)
- TypeScript type definitions
- Storybook for component preview

**Monorepo Structure:**

This repo contains **2 npm packages**:
- **chat-components** - Base components (buttons, icons, primitives)
- **chat-widget** - Higher-level chat-specific components

**Integration:**
- **Peer dependency:** `omnichannel-chat-sdk` (for chat operations)
- **Peer dependency:** `react` and `react-dom` (17+ recommended)
- **Consumed by:** LiveChatWidget, external customers building custom widgets

---

## Build & Test Workflow

### Prerequisites
- Node.js (version in package.json engines)
- npm package manager

### Setup
```bash
cd omnichannel-chat-widget

# Install dependencies (both packages)
npm install
```

### Common Commands

**Build:**
- **Build all packages:** `npm run build` - Build both chat-components and chat-widget
- **Build specific package:**
  - `npm run build:chat-components`
  - `npm run build:chat-widget`
- **Watch mode:** `npm run watch` - Incremental development

**Test:**
- **Unit tests:** `npm test` - Jest tests for all packages
- **Test specific package:**
  - `npm run test:chat-components`
  - `npm run test:chat-widget`
- **Coverage:** `npm run coverage` - Test coverage report
- **Lint:** `npm run lint` - ESLint validation

**Storybook:**
- **Start Storybook:** `npm run storybook` - Component preview at localhost:6006
- **Build Storybook:** `npm run build-storybook` - Static site for deployment

**Release:**
- **Publish:** `npm run publish:packages` - Publish both packages to npm
- **Version bump:** Use lerna or npm workspaces for versioning

---

## Coding Standards

### TypeScript Best Practices

- **Avoid `any` type** - Use proper React prop types
- **Explicit prop types** - Use interfaces for component props
- **React Hooks** - Prefer functional components with hooks
- **Accessibility** - Always include ARIA attributes

**Example Component:**
```typescript
// ✅ CORRECT - Explicit prop types, accessibility
import React from 'react';

export interface MessageBubbleProps {
    message: string;
    sender: 'agent' | 'customer';
    timestamp: Date;
    onRetry?: () => void;
    className?: string;
}

export const MessageBubble: React.FC<MessageBubbleProps> = ({
    message,
    sender,
    timestamp,
    onRetry,
    className
}) => {
    return (
        <div
            className={`message-bubble ${sender} ${className || ''}`}
            role="article"
            aria-label={`Message from ${sender} at ${timestamp.toLocaleTimeString()}`}
        >
            <div className="message-content">{message}</div>
            <div className="message-timestamp" aria-hidden="true">
                {timestamp.toLocaleTimeString()}
            </div>
            {onRetry && (
                <button
                    onClick={onRetry}
                    aria-label="Retry sending message"
                >
                    Retry
                </button>
            )}
        </div>
    );
};
```

### Component Structure

**File organization:**
```
chat-widget/src/
├── components/
│   ├── MessageBubble/
│   │   ├── MessageBubble.tsx       # Component implementation
│   │   ├── MessageBubble.test.tsx  # Jest tests
│   │   ├── MessageBubble.stories.tsx # Storybook stories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/omnichannel-chat-widget](https://github.com/microsoft/omnichannel-chat-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
