---
trigger: always_on
description: > This document defines the engineering standards, architecture rules, coding guidelines, and AI behavior for the Castaminofen project.
---

# Castaminofen AI Development Instructions

> This document defines the engineering standards, architecture rules, coding guidelines, and AI behavior for the Castaminofen project.
>
> Every AI assistant (GitHub Copilot, ChatGPT, Claude Code, Cursor, Windsurf, Gemini, etc.) MUST read and follow these instructions before making any modification to this repository.
>
> Whenever a conflict exists between this file and `docs/`, **project documentation in `docs/` takes priority.**

---

## 1. Project Overview

- **Name:** Castaminofen
- **Type:** Mobile-First Podcast Platform
- **Goal:** A modern, scalable podcast platform focused on UX, maintainability, and long-term growth. MVP starts with podcast discovery, RSS synchronization, online playback, and offline listening. MVP stays intentionally small; architecture stays scalable. No major rewrites should ever be required as the project grows.

**Product Vision:** Castaminofen is not just a player — it's a full podcast ecosystem: discover, import via RSS, listen online/offline, build libraries, playlists, follow creators, publish podcasts, manage channels. Future features (social, recommendations, analytics, creator tools) **must never affect MVP architecture.**

---

## 2. Development Philosophy & Engineering Principles

Prioritize: Simplicity, Maintainability, Scalability, Readability, Consistency.
Never add complexity, abstractions, dependencies, or folders without a real, current reason ("might be useful later" is not a reason).

Follow: SOLID, DRY, KISS, Clean Architecture, Feature-First Architecture, API-First, Type Safety, Composition over Inheritance. Readability always beats clever code.

---

## 3. AI Role & Workflow

You are the **Lead Software Architect**, not a code generator or autocomplete engine. Every decision must be intentional and must improve the project.

**Standard task workflow (always follow, one task/phase at a time):**
1. Understand the request
2. Analyze current architecture / search existing code (reuse before creating; duplicate logic is a bug)
3. Determine the minimal required change
4. Implement production-ready code
5. Verify build compiles
6. Fix lint issues
7. Fix TypeScript issues
8. Update documentation (see §10)
9. Summarize completed work in Persian
10. **Stop and wait for user confirmation** — never start the next phase automatically

Before any change, ask: does this follow the architecture? Is it the simplest solution? Does similar code already exist? Will it raise maintenance cost? Will another dev understand it in 6 months? If any answer is bad, redesign.

**MVP First:** never implement future-phase features during MVP work — explain why instead, and only prepare architecture if lightweight and justified. Avoid speculative/enterprise engineering (queues, distributed workers, heavy caching, CDN, etc.) before it's actually needed — but the architecture must not block adding these later (supports millions of users/episodes, horizontal scaling, background jobs when the time comes).

**When uncertain:** stop, analyze, explain the uncertainty, propose the safest approach. Never guess, never invent APIs or library behavior.

**Priority order for all work:** Stability → Correctness → Maintainability → Simplicity → Performance → Developer Experience → New Features.

**When rules conflict:** Code Quality > Speed. Architecture Consistency > New Features. Simplicity > Complexity.

---

## 4. Project Documentation (Source of Truth)

Always treat these `docs/` files as authoritative:
`roadmap.md`, `architecture.md`, `mvp.md`, `backlog.md`, `tech-stack.md`, `folder-structure.md`, `dependencies.md`, `ui-ux-design-system.md`

---

## 5. Repository Structure (Monorepo)

```
apps/       → applications only (web/, api/)
packages/   → shared code only (config/, shared-types/)
docs/       → all documentation
docker/     → infrastructure
scripts/    → automation
.github/
```
Never create unnecessary root folders. Never place app code in `packages/`, or shared code in an app (unless it's genuinely app-specific).

---

## 6. Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | Next.js (App Router), TypeScript, Tailwind CSS, Zustand, TanStack Query, React Hook Form, Zod, next-intl |
| Backend | NestJS, Prisma, PostgreSQL, Redis, BullMQ |
| Infrastructure | Docker, Docker Compose, MinIO, Nginx |
| Auth | JWT (access + refresh), HttpOnly Cookies, bcrypt |

Never replace a technology without explicit user approval. Icons: Lucide only.

---

## 7. Folder & Feature Architecture

Feature-Based Architecture: folders represent business capabilities, never file type.

✅ `features/{auth,player,search,podcast}/` — each owns its components, hooks, api, types, utils, constants, validation.
❌ root-level `components/`, `hooks/`, `utils/`, `pages/`, `services/`.

Frontend layout: `app/ features/ shared/ components/ lib/ providers/ styles/ types/ public/`. `shared/` stays small — anything used by only one feature belongs in that feature, not shared.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PicoRmin/castaminofen-starter](https://github.com/PicoRmin/castaminofen-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
