---
trigger: always_on
description: PipesHub is an AI-powered knowledge management dashboard on **Next.js** (App Router, client-rendered React).
---

# CLAUDE.md - PipesHub Dashboard UI

## Project Overview

PipesHub is an AI-powered knowledge management dashboard on **Next.js** (App Router, client-rendered React).

**Tech Stack:**
- Next.js 14+ (App Router, CSR-only with `'use client'`)
- TypeScript (strict mode)
- Zustand (state management)
- Radix UI Themes v3.2.1 (styling & components) - **No Tailwind CSS**
- Google Fonts (Manrope) + Google Material Icons
- React Hook Form + Zod (complex forms) / Native (simple forms)
- SSE (chat streaming) + WebSocket (notifications)
- i18n: German, English, Spanish, Hindi

## Naming: Collections vs Knowledge Base

**Important:** The feature formerly called "Knowledge Base" is now called **"Collections"** in the UI.

| Context | Name Used |
|---------|-----------|
| UI labels & text | "Collections" or "All Records" |
| Route path | `/knowledge-base` |
| API endpoints | `/api/v1/knowledgeBase` |
| Code (types, stores, variables) | `KnowledgeBase`, `kb`, `kbId` |
| Component files | `sidebar.tsx`, `header.tsx`, `filter-bar.tsx`, `kb-data-table.tsx` |

**Collections mode sidebar sections:**
- WORKSPACE (user's own collections)
- SHARED (collections shared by others)
- PRIVATE (private collections)

**All Records mode sidebar sections:**
- All Records (shows all records across sources)
- Collections (flat list of collections - clicking filters records)
- Connectors (Slack, Google Drive, Jira, etc.)

## Folder Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/             # Auth route group
│   │   ├── sign-in/        # Each page has: page.tsx, api.ts, store.ts, types.ts, components/
│   │   ├── sign-up/
│   │   └── reset-password/
│   ├── (dashboard)/        # Main app (uses query params, not dynamic routes)
│   │   ├── knowledge-base/ # Collections page - Uses ?kbId=xxx&folderId=xxx
│   │   │   ├── page.tsx
│   │   │   ├── api.ts      # Page-specific API calls
│   │   │   ├── store.ts    # Page-specific Zustand store
│   │   │   ├── types.ts    # Page-specific types
│   │   │   └── components/ # Page-specific components
│   │   ├── agents/         # Uses ?agentId=xxx
│   │   ├── chat/           # Uses ?conversationId=xxx
│   │   ├── connectors/
│   │   ├── account/
│   │   ├── users/
│   │   ├── groups/
│   │   └── notifications/
│   ├── layout.tsx          # Root layout (providers)
│   └── page.tsx            # Redirect to dashboard or auth
│
├── components/             # Shared UI components (stateless)
│   ├── ui/                 # Radix Themes components (MaterialIcon, Select, etc.)
│   ├── form/               # React Hook Form wrappers
│   ├── layout/             # Sidebar, header, breadcrumbs
│   ├── data-display/       # DataTable, empty/error/loading states
│   ├── feedback/           # Toast, confirmation dialogs
│   └── icons/              # Google Material Icons wrapper
│
├── lib/                    # Core utilities
│   ├── api/                # API layer
│   │   ├── axios-instance.ts  # Axios client with interceptors
│   │   ├── api-error.ts       # ErrorType & ProcessedError
│   │   ├── swr-fetcher.ts     # SWR fetcher
│   │   ├── streaming.ts       # SSE streaming (native fetch)
│   │   └── index.ts           # Barrel export
│   ├── store/              # Global Zustand stores
│   │   └── auth-store.ts      # Auth state (tokens, user)
│   ├── hooks/              # Global utility hooks
│   ├── utils/              # formatters, validators
│   ├── constants/          # Routes, storage keys, API endpoints
│   └── i18n/               # i18next config and locales
│
├── styles/                 # Global CSS and fonts
├── types/                  # Shared TypeScript types
├── config/                 # Site config and env variables
└── middleware.ts           # Auth redirect middleware
```

## Naming Conventions

### Files & Folders
| Type | Convention | Example |
|------|------------|---------|
| Folders | kebab-case | `knowledge-base/`, `data-display/` |
| Components | kebab-case.tsx | `kb-card.tsx`, `message-bubble.tsx` |
| Hooks | use-*.ts | `use-auth.ts`, `use-debounce.ts` |
| Stores | store.ts | Page-level store (`app/(dashboard)/chat/store.ts`) |
| APIs | api.ts | Page-level API (`app/(dashboard)/chat/api.ts`) |
| Types | types.ts | Page-specific types |
| Utils | kebab-case.ts | `format-date.ts`, `cn.ts` |

### Code Style
| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `KbCard`, `MessageBubble` |
| Hooks | camelCase with `use` | `useAuth`, `useDebounce` |
| Functions | camelCase | `formatDate`, `handleSubmit` |
| Constants | SCREAMING_SNAKE | `STORAGE_KEYS.JWT_TOKEN` |
| Types/Interfaces | PascalCase | `KnowledgeBase`, `User` |

## Key Architecture Patterns

### Component Types
- **Stateless** (`/components/ui/`): Pure presentational, no internal state, uses TypeScript union types for variants
- **Stateful** (`/app/(dashboard)/*/components/`): Page-specific with hooks, state, business logic

### Page-Level Co-location
Each page folder contains its own resources:
```
app/(dashboard)/[page]/
├── page.tsx        # Main page component (reads query params)
├── api.ts          # Page-specific API calls
├── store.ts        # Page-specific Zustand store

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pipeshub-ai/pipeshub-ai](https://github.com/pipeshub-ai/pipeshub-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
