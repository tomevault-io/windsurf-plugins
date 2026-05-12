---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Document Chat System is a production-ready Next.js 15 application that combines intelligent document management with AI-powered conversations. It's a multi-tenant SaaS platform supporting multiple AI providers (OpenRouter, OpenAI, Anthropic), background job processing with Inngest, and vector search via Pinecone/pgvector.

## Development Commands

### Essential Commands
```bash
# Development
npm run dev                 # Start dev server (localhost:3000)
npm run build              # Production build (runs prisma generate first)
npm run start              # Start production server
npm run lint               # Run ESLint
npm run type-check         # TypeScript type checking

# Database (Prisma)
npm run db:generate        # Generate Prisma client
npm run db:push            # Push schema changes to database
npm run db:migrate         # Create and run migration
npm run db:studio          # Open Prisma Studio GUI
npm run db:seed            # Seed database with initial data
npm run db:reset           # ⚠️ Reset database (deletes all data)

# Combined Development
npm run dev:setup          # npm install + prisma generate + db push
npm run dev:reset          # db reset + db seed + dev server
```

### Running Tests
```bash
npm test                   # Run Jest tests
npm run test:watch         # Watch mode
npm run test:coverage      # Coverage report
```

### Background Jobs (Inngest)
```bash
npx inngest-cli dev        # Start Inngest dev server at localhost:8288
```

## Architecture Overview

### Technology Stack
- **Frontend**: Next.js 15 (App Router), React 19, TypeScript 5, Tailwind CSS, shadcn/ui, Zustand
- **Backend**: Next.js API Routes, Prisma ORM, PostgreSQL 14+
- **AI**: OpenRouter (100+ models), OpenAI, Anthropic via Vercel AI SDK
- **Auth**: Clerk (multi-tenant with organizations)
- **Storage**: Supabase (files) + PostgreSQL (metadata)
- **Background Jobs**: Inngest (document processing pipeline)
- **Vector Search**: Pinecone or pgvector
- **Cache**: Upstash Redis (optional)
- **Billing**: Stripe (optional)

### Directory Structure Deep Dive

```
src/
├── app/                          # Next.js 15 App Router
│   ├── api/                      # API routes
│   │   ├── inngest/             # Inngest webhook endpoint
│   │   └── v1/                  # Versioned REST APIs
│   ├── [page-routes]/           # Page routes (dashboard, chat, documents, etc.)
│   ├── layout.tsx               # Root layout with providers
│   └── middleware.ts            # Auth, CORS, rate limiting
│
├── lib/                         # Core business logic
│   ├── ai/                      # AI service layer (CRITICAL)
│   │   ├── providers/          # AI provider adapters
│   │   │   ├── openai-adapter.ts
│   │   │   ├── anthropic-adapter.ts
│   │   │   ├── smart-openrouter-adapter.ts  # Main OpenRouter implementation
│   │   │   └── factory.ts      # Provider factory pattern
│   │   ├── services/           # AI processing services
│   │   │   ├── embedding-service.ts         # Generate embeddings
│   │   │   ├── vector-search.ts            # Semantic search
│   │   │   ├── document-chunker.ts         # Split documents into chunks
│   │   │   ├── hybrid-search.ts            # Combine vector + keyword search
│   │   │   └── document-*-analyzer.ts      # Document analysis services
│   │   ├── routing/            # Intelligent request routing
│   │   ├── monitoring/         # Metrics, cost tracking, alerts
│   │   ├── config/             # AI configuration, feature flags, model registry
│   │   ├── prompts/            # Prompt templates and library
│   │   └── ai-service-manager.ts           # Main AI orchestration layer
│   │
│   ├── inngest/                # Background job processing
│   │   ├── client.ts           # Inngest client configuration
│   │   └── functions/          # Job definitions
│   │       ├── process-document-basic.ts   # Text extraction + sections
│   │       ├── process-document-full.ts    # Complete analysis
│   │       ├── vectorize-document.ts       # Generate embeddings
│   │       ├── analyze-document.ts         # AI-powered analysis
│   │       └── batch-process-documents.ts  # Batch processing
│   │
│   ├── documents/              # Document management logic
│   ├── file-processing/        # File type handlers (PDF, DOCX, images, etc.)
│   ├── auth/                   # Clerk authentication utilities
│   ├── storage/                # Supabase file storage
│   ├── db/                     # Prisma utilities and helpers
│   ├── cache/                  # Redis caching layer
│   ├── errors/                 # Custom error classes and handling
│   ├── audit/                  # Audit logging system
│   └── validation/             # Zod schemas for input validation
│
├── components/                  # React components
│   ├── ui/                     # shadcn/ui primitives
│   ├── documents/              # Document management UI
│   ├── chat/                   # AI chat interface
│   ├── billing/                # Stripe billing UI
│   └── layout/                 # App layout components
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [watat83/document-chat-system](https://github.com/watat83/document-chat-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
