---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Undocx is a **Google Docs-like realtime collaborative rich text editor** built with Next.js 16 (App Router), Lexical.dev, Supabase Realtime, and shadcn UI. The application provides a fully-featured document editing experience with real-time collaboration, advanced formatting, tables, images, and comments. The project includes two editor implementations: a basic markdown editor at `/editor-md` and a full-featured collaborative rich text editor at `/editor` with real-time synchronization powered by Supabase.

## Development Commands

### Running the Application

```bash
npm run dev          # Start Next.js development server on http://localhost:3000
npm run build        # Build production bundle
npm start            # Start production server
npm run lint         # Run ESLint
```

### Supabase Local Development

```bash
npm run supabase     # Access Supabase CLI
```

**Important Supabase Ports (Local Development):**

- API: `http://127.0.0.1:54321`
- Studio: `http://127.0.0.1:54323`
- Database: `localhost:54322`
- Inbucket (Email testing): `http://127.0.0.1:54324`

Environment variables required:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_OR_ANON_KEY`

## Architecture

### Tech Stack

- **Frontend Framework**: Next.js 16 (App Router) with React
- **Rich Text Editor**: Lexical.dev - Extensible text editor framework
- **UI Components**: shadcn UI (Radix UI primitives + Tailwind CSS)
- **Backend & Realtime**: Supabase (Authentication, Realtime Database, Storage)
- **Styling**: Tailwind CSS with CSS variables for theming
- **TypeScript**: Full type safety across the application

### Authentication & Middleware

The app uses a custom middleware pattern with Supabase SSR:

1. **Middleware Proxy Pattern**: Instead of a traditional `middleware.ts`, this project uses `proxy.ts` with the same export config. The middleware refreshes Supabase sessions and redirects unauthenticated users to `/auth/login`.

2. **Supabase Client Patterns**:
   - **Server Components**: Use `lib/supabase/server.ts` - creates a new client per request (required for Fluid compute)
   - **Client Components**: Use `lib/supabase/client.ts`
   - **Middleware**: Uses inline `createServerClient` in `lib/supabase/middleware.ts`

**CRITICAL**: When working with Supabase server client:

- Never store the client in a global variable
- Always create a new client within each function
- Do not run code between `createServerClient` and `supabase.auth.getClaims()` as this can cause random logouts

### Editor Architecture (Lexical)

The rich text editor (`/editor`) is built with a plugin-based architecture using Lexical. **All Lexical editor-related components are organized under `components/editor/`** in a structured, modular way.

**Core Editor Wrapper:**

- `components/blocks/editor-x/editor.tsx` - Main editor wrapper (LexicalComposer)
- `components/blocks/editor-x/editor-header.tsx` - Editor header with document actions
- `components/blocks/editor-x/plugins.tsx` - Plugin orchestration and layout
- `components/blocks/editor-x/nodes.ts` - Custom node registrations

**Editor Directory Structure (`components/editor/`):**

```
components/editor/
├── context/              # Shared state management
│   └── toolbar-context.tsx    - ToolbarContext for plugin communication
├── editor-hooks/         # Custom React hooks
│   ├── use-debounce.ts        - Debouncing utility
│   ├── use-modal.tsx          - Modal state management
│   └── use-update-toolbar.ts  - Toolbar state updates
├── editor-ui/            # Reusable UI components
│   ├── code-button.tsx        - Code language selector
│   ├── color-picker.tsx       - Color selection UI
│   ├── content-editable.tsx   - Editable content wrapper
│   ├── image-component.tsx    - Image display component
│   └── image-resizer.tsx      - Image resize handles
├── nodes/                # Custom Lexical nodes
│   └── image-node.tsx         - Image node implementation
├── plugins/              # Lexical plugins (core functionality)
│   ├── toolbar/              - Top toolbar plugins
│   │   ├── toolbar-plugin.tsx              - Main toolbar container
│   │   ├── block-format-toolbar-plugin.tsx - Block type dropdown
│   │   ├── font-format-toolbar-plugin.tsx  - Bold, italic, etc.
│   │   ├── font-family-toolbar-plugin.tsx  - Font selection
│   │   ├── font-size-toolbar-plugin.tsx    - Font size picker
│   │   ├── font-color-toolbar-plugin.tsx   - Text color
│   │   ├── font-background-toolbar-plugin.tsx - Highlight color
│   │   ├── element-format-toolbar-plugin.tsx  - Alignment
│   │   ├── link-toolbar-plugin.tsx         - Link insertion
│   │   ├── image-toolbar-plugin.tsx        - Image insertion
│   │   ├── table-toolbar-plugin.tsx        - Table insertion
│   │   ├── horizontal-rule-toolbar-plugin.tsx - HR insertion
│   │   ├── code-language-toolbar-plugin.tsx   - Code block language
│   │   ├── block-insert-plugin.tsx         - Insert dropdown
│   │   └── history-toolbar-plugin.tsx      - Undo/redo
│   ├── picker/               - Slash command menu items

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htmujahid/undocx-archived](https://github.com/htmujahid/undocx-archived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
