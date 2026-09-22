---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Accounting AI Agent - full-stack monorepo for AI-powered accounting automation with Polish wFirma system integration. Features AI chat (Claude/GPT-4 via LangGraph), wFirma API integration with caching, OAuth authentication.

## Commands

```bash
# Development (starts both API and Web)
npm run dev

# Build all packages
npm run build

# Test all packages
npm run test

# Lint all packages
npm run lint

# Database
npm run prisma:generate    # Generate Prisma client
npm run prisma:migrate     # Run migrations
npm run prisma:studio      # Open Prisma GUI

# Docker
npm run docker:up          # Start PostgreSQL + Redis
npm run docker:down        # Stop services

# Single package commands
npm run dev --filter=@accounting-ai-agent/api
npm run test --filter=@accounting-ai-agent/web

# E2E tests (from packages/web)
npm run test:e2e
```

## Architecture

### Monorepo Structure (Turbo)
- `packages/api` - Express.js backend (TypeScript, Prisma, Redis)
- `packages/web` - Next.js 15 frontend (React 19, Tailwind, Zustand)

### Backend Layers (packages/api/src/)
```
routes/      → controllers/      → services/      → Prisma/Redis
             HTTP handlers        Business logic   Data access
```

### Key Services
- **WFirmaIntegrationService** - wFirma API calls with retry logic
- **WFirmaCacheService** - PostgreSQL caching layer (TTL-based)
- **AIChatService** - Thin orchestrator for AI chat; delegates to `LangGraphAgentRunner` (LangGraph single agent, 58 domain tools — up to 82 with HR + KSeF), `ConversationRepository` (Prisma CRUD + access control), and `TTSIntegration` (audio + cost tracking)
- **LangGraphAgentRunner** - Builds and runs the LangGraph `StateGraph` (agent → tools loop), model selection, tool binding
- **AIMemoryService** - Persistent cross-session AI context memory (CRUD + prompt injection)
- **AIMemoryExtractionService** - Fire-and-forget memory extraction from conversations (pattern-based, zero LLM cost)
- **AuthService** - JWT + OAuth (Google); HTTP layer split into AuthCore/OAuth/Token/Profile controllers
- **TTSService** - Text-to-speech via OpenAI TTS API
- **OrganizationService** - Company grouping with admin/member roles and membership approval
- **TelegramBotService** - Telegraf chatbot orchestrator; account linking via 6-digit codes. Delegates to sub-modules: `AIChatRouter` (text → AIChatService), `OcrFlowHandler` (photo → receipt OCR → expense), `TelegramRateLimiter` (Redis-backed per-user limits)
- **TaxDeadlineReminderService** - Hourly scheduler (started in `index.ts`) that proactively DMs linked Telegram users about upcoming Polish tax deadlines; deduped per day via Redis
- **ReferralService** - Referral program with Stripe credit rewards and coupon discounts
- **TaxCalendarService** - Polish statutory tax deadlines (VAT, CIT, PIT, ZUS, PCC, dividends) with weekend/holiday shifting

### Frontend Structure (packages/web/src/)
- `app/` - Next.js App Router pages
- `components/` - React components (chat/, auth/, ui/)
- `contexts/` - React contexts (Auth, Locale, TTS)
- `hooks/` - Custom hooks (useChat, useAuth, useTextToSpeech, useVoiceDictation)
- `i18n/` - Translations (en, pl, ru)
- `lib/api/` - Axios API client

### Database
PostgreSQL with Prisma ORM. Key models: User, AIConversation, Organization, TelegramLink, WFirmaCache, WFirmaInvoice, WFirmaCustomer, AIMemory, AIToolUsage, Referral.

## Tech Stack

**Backend:** Node.js 18+, Express, TypeScript, Prisma, Redis, LangChain/LangGraph, Telegraf, Zod, Winston
**Frontend:** Next.js 15, React 19, Tailwind CSS, React Query, Zustand, next-intl
**Infrastructure:** Docker Compose, Turbo, GitHub Actions

## Environment

API requires `.env` with: DATABASE_URL, REDIS_URL, JWT_SECRET, WFIRMA_* credentials, TELEGRAM_CHATBOT_TOKEN (optional)
Web requires `.env.local` with: NEXT_PUBLIC_API_URL

## Key Patterns

- Singleton services via `*.instance.ts` files
- Zod validation on both frontend and backend
- wFirma responses cached in PostgreSQL with configurable TTL
- AI tools return localized markdown (auto-detect user language)
- Protected routes via AuthContext + JWT middleware
- Text-to-speech via Web Speech API or OpenAI TTS (TTSContext, useTextToSpeech hook)
- Voice input via Web Speech Recognition API (useVoiceDictation hook)
- OpenAI TTS: higher quality voices (Nova, Alloy, Echo, etc.) when user has OpenAI key
- AI Context Memory: persistent cross-session memory injected into system prompt (categories: business_fact, frequent_entity, user_preference, workflow_pattern)
- Memory extraction is fire-and-forget after each AI response (zero LLM cost — pattern-based only)
- Organizations: users grouped by company name, org-level roles (admin/member), membership approval flow
- Shared conversations: org members share AI chats, real-time polling (5s messages, 10s list)
- Telegram bot: AI chat via Telegraf, account linking via 6-digit Redis codes
- Referral program: unique 8-char codes, Stripe credit for referrer (max 10/year), 20% coupon for referred, 7-day revocation window

## Documentation

Detailed docs in `packages/api/docs/` (API.md, ARCHITECTURE.md, AUTH_API.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [micode-ai/accounting-ai-agent](https://github.com/micode-ai/accounting-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
