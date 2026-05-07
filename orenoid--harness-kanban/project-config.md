---
trigger: always_on
description: Harness Kanban is an AI-powered issue tracking system - a modern, AI-native alternative to traditional issue trackers. It combines traditional issue management with an AI agent system that understands natural language commands to manage issues, add comments, track todos, and interact with users through an intelligent chat interface.
---

# Repository Guidelines

## Project Introduction

Harness Kanban is an AI-powered issue tracking system - a modern, AI-native alternative to traditional issue trackers. It combines traditional issue management with an AI agent system that understands natural language commands to manage issues, add comments, track todos, and interact with users through an intelligent chat interface.

### Key Features

- **AI Agent System**: Natural language chat interface with 20+ tools for issue management, comments, subscriptions, and personal todos
- **Harness Kanban Orchestration**: An in-progress `harness-kanban` backend for queueing and routing work to long-running coding workers
- **Flexible Issue Management**: Customizable property system allowing custom fields on issues with various types and validation
- **Multi-Provider AI Support**: Works with OpenAI, Anthropic, Google, Groq, and 20+ other LLM providers via Vercel AI SDK
- **Real-time Streaming**: Real-time AI responses with tool execution and UI rendering
- **Notification System**: Internal inbox notifications with an extensible channel abstraction for future delivery channels

### Technology Stack

**Backend**: NestJS 11, PostgreSQL with Prisma ORM, Better Auth, Vercel AI SDK, LangChain/LangGraph, ChromaDB
**Frontend**: Next.js 15 (App Router), React 18, Tailwind CSS 4, shadcn, Radix UI, TanStack Query
**Monorepo**: pnpm workspaces, Turborepo, TypeScript throughout

## Project Structure

> **Note**: To avoid outdated or incorrect project structure information, AI should always explore the current codebase to understand the latest structure. Recommended approach:
>
> 1. First browse the overall structure and key files to get a general understanding
> 2. Then dive deeper into specific modules related to the user's request
>
> This ensures working with the most up-to-date codebase state rather than relying on potentially stale documentation.

## Review Rules

### Commands that must run and pass

<!-- Each command should only be run after the previous one has completed without errors. -->

- `pnpm type-check` must pass
- `pnpm build` must pass in root project
- `pnpm test` must pass, unless specified by users that some tests failure can be ignored temporarily.
- For frontend changes, run `pnpm --filter @harness-kanban/web test:storybook`. Before running, use `pnpm --filter @harness-kanban/web storybook-alive` to check if Storybook is already running. If not, start it first with `pnpm --filter @harness-kanban/web storybook`.

### Code review

- **[IMPORTANT]** ALWAYS use accessible tools (e.g., chrome-devtools MCP, bash, etc.) to verify the new function actually works as expected !!! Passing unit tests doesn't guarantee the system works correctly !!! You MUST personally verify in the product that the modified features work correctly !!!
- **[IMPORTANT]** ALWAYS use accessible tools (e.g., chrome-devtools MCP, bash, etc.) to verify the new function actually works as expected !!! Passing unit tests doesn't guarantee the system works correctly !!! You MUST personally verify in the product that the modified features work correctly !!!
- **[IMPORTANT]** ALWAYS use accessible tools (e.g., chrome-devtools MCP, bash, etc.) to verify the new function actually works as expected !!! Passing unit tests doesn't guarantee the system works correctly !!! You MUST personally verify in the product that the modified features work correctly !!!
- for frontend changes, verify that hooks and Storybook mock structures align with backend API interfaces, including field names and value validity
- Backend main workflows should have E2E tests, branch logic should have unit test coverage
- New frontend stories should have proper component structure tests
- ensure there are no Chinese text in code or comments

## Development Convention and Guidance

### Available Scripts

#### Root Monorepo (`package.json`)

| Command            | Description                                        |
| ------------------ | -------------------------------------------------- |
| `pnpm build`       | Build all apps and packages using Turbo            |
| `pnpm dev`         | Start development mode for all apps using Turbo    |
| `pnpm start`       | Start web and API production servers using Turbo   |
| `pnpm lint`        | Run linting across all packages using Turbo        |
| `pnpm format`      | Format code across all packages using Turbo        |
| `pnpm test`        | Run unit tests across all packages using Turbo     |
| `pnpm test:e2e`    | Run E2E tests (migrates DB first using `.env.e2e`) |
| `pnpm db:generate` | Generate Prisma client                             |
| `pnpm db:migrate`  | Run database migrations                            |
| `pnpm db:studio`   | Open Prisma Studio                                 |
| `pnpm db:push`     | Push schema changes to database                    |
| `pnpm prepare`     | Run Husky git hooks setup                          |

#### API Server (`apps/api-server`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orenoid/harness-kanban](https://github.com/Orenoid/harness-kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
