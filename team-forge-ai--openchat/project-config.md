---
trigger: always_on
description: This document summarizes the structure, responsibilities, and key technologies used across `src/` to provide actionable context for future engineering tasks.
---


## OpenChat src/ architecture overview

This document summarizes the structure, responsibilities, and key technologies used across `src/` to provide actionable context for future engineering tasks.

### Key technologies

- **React + TypeScript**: Functional components with hooks, strong typing
- **Tailwind CSS**: Global theme tokens and utilities in `src/App.css`
- **shadcn/ui + Radix Primitives**: Accessible, headless UI primitives in `src/components/ui/*`
- **TanStack Query**: Server-state query/mutation and cache invalidation for conversations/messages
- **ReactMarkdown + remark/rehype + KaTeX + Shiki**: Markdown rendering with code highlighting and custom elements
- **Lucide Icons**: `lucide-react` icons
- **Tauri + @tauri-apps/plugin-sql**: Desktop shell and SQLite DB access

## App entry and layout

- `src/app.tsx`
  - Top-level composition and providers: `MLXServerProvider`, `ConversationProvider`, and `SidebarProvider`.
  - App shell: header + left sidebar + chat window.
- `src/App.css`
  - Tailwind setup (`@import 'tailwindcss'`), custom CSS variables (OKLCH-based palette), dark mode tokens, base resets.
  - Global UX details: disable rubber-banding, cursor defaults, animated “aurora” background used by empty chat state.

## Context providers

- `src/contexts/conversation-context.tsx`
  - Holds `selectedConversationId` and setter for cross-app selection.
- `src/contexts/mlx-server-context.tsx`
  - Wires to `mlxServer` service. Exposes `status`, `error`, and `restartServer()`.
  - Initializes server event listeners, fetches initial status, keeps state updated.
- `src/contexts/theme-context.tsx`
  - Simple theme manager (`light`/`dark`/`system`) toggling root class.

## App shell and sidebar

- `src/components/app-header.tsx`
  - Top bar with sidebar trigger and `MLXServerStatus` indicator.
- `src/components/app-sidebar.tsx`
  - Wraps header, conversations, and footer inside `Sidebar` UI primitive.
- `src/components/app-sidebar/*`
  - `app-sidebar-header.tsx`: “New chat” button (uses conversations hook).
  - `app-sidebar-conversations.tsx`: Search input and conversation list; integrates `useConversations` and `useConversation` for selection and deletion.
  - `app-sidebar-conversation-item.tsx`: Single row with contextual menu, share/delete actions, confirmation dialog.
  - `app-sidebar-footer.tsx`: Placeholder settings button.

## Chat experience

- `src/components/chat-window.tsx`
  - Orchestrates chat: gets selected conversation, messages, and MLX status.
  - On submit: lazily creates a conversation if needed, then streams a response.
- `src/components/chat-input.tsx`
  - Textarea with dynamic height and roundedness, Enter-to-send (Shift+Enter for newline), send/plus actions, loading state.
- `src/components/chat-messages-list.tsx`
  - Virtual list container with auto-scroll-to-bottom behavior; shows `ChatEmptyState` when appropriate.
- `src/components/chat-message.tsx`
  - Renders user vs assistant bubbles, timestamp, copy-to-clipboard button.
  - Assistant reasoning (if present) via `ReasoningDisplay`.
- `src/components/chat-error-banner.tsx`
  - Inline error surface for failed send/stream scenarios.
- `src/components/chat-empty-state.tsx`
  - Centered welcome with animated aurora background overlay.
- `src/components/reasoning-display.tsx`
  - Collapsible “Reasoning” panel with approximate token count; supports loading state.

## Markdown rendering system

- `src/components/markdown/markdown.tsx`
  - ReactMarkdown root configured with `remarkGfm`, `remarkMath`, `rehypeRaw`, `rehypeKatex`, and custom `rehypeMarkCodeBlocks`.
  - Applies typography-tailwind classes and code block overflow rules.
- `src/components/markdown/components.tsx`
  - Maps markdown elements to React components:
    - `code`: Distinguishes inline vs block; uses `CodeBlock` for blocks.
    - `img`: Upgrades to `MarkdownImage` with modal preview and download.
    - `email-artifact` and `quick-reply`: Custom elements; `QuickReply` accepts optional `onSend` handler.
- `src/components/code-block.tsx`
  - Uses Shiki to convert code to HAST, then `hast-util-to-jsx-runtime` for SSR-friendly rendering. Includes `CopyButton`.
- `src/components/markdown/rehype-plugins.ts`
  - `rehypeMarkCodeBlocks`: Flags `code` nodes as inline or block based on parent `pre`.
- `src/components/markdown/email-artifact.tsx`
  - Shows email preview with “Open in Email Client” (mailto) action.
- `src/components/markdown/markdown-image.tsx`
  - Click-to-zoom image in Dialog, toolbar actions for download and close.
- `src/components/markdown/quick-reply.tsx`
  - Renders a button that can invoke an injected `onSend(text)` callback.
- `src/components/markdown/utils.ts`, `types.ts`
  - Helpers (extract text, determine language class) and stronger node typings.

## UI primitives (shadcn/ui over Radix)

- Buttons/inputs: `ui/button.tsx`, `ui/input.tsx`
- Overlay and feedback: `ui/dialog.tsx`, `ui/alert-dialog.tsx`, `ui/dropdown-menu.tsx`, `ui/tooltip.tsx`, `ui/skeleton.tsx`, `ui/collapsible.tsx`, `ui/dialog-ext.tsx`
- Sidebar system: `ui/sidebar.tsx`
  - Full responsive off-canvas/icon modes, keyboard shortcut (Cmd/Ctrl+B), mobile Sheet integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-forge-ai/openchat](https://github.com/team-forge-ai/openchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
