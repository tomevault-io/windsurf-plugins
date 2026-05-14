---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation

- **README.md** (Portuguese): Main user-facing documentation
- **README_EN.md** (English): English version of the README
- **CLAUDE.md** (This file): Developer documentation for Claude Code
- **ARCHITECTURE.md**: System architecture diagrams and component overview
- **LIMITS.md**: Rate limiting, connection limits, and performance bottlenecks

## Project Overview

**Gambiarra LLM Club Arena Local** - A LAN-based arena for creative competitions using locally-run LLMs. Inspired by the Homebrew Computer Club, this platform celebrates creative solutions ("gambiarras") and community over pure performance benchmarks.

**Core Features:**
- Live challenges broadcast to participants
- Real-time token streaming from multiple LLM instances
- Public display (telão) showing generation progress
- Voting system for audience participation
- CSV export of results and metrics

## Technology Stack

- **Backend:** Node.js 20+ with TypeScript, Fastify, Prisma ORM, SQLite
- **Frontend:** React 18+ with Vite, Tailwind CSS
- **WebSocket:** @fastify/websocket for real-time streaming
- **Validation:** Zod schemas for type-safe messaging
- **Package Manager:** pnpm (monorepo with workspaces)

**Why this stack?** Fastify provides excellent WebSocket performance for low-latency streaming, Prisma ensures type-safety across the database layer, and pnpm enables fast installation crucial for participant onboarding.

## Architecture

### Monorepo Structure

```
├── server/              # Fastify backend with WebSocket hub
├── client-typescript/   # TypeScript CLI for participants to connect their LLMs
├── telao/               # React frontend for public display
```

> **Note:** The Python client has been moved to a separate repository.

### Key Components

**Server (`server/`):**
- `src/ws/hub.ts`: WebSocket connection manager, handles token streaming and broadcasts
- `src/core/rounds.ts`: Round lifecycle management (create, start, stop)
- `src/core/votes.ts`: Voting and scoreboard aggregation
- `src/http/routes.ts`: REST API for session/round control
- `prisma/schema.prisma`: Database schema (Session, Participant, Round, Metrics, Vote)

**Client TypeScript (`client-typescript/`):**
- `src/runners/ollama.ts`: Ollama API integration
- `src/runners/lmstudio.ts`: LM Studio API integration
- `src/runners/mock.ts`: Simulated token generation for testing
- `src/net/ws.ts`: WebSocket client with reconnection logic

**Telão (`telao/`):**
- `src/components/Arena.tsx`: Main display with participant grid (supports SVG rendering mode)
- `src/components/Voting.tsx`: Voting interface (accessible via QR code)
- URL routes: `/voting`, `/scoreboard`, `/admin` (path-based routing)

## Common Development Commands

```bash
# Root
pnpm install          # Install all workspace dependencies
pnpm dev              # Start server + telao in dev mode
pnpm build            # Build all workspaces
pnpm simulate         # Run 5 simulated clients
pnpm test             # Run all tests

# Server
cd server
pnpm db:migrate       # Run database migrations
pnpm db:generate      # Generate Prisma Client
pnpm db:studio        # Open Prisma Studio GUI
pnpm seed             # Seed with test data (PIN: 123456)
pnpm dev              # Start server with hot reload
pnpm test:coverage    # Run tests with coverage

# Client TypeScript
cd client-typescript
pnpm dev -- --url ws://localhost:3000/ws --pin 123456 \
  --participant-id test-1 --nickname "Test" --runner mock

# Telão
cd telao
pnpm dev              # Start Vite dev server on port 5173
pnpm lint             # Run ESLint
```

## Database Schema

- **Session**: Contains `pinHash` (bcrypt), `status` (active/ended)
- **Participant**: Links to Session, stores `nickname`, `runner`, `model`
- **Round**: Contains `prompt`, `maxTokens`, `temperature`, `deadlineMs`, `seed`, `svgMode`
- **Metrics**: Stores `tokens`, `latencyFirstTokenMs`, `durationMs`, `tpsAvg`, `generatedContent` per participant/round
- **Vote**: Links voter (hashed) to participant with `score` (1-5)

## Message Protocols

All messages validated with Zod schemas in `server/src/ws/schemas.ts`.

**Server → Client:**
- `challenge`: Broadcast when round starts
- `heartbeat`: Periodic keepalive (30s interval)

**Client → Server:**
- `register`: Initial authentication with PIN
- `token`: Streaming tokens with sequential `seq` number
- `complete`: Final metrics after generation completes
- `error`: Client-side error reporting

**Telão → Server:**
- `telao_register`: Telão client registration (optional `view` field)
- `vote`: Submit vote with `voter_id`, `participant_id`, and `score` (1-5)

## Testing Strategy

- **Unit tests**: Schema validation (`server/src/ws/schemas.test.ts`), runner logic (`client-typescript/src/runners/mock.test.ts`)
- **Integration**: Simulation script connects 5 clients and validates token sequencing
- **Manual**: Use `pnpm seed` + `pnpm simulate` for end-to-end validation

**Running single tests:**
```bash
# Run specific test file
pnpm --filter @gambiarra/server test src/ws/schemas.test.ts

# Run tests matching pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gambiarraclub/gambiarra-arena](https://github.com/gambiarraclub/gambiarra-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
