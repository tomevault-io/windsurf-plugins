---
trigger: always_on
description: IntelliDraw is a Next.js-based AI-powered diagram creation platform that integrates draw.io editor with AI-assisted diagram generation through natural language. It supports real-time collaboration using Yjs CRDT, team spaces, diagram marketplace, and connects to backend AI services via SSE streaming.
---

# IntelliDraw - AI-Powered Collaborative Diagram Platform

## Project Overview

IntelliDraw is a Next.js-based AI-powered diagram creation platform that integrates draw.io editor with AI-assisted diagram generation through natural language. It supports real-time collaboration using Yjs CRDT, team spaces, diagram marketplace, and connects to backend AI services via SSE streaming.

**Live Demo**: https://www.intellidraw.top/

## Tech Stack

| Category | Technologies |
|----------|-------------|
| Framework | Next.js 16 (App Router), React 19 |
| Language | TypeScript 5 |
| Styling | Tailwind CSS 4, Ant Design 5, Radix UI, shadcn/ui |
| State Management | Redux Toolkit |
| Collaboration | Yjs CRDT, WebSocket (@hocuspocus/provider) |
| AI Integration | Backend SSE Streaming |
| i18n | next-intl |
| Linting | Biome 2.3.8 |
| Build | Husky, lint-staged |

## Project Structure

```
├── app/                    # Next.js App Router pages
│   ├── api/chat/stream/    # AI chat SSE route handler
│   ├── diagram/            # Diagram edit/view pages
│   ├── diagram-marketplace/ # Public diagram marketplace
│   ├── my-diagrams/        # User's diagrams
│   ├── my-spaces/          # Team spaces
│   ├── my-rooms/           # Collaboration rooms
│   ├── templates/          # Diagram templates (SEO)
│   ├── solutions/          # Solution pages (SEO)
│   ├── wiki/               # Wiki pages (SEO)
│   ├── user/               # Auth pages (login/register/profile)
│   ├── admin/              # Admin dashboard
│   ├── layout.tsx          # Root layout with i18n
│   ├── providers.tsx       # Redux, Antd, Diagram context providers
│   └── globals.css         # Global styles
│
├── components/             # React components
│   ├── chat-panel.tsx      # Main AI chat panel (core component)
│   ├── simple-chat-panel.tsx # Simplified chat for demo pages
│   ├── demo-chat-panel.tsx # Demo-specific chat panel
│   ├── collaboration-panel.tsx # Real-time collaboration UI
│   ├── DrawioHome.tsx      # Main draw.io integration component
│   ├── diagram-toolbar.tsx # Diagram toolbar actions
│   ├── settings-dialog.tsx # AI provider settings dialog
│   ├── save-dialog.tsx     # Save/export dialog
│   ├── download-dialog.tsx # Download options
│   ├── file-preview-list.tsx # Uploaded file previews
│   ├── chat-input.tsx      # Chat input with file upload
│   ├── chat-message-display.tsx # Message rendering
│   ├── chat-example-panel.tsx # Example prompts
│   ├── history-dialog.tsx  # Version history
│   ├── ai-config-dialog.tsx # AI model configuration
│   ├── thinking-block.tsx  # AI reasoning/thinking display
│   ├── code-block.tsx      # Code block with syntax highlighting
│   ├── ui/                 # shadcn/ui components (button, dialog, etc.)
│   ├── admin/              # Admin management components
│   ├── room/               # Room management components
│   └── user/               # User-related components
│
├── lib/                    # Utilities and hooks
│   ├── ai-config.ts        # AI config from localStorage (user-defined)
│   ├── utils.ts            # XML validation/formatting utilities
│   ├── use-websocket-collaboration.ts # WebSocket collaboration hook
│   ├── use-yjs-collaboration.ts # Yjs collaboration hook
│   ├── yjs-collab.ts       # Yjs document management
│   ├── collab-protocol.ts  # Collaboration protocol definitions
│   ├── collab-packet.ts    # Collaboration packet handling
│   ├── cryptoUtils.ts      # Encryption for collaboration
│   ├── use-persistence.ts  # Diagram persistence hook
│   ├── use-diagram-save.ts # Diagram save logic
│   ├── use-file-processor.tsx # File upload/processing
│   ├── use-backend-chat.ts # Backend API chat integration (SSE)
│   ├── use-quota-manager.tsx # Quota management
│   ├── pdf-utils.ts        # PDF extraction utilities
│   ├── token-counter.ts    # Token counting for AI
│   ├── storage.ts          # LocalStorage utilities
│   ├── cached-responses.ts # Response caching
│   ├── request.ts          # HTTP request utilities
│   └── seo-data.ts         # SEO metadata
│
├── contexts/               # React contexts
│   ├── diagram-context.tsx # Diagram state management context
│
├── api/                    # Backend API client definitions
│   ├── index.ts            # API client initialization
│   ├── typings.d.ts        # API type definitions
│   ├── aiClientController.ts # AI client management
│   ├── diagramController.ts # Diagram CRUD
│   ├── conversionController.ts # Diagram conversion/chat history
│   ├── roomController.ts   # Collaboration room
│   ├── roomMemberController.ts # Room membership
│   ├── spaceController.ts  # Team space management
│   ├── spaceUserController.ts # Space membership
│   ├── userController.ts   # User management
│   ├── materialController.ts # Material/assets
│   ├── feedBackController.ts # User feedback
│   ├── announcementController.ts # Announcements
│   ├── systemAdminController.ts # System admin
│   ├── fileController.ts   # File upload/management
│   ├── codeParser.ts       # Code parsing API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangfenghuan/ai-draw-io-fronted](https://github.com/wangfenghuan/ai-draw-io-fronted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
