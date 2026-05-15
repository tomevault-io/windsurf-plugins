---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📁 Documentation Structure

**IMPORTANT**: All project documentation is located in the `/docs` directory for better organization.

### Documentation Guidelines for AI Assistants

When creating or updating documentation:
1. **Always place `.md` files in `/docs` directory** (except `README.md` in root)
2. **Update links** when moving files to maintain consistency
3. **Use relative paths** from the root when linking: `docs/FILENAME.md`
4. **Keep README.md** in the root for GitHub visibility
5. **Organize by topic**:
   - Setup guides → `docs/SETUP.md`, `docs/TRANSCRIPTION_SETUP.md`
   - Architecture → `docs/ARCHITECTURE.md`, `docs/WEBRTC_FLOW.md`
   - Development → `docs/TESTING.md`, `docs/TODO.md`
   - Resources → `docs/RESOURCES.md`

## 📚 Available Documentation

### Quick Start
- **[SETUP.md](./SETUP.md)** - Initial project setup guide
- **[TRANSCRIPTION_SETUP.md](./TRANSCRIPTION_SETUP.md)** - Quick setup guide for transcription models ⭐ NEW

### Core Documentation
- **[TRANSCRIPTION.md](./TRANSCRIPTION.md)** - Detailed transcription models documentation ⭐ NEW
- **[WEBRTC_FLOW.md](./WEBRTC_FLOW.md)** - Detailed explanation of WebRTC recording and transcription flow
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Complete system architecture diagrams and data flows

### Development
- **[TODO.md](./TODO.md)** - Project roadmap and current status
- **[TESTING.md](./TESTING.md)** - Testing documentation and guidelines
- **[RUN_TESTS.md](./RUN_TESTS.md)** - How to run tests
- **[TEST_RESULTS.md](./TEST_RESULTS.md)** - Latest test results

### Resources
- **[RESOURCES.md](./RESOURCES.md)** - AI tools and speech analysis resources
- **[PROJECT_STATUS.md](./PROJECT_STATUS.md)** - Current project status
- **[REMAINING_TASKS.md](./REMAINING_TASKS.md)** - Outstanding tasks

### Summaries
- **[TRANSCRIPTION_SUMMARY.md](./TRANSCRIPTION_SUMMARY.md)** - Complete transcription integration overview

## Project Overview

**Orator AI** is a production-ready Next.js 14 application for AI-powered voice challenges with authentication, credit-based payments, and AI feedback. Users register, purchase credits, take voice challenges via WebRTC, and receive AI-generated performance analysis.

## Technology Stack

- **Framework**: Next.js 14 (App Router, Server Components, Server Actions)
- **Language**: TypeScript (strict mode)
- **Database**: SQLite via Drizzle ORM (better-sqlite3 driver)
- **Auth**: NextAuth v4.24 with JWT and Drizzle adapter
- **Payments**: Stripe and Paddle webhooks
- **Styling**: Tailwind CSS + shadcn/ui components
- **Audio**: WebRTC adapter
- **Transcription**: OpenAI Whisper + ElevenLabs Scribe v2 (dual model support)
- **Analysis**: GPT-4 for speech feedback
- **Testing**: Jest with ts-jest

## Common Commands

```bash
# Development
npm run dev          # Start dev server on localhost:3000
npm run build        # Build production bundle
npm start            # Start production server
npm run lint         # Run ESLint
npm test             # Run Jest tests

# Database (Drizzle)
npx drizzle-kit push              # Push schema changes to database
npx drizzle-kit studio            # Open Drizzle Studio (DB GUI)

# Docker
docker-compose up --build        # Build and run production container
```

## Architecture & Key Patterns

### Directory Structure

```
/app                 # Next.js App Router pages (client/server components)
/components          # Reusable UI components
/pages/api           # API routes (all backend logic)
/docs                # 📚 All project documentation (NEW)
/drizzle             # Database schema and connection
  ├── schema.ts      # Table definitions (users, sessions, challenges, payments, callHistory)
  └── db.ts          # SQLite connection instance
/lib                 # Shared utilities and configurations
  └── transcription  # Transcription service layer (Whisper + Scribe v2)
/storage             # Local file storage (SQLite DB + WAV recordings)
```

### Database Schema

Five main tables:
- **users**: Authentication (email/password) and credit balance
- **sessions**: Active/completed call sessions with audio file paths + transcription model
- **challenges**: Predefined voice challenge templates
- **payments**: Transaction history for credit purchases
- **callHistory**: AI feedback, performance metrics, and transcripts per call

All timestamps use `integer` with `mode: 'timestamp'`. Foreign keys link users to their sessions, payments, and call history.

### Call Flow Architecture

1. **Start Call**: `/api/call/start` deducts 1 credit, creates session with UUID + selected transcription model, returns sessionId
2. **Live Call**: `/call` page displays audio visualizer, records via WebRTC
3. **End Call**: `/api/call/end` saves WAV to `/storage/sessions/`, marks session complete, triggers evaluation
4. **Evaluation**: `/api/eval` transcribes audio (using selected model), analyzes with GPT-4, saves to callHistory
5. **Results**: `/result` page displays clarity score, filler words, tone, confidence, highlights, and transcript

### Transcription System (NEW)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jacke/lutherking](https://github.com/Jacke/lutherking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
