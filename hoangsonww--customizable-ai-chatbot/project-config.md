---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Essential Commands

```bash
# Development
npm run dev          # Start Next.js dev server on localhost:3000
make dev            # Alternative using Makefile

# Building
npm run build       # Build production bundle
make build          # Alternative using Makefile

# Testing
npm test            # Run test suite
make test           # Alternative using Makefile

# Linting & Formatting
npm run lint        # Run ESLint
npm run format      # Format code with Prettier

# Setup
make setup          # Initial project setup (clone, install deps, scaffold .env)

# Deployment
make deploy         # Deploy to Vercel

# Pinecone/RAG
make upsert         # Upsert documents to Pinecone for RAG

# Customization
make customize      # Interactive config for UI/identity/prompts
```

### Package Manager

This project uses **pnpm** (version 9.12.0) as specified in package.json. If you need to install dependencies:

```bash
pnpm install
```

## Architecture Overview

This is a **Next.js 14 App Router** application implementing a multi-provider AI chatbot with RAG (Retrieval-Augmented Generation) capabilities.

### Core Architecture Pattern

The system follows a **three-tier request flow**:

```
User Input → Intention Detection → Response Generation → Streaming Output
```

### Key Architectural Components

#### 1. API Layer (`app/api/chat/route.ts`)

Single POST endpoint that orchestrates the entire chat flow:
- Receives chat object with message history
- Calls `IntentionModule.detectIntention()` to classify user intent
- Routes to appropriate response handler in `ResponseModule`
- Returns a ReadableStream with Server-Sent Events (SSE)

**Critical**: The API route initializes all three AI providers (OpenAI, Anthropic, Fireworks) and Pinecone client at module level, not per-request.

#### 2. Intention Detection (`modules/intention.ts`)

Uses OpenAI's structured output with Zod schema to classify messages into:
- `"question"` - Requires RAG pipeline and knowledge retrieval
- `"hostile_message"` - Inappropriate content, needs gentle deflection
- `"random"` - General conversation, no context needed

**Implementation**: Uses `openai.beta.chat.completions.parse()` with `zodResponseFormat()` for guaranteed structured JSON responses.

#### 3. Response Generation (`modules/response.ts`)

Three distinct response methods:

**A. `respondToQuestion()` - RAG Pipeline**
1. Generate hypothetical answer using HyDE technique (`utilities/chat.ts:generateHypotheticalData()`)
2. Embed hypothetical answer with `text-embedding-ada-002`
3. Query Pinecone for top-k similar chunks (default: 7)
4. Aggregate chunks by source and build context with citations
5. Stream response from configured AI provider with inline citation markers `[1]`, `[2]`, etc.

**B. `respondToRandomMessage()` - Direct Generation**
- Uses recent message history (configurable via `HISTORY_CONTEXT_LENGTH`)
- No RAG, just conversational AI with system prompt

**C. `respondToHostileMessage()` - Defensive Response**
- No message history (fresh context)
- Polite decline with identity protection (never reveals technical details or "OpenAI")

#### 4. Streaming System (`actions/streaming.ts`)

**Provider-agnostic streaming** with two implementations:
- `handleOpenAIStream()` - For OpenAI and Fireworks (OpenAI SDK compatible)
- `handleAnthropicStream()` - For Anthropic Claude models

**Stream Message Types**:
```typescript
{ type: "loading", indicator: { status: string, icon: IconType } }
{ type: "message", message: { role, content, citations } }
{ type: "error", indicator: { status: string, icon: "error" } }
{ type: "done", final_message: string }
```

Frontend must parse newline-delimited JSON stream.

### RAG Pipeline Deep Dive

**HyDE (Hypothetical Document Embeddings) Technique**:

Instead of embedding the user's question directly, the system:
1. Generates what an ideal answer *would* look like (`HYDE_MODEL`: gpt-4o-mini)
2. Embeds the hypothetical answer (more semantically rich)
3. Uses that embedding to find similar real documents in Pinecone

**Why HyDE?** Questions and answers have different semantic structures. Embedding a hypothetical answer finds documents that *answer* the question, not just documents that mention similar keywords.

**Pinecone Metadata Structure**:
```typescript
{
  text: string              // Main chunk content
  pre_context: string       // Text before chunk for continuity
  post_context: string      // Text after chunk for continuity
  source_url: string        // Unique source identifier
  source_description: string // Human-readable source name
  order: number             // Position in original document
}
```

**Citation System**: Chunks from the same `source_url` are grouped and numbered sequentially. The system injects `[1]`, `[2]` markers into the context sent to the AI, which the AI naturally includes in responses.

## Configuration System

The entire chatbot behavior is driven by files in `/configuration/`:

### Critical Configuration Files

#### `configuration/identity.ts`
Defines the AI's persona and owner information. Used in **all system prompts**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsonww/Customizable-AI-Chatbot](https://github.com/hoangsonww/Customizable-AI-Chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
