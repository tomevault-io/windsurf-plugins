---
trigger: always_on
description: This file defines how to build, test, and evolve Schedule Wizard.
---

# Schedule Wizard Agent Guide

This file defines how to build, test, and evolve Schedule Wizard.

## 1. Project Overview

- Product: Schedule Wizard
- Goal: a voice-first iOS app for schedule management, todos, and expense tracking.
- Core UX: user speaks, types, or uploads an image; AI interprets one or more actions; the app shows a confirmation sheet; database writes happen only after user confirmation.
- Architecture: thin client. The client handles UI, native capabilities, and streaming display. The server owns business logic, AI orchestration, validation, persistence, and side effects.

## 2. Source of Truth

- Product behavior: [`PRD.md`](/Users/pierrewang/Documents/repos/schedule-wizard/PRD.md)
- Technical architecture: [`TECH_DESIGN.md`](/Users/pierrewang/Documents/repos/schedule-wizard/TECH_DESIGN.md)
- If this file conflicts with those documents, follow the PRD for product behavior and the tech design for architecture.
- If implementation reveals a real conflict between the PRD and tech design, document it explicitly and resolve it before building further.

## 3. Current Repository State

- The repository is still close to the default Ionic starter app.
- Existing `Tab1/Tab2/Tab3`, starter tests, and placeholder UI are scaffolding and should be replaced.
- `server/`, `shared/`, database schema, AI orchestration, and production-grade feature modules still need to be created.
- There is an existing `package-lock.json`; from this point forward the project standard is `pnpm`, not `npm`.

## 4. Required Stack

Use the stack defined in `TECH_DESIGN.md`, with the latest stable compatible versions at implementation time.

### Client

- Ionic + React for the iOS-first app shell
- Capacitor for native integrations
- React 19
- React Router v5 for Ionic compatibility unless the app is intentionally migrated with a validated replacement plan
- TanStack Query for server state
- Zustand only for UI-local state
- TypeScript with strict mode
- Vite for frontend builds

### Server

- Node.js current active LTS
- Hono for the API layer
- Drizzle ORM with PostgreSQL on Neon
- Clerk for authentication
- Cloudflare R2 for uploaded files
- Vercel AI SDK for streaming, structured output, and tools

### Package Management

- Use `pnpm` only.
- Create and maintain `pnpm-lock.yaml`.
- Do not introduce or update `package-lock.json`.
- When migrating existing dependencies from the starter app, prefer the latest stable compatible versions rather than copying outdated examples.

## 5. Non-Negotiable Product Rules

- AI parsing must not write directly to the database.
- All create, update, and delete actions proposed by AI must first become `PendingTask` items.
- Users must be able to confirm or cancel the proposed actions before they take effect.
- A single input may produce multiple tasks across domains.
- The client is responsible for recording, image capture, haptics, rendering, and streaming UX.
- The server is responsible for intent parsing, task planning, validation, persistence, and authorization.
- All user data access must be scoped by authenticated `userId`.

## 6. Implementation Priorities

Build in this order unless a dependency forces a change.

1. Replace starter tabs with the real application shell and routing.
2. Establish shared types and the backend project structure.
3. Implement auth plumbing and API client conventions.
4. Implement database schema, migrations, and core CRUD services.
5. Implement AI chat SSE flow and pending task storage.
6. Implement confirmation flow and database commit path.
7. Implement Today view, timeline views, calendar views, and day detail flows.
8. Add upload, receipt/image handling, reminders, and reporting enhancements.

## 7. Development Specifications

### Project Structure

Target structure should follow `TECH_DESIGN.md`:

- `src/`: Ionic React client
- `server/`: Hono API server
- `shared/`: shared models, DTOs, and validation schemas

Keep boundaries strict:

- UI components do not contain business rules that belong on the server.
- API route handlers stay thin and delegate to services.
- Shared types must not import client-only or server-only modules.

### State Management

- TanStack Query is the default for any server-backed data.
- Zustand is only for ephemeral UI state such as recording state, current view mode, active sheet, and conversation UI state.
- Do not duplicate server state in Zustand.

### Validation and Types

- All API inputs and outputs should have runtime validation and static TypeScript types.
- Prefer a shared schema-first approach for request and response contracts.
- Never trust AI output, client payloads, or uploaded file metadata without validation.

### AI Flow

- Use structured outputs for orchestration and tool inputs.
- Domain agents should be constrained to explicit tools and validated payload shapes.
- Log enough metadata to debug failed task generation, but never log secrets or raw auth tokens.
- Handle ambiguity with clarification instead of guessing when the action could be wrong or destructive.

### Data and Time

- Store timestamps with timezone support.
- Be explicit about timezone handling across client, server, and AI prompts.
- Currency defaults to `CNY` unless the product requirement changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianghuawang/schedule-wizard](https://github.com/jianghuawang/schedule-wizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
