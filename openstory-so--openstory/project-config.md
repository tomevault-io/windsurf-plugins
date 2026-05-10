---
trigger: always_on
description: Core project context and architecture for OpenStory AI video platform
---


# OpenStory Project Context

You are working on **OpenStory**, an AI-powered cinematic content creation platform that transforms scripts into consistent, styled video productions.

## Project Overview

OpenStory democratizes cinematic content creation through:

- **Script-to-storyboard AI**: Automatic scene breakdown and frame generation
- **Style Stacks**: JSON-based presets that ensure consistent artistic vision across AI models
- **Multi-model support**: Integration with 19+ AI models (Fal.ai, Runway, Kling, etc.)
- **Character consistency**: LoRA model integration for consistent characters
- **Motion generation**: Image-to-video with VFX capabilities

## Project Architecture

Separated frontend/backend monorepo architecture:

```
openstory/
├── apps/
│   ├── backend/                # Elysia API server
│   │   ├── src/
│   │   │   ├── index.ts       # Elysia app entry point
│   │   │   ├── routes/        # Elysia route handlers
│   │   │   │   ├── sequences.ts
│   │   │   │   ├── frames.ts
│   │   │   │   ├── styles.ts
│   │   │   │   └── jobs.ts
│   │   │   ├── services/      # Business logic layer
│   │   │   │   ├── frame-generator.ts
│   │   │   │   ├── script-analyzer.ts
│   │   │   │   └── style-stack.ts
│   │   │   ├── db/            # Drizzle ORM setup & queries
│   │   │   │   ├── index.ts   # Database client
│   │   │   │   ├── schema/    # Drizzle schema definitions
│   │   │   │   └── queries/   # Reusable query functions
│   │   │   ├── workflows/     # Temporal workflows
│   │   │   │   ├── client.ts  # Temporal client
│   │   │   │   ├── frame-generation.ts
│   │   │   │   ├── motion-generation.ts
│   │   │   │   └── activities/
│   │   │   ├── plugins/       # Elysia plugins
│   │   │   │   ├── auth.ts    # BetterAuth integration
│   │   │   │   ├── cors.ts
│   │   │   │   └── logger.ts
│   │   │   ├── lib/           # Backend utilities
│   │   │   │   ├── ai/        # AI client wrappers
│   │   │   │   ├── storage/   # S3-compatible storage
│   │   │   │   └── errors.ts
│   │   │   └── migrations/    # Drizzle migrations
│   │   └── package.json
│   └── frontend/              # Next.js App Router (UI only)
│       ├── src/
│       │   ├── app/           # Next.js pages & layouts
│       │   │   ├── sequences/ # Sequence pages
│       │   │   ├── (auth)/    # Auth pages
│       │   │   └── layout.tsx
│       │   ├── components/
│       │   │   ├── sequence/  # Storyboard, frame editor
│       │   │   ├── ui/        # shadcn/ui components
│       │   │   └── layout/    # App layout
│       │   ├── hooks/         # TanStack Query hooks (API calls)
│       │   │   ├── use-frames.ts
│       │   │   ├── use-sequences.ts
│       │   │   └── use-user.ts
│       │   └── lib/           # Client-side utilities
│       │       ├── api-client.ts
│       │       └── auth-client.ts
│       └── package.json
├── packages/                  # Shared code
│   ├── database/              # Shared DB schema (future)
│   ├── types/                 # Shared TypeScript types
│   └── validation/            # Shared Zod schemas
└── docs/                      # Architecture & PRD docs
```

## Technology Stack

### Core Framework

- **Frontend**: Next.js 15.5 (App Router), React 19, TypeScript 5
- **Runtime**: Bun (package manager + test runner)
- **Build**: Turbopack (Next.js default)

### UI & Styling

- **UI**: shadcn/ui components, Radix UI primitives
- **Styling**: Tailwind CSS 4
- **Icons**: Lucide React
- **Drag & Drop**: dnd-kit

### Backend & Infrastructure

- **API Server**: Elysia (Bun-native web framework)
- **Database**: Supabase PostgreSQL via Drizzle ORM (type-safe queries, schema-first)
- **Validation**: ArkType (runtime type validation for Elysia routes)
- **Auth**: BetterAuth (anonymous + email/password + OAuth) - NOT Supabase Auth
- **Storage**: Supabase Storage (generated images/videos)
- **Real-time**: WebSockets via Elysia (generation status updates)
- **Workflow Engine**: Temporal (replaces QStash for async orchestration)
- **Caching**: Upstash Redis (session storage, rate limiting)

### State Management

- **Server State**: TanStack Query v5
- **Query Pattern**: Centralized query key factories
- **Caching**: Optimistic updates, automatic invalidation

### AI & Generation

- **Primary Provider**: Fal.ai SDK (@fal-ai/client)
- **Image Models**: flux-pro, imagen4, flux-krea-lora
- **Video Models**: veo3, kling-video, minimax-hailuo, wan-pro
- **Script Analysis**: OpenAI GPT-4 / Anthropic Claude
- **Validation**: ArkType schemas

### Development Tools

- **Testing**: Bun test (Jest-compatible)
- **Linting**: Oxclint (fast, replaces ESLint + Prettier)
- **Git Hooks**: Lefthook
- **Type Safety**: Drizzle Kit generates types from schema
- **API Documentation**: Elysia Swagger plugin (auto-generated)

## Core Features

### 1. Script-to-Storyboard Pipeline

1. User pastes script
2. AI analyzes and breaks into scenes/frames
3. Generates frame descriptions
4. Creates editable storyboard

### 2. Frame Generation

- Per-frame model selection
- Style Stack application for consistency
- Character LoRA injection
- Reference image support
- Parallel generation via Temporal workflows

### 3. Motion Generation

- Image-to-video conversion
- Multiple model options (veo3, kling, wan)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
