---
trigger: always_on
description: **Sandpiper** is the National Tutoring Observatory's (NTO) React-based web application designed for analyzing one-on-one tutoring data. The app enables researchers to normalize, de-identify, and automatically annotate tutoring transcripts using large language models (LLMs). Users can create custom prompts, organize work into projects/runs/sessions, and export annotated data in CSV/JSON formats.
---

# Sandpiper - Copilot Instructions

## Repository Overview

**Sandpiper** is the National Tutoring Observatory's (NTO) React-based web application designed for analyzing one-on-one tutoring data. The app enables researchers to normalize, de-identify, and automatically annotate tutoring transcripts using large language models (LLMs). Users can create custom prompts, organize work into projects/runs/sessions, and export annotated data in CSV/JSON formats.

**Repository size**: ~446 source files (excluding node_modules), ~276MB total
**Primary language**: TypeScript (React 19)
**Framework**: React Router v7 with SSR
**Build tool**: Vite 6
**Package manager**: Yarn 1.22+ (preferred) or npm
**Node version**: 20.x (as per Dockerfile and CI)
**UI Framework**: Tailwind CSS 4 with shadcn/ui components (Radix UI)

## Problem-Solving Approach

**CRITICAL**: Always think before coding. Understand the problem fully before implementing solutions.

### Before Writing Code

1. **Analyze the request** - Break down what the user is actually asking for
2. **Ask clarifying questions** - If anything is unclear or ambiguous
3. **Understand the context** - Check existing code patterns and architecture
4. **Plan your approach** - Think through the solution before implementing
5. **Consider edge cases** - What could go wrong or be misunderstood?

### When to Ask Questions

- **Unclear requirements** - "Which specific component should handle this?"
- **Multiple approaches** - "Would you prefer to modify the existing component or create a new one?"
- **Scope ambiguity** - "Should this apply to all forms or just this specific one?"
- **Technical decisions** - "Should I use the existing API pattern or create a new endpoint?"
- **Missing context** - "What should happen when the user doesn't have permission?"

### Examples of Good Clarification

```
// ❌ Jumping straight to code
User: "Add a delete button"
Assistant: *immediately writes delete button code*

// ✅ Understanding first
User: "Add a delete button"
Assistant: "I'd like to clarify a few things:
- Which component/page should have the delete button?
- What should it delete (user, project, session)?
- Should there be a confirmation dialog?
- What permissions are needed?
Let me check the existing patterns..."
```

### When to proceed with coding

- Once you have a clear understanding of the requirements and context
- After confirming any ambiguities with the user
- Start development and apply the changes to the codebase

### Problem-Solving Workflow

1. **Read and understand** the full request
2. **Examine existing code** to understand patterns and find similar implementations
3. **Ask questions** if anything is unclear
4. **Look for reusable components** or established patterns before creating new ones
5. **Explain your approach** before implementing
6. **Write code** that follows project conventions
7. **Test and verify** the solution works

## Build & Validation Commands

### Prerequisites

1. **Node.js 20.x or higher** is required
2. **Yarn 1.22+** is the preferred package manager (though npm will work)
3. **Environment file**: Copy `.env.example` to `.env` before running the app

### Installation

Always run installation with frozen lockfile to match CI:

```bash
yarn install --frozen-lockfile
```

**Time**: ~30-40 seconds
**Note**: You may see a warning about `package-lock.json` - this is expected and safe to ignore

### TypeCheck

Run before committing to catch type errors (required by CI):

```bash
yarn typecheck
```

**Time**: ~7-8 seconds
**What it does**: Runs `react-router typegen && tsc` to generate route types and check TypeScript

### Build

Always run before testing production behavior:

```bash
yarn app:build
```

**Time**: ~8-10 seconds
**What it does**:

1. Runs `node ./app/adapters.js` to generate storage adapter imports
2. Runs `react-router build` to create production build in `./build/`
   **Expected warnings**: You'll see warnings about unused imports (useCallback, icons, etc.) - these are safe to ignore

### Development Server

```bash
yarn app:dev
```

**Port**: http://localhost:5173
**What it does**: Runs adapters.js first, then starts dev server with HMR
**Note**: For full functionality (workers, Socket.IO), also run `yarn redis` in a separate terminal

### Production Server

```bash
yarn start
```

**Port**: 5173 (configurable via PORT env var)
**Prerequisites**: Must run `yarn build` first

### Redis (For Background Jobs & Socket Communication)

```bash
yarn local:redis
```

**What it does**: Starts redis-memory-server for local development (if REDIS_URL not set)
**When needed**: Required for workers, Socket.IO, and BullMQ functionality
**Production**: Set REDIS_URL environment variable instead

### Workers (Background Jobs)

```bash
yarn workers:dev
```

**What it does**: Starts BullMQ workers for background task processing
**Prerequisites**: Redis must be running (use `yarn redis` or set REDIS_URL)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [National-Tutoring-Observatory/sandpiper](https://github.com/National-Tutoring-Observatory/sandpiper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
