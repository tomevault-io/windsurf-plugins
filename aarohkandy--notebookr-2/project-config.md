---
trigger: always_on
description: Notebookr is an **agentic writer** application focused on generating **longer text** through AI-powered assistance. The application enables users to create comprehensive documents through a conversational AI interface. It features standalone authentication, AI-powered content generation using OpenRouter (with OpenAI fallback), and a credit-based monetization system. The goal is to streamline long-form writing and make it efficient and accessible.
---

# Notebookr - AI Context for Development

## Project Overview

Notebookr is an **agentic writer** application focused on generating **longer text** through AI-powered assistance. The application enables users to create comprehensive documents through a conversational AI interface. It features standalone authentication, AI-powered content generation using OpenRouter (with OpenAI fallback), and a credit-based monetization system. The goal is to streamline long-form writing and make it efficient and accessible.

**Communication Style:** Use simple, everyday language when communicating with users or in documentation.

**Application Focus:** This is an agentic writer optimized for longer text generation, not short snippets or quick responses. The AI workflow and UI are designed to support extended writing sessions and comprehensive document creation.

## Tech Stack

### Frontend
- **React 18** with **TypeScript**
- **Vite** for build tooling and dev server
- **Wouter** for client-side routing
- **TanStack Query** for data fetching and state management
- **Radix UI** primitives with **shadcn/ui** components
- **Tailwind CSS** for styling
- **Lucide React** for icons

### Backend
- **Node.js** with **Express.js** (TypeScript)
- **Drizzle ORM** with **PostgreSQL** (Neon serverless)
- **connect-pg-simple** for PostgreSQL session storage
- **Zod** for schema validation
- **Server-Sent Events (SSE)** for streaming AI responses

### AI Services
- **OpenRouter** (primary) - Multi-model fallback system with API key rotation
  - Models: Llama-3.3-70B, Gemini-2.0-Flash, Claude-3.5-Sonnet
- **OpenAI API** (fallback provider)
- Credit-based model selection (free users use OpenRouter, paid users can access faster OpenAI models)

### External Services
- **Stripe** for payment processing and credit purchases
- **Google Fonts**: Inter (UI/content) and JetBrains Mono (code blocks)

## Project Structure

```
/
├── client/          # Frontend React application
│   ├── src/
│   │   ├── components/     # React components
│   │   │   ├── ui/         # shadcn/ui components
│   │   │   └── ...
│   │   ├── pages/          # Page components
│   │   ├── hooks/          # React hooks
│   │   └── lib/            # Utility functions
│   └── index.html
├── server/          # Express backend
│   ├── index.ts     # Server entry point
│   ├── routes.ts    # API route registration
│   ├── auth.ts      # Authentication middleware
│   ├── ai-service.ts # AI generation logic
│   ├── db.ts        # Database connection
│   └── storage.ts   # Storage interface
├── api/             # Vercel serverless API routes
│   ├── _shared/     # Shared utilities (auth, cors, db, storage)
│   ├── auth/        # Authentication endpoints
│   ├── ai/          # AI generation endpoints
│   ├── notebooks/   # Notebook CRUD endpoints
│   ├── sections/    # Section endpoints
│   └── ...
├── shared/          # Shared code between client and server
│   └── schema.ts    # Drizzle schema and Zod validation
└── ...
```

## Architecture Patterns

### API Design
- **RESTful API** with Express routes
- All protected routes require authentication via `isAuthenticated` middleware
- Request validation using Zod schemas from `shared/schema.ts`
- User ownership validation for notebooks and sections
- API routes are in `/api` directory structure (Vercel serverless functions)

### Data Models
- **Users**: Authentication, credits, selected AI model
- **Notebooks**: User-scoped, contain `aiMemory` JSONB field for AI context
- **Sections**: Belong to notebooks, have order index
- **Messages**: Chat messages with role (user/assistant/system), messageType (status/completion)
- **SectionVersions**: Automatic snapshots before updates (for restore functionality)
- **Transactions**: Credit purchase and deduction history

### Authentication
- Standalone username/password authentication
- Password hashing using scrypt
- PostgreSQL session storage
- User sessions stored in `sessions` table

### AI System Architecture
- **Four-Phase Workflow**: Plan → Execute → Review → Post-Process
  - **Planning**: Conversational Q&A to understand user intent, extracts variables, creates document plan
  - **Execution**: Generates sections one at a time (2-5 paragraphs minimum, optimized for longer text)
  - **Review**: Evaluates quality and completeness
  - **Post-Process**: Fixes AI detection patterns
- **Agentic Writer Focus**: Optimized for longer text generation - sections should be substantial (multi-paragraph), with flowing transitions and comprehensive content
- **AI Memory**: Stored in `notebook.aiMemory` JSONB field (TODO lists, document plans, original instructions)
- **Streaming**: SSE with `threePhaseGenerationStream` generator, heartbeat events to prevent timeouts
- **Version History**: Automatic section snapshots before updates, restore functionality available

## Design System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aarohkandy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
