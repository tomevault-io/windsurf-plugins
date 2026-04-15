---
trigger: always_on
description: AFC Stack is a production-ready full-stack monorepo template with an interactive CLI for customizable project generation. It combines Next.js 15, TypeScript, Drizzle ORM, WebSockets, and modern tooling.
---

# GitHub Copilot Instructions for AFC Stack

## Project Overview

AFC Stack is a production-ready full-stack monorepo template with an interactive CLI for customizable project generation. It combines Next.js 15, TypeScript, Drizzle ORM, WebSockets, and modern tooling.

## Core Architecture

### Monorepo Structure

- **Root**: Turborepo workspace with shared configs
- **apps/web**: Next.js 15 App Router frontend (TypeScript)
- **apps/ws**: Fastify WebSocket service (Bun runtime)
- **packages/db**: Shared Drizzle ORM schema and client
- **cli-templates/**: Template files for project generation

### Technology Stack

- **Frontend**: Next.js 15, React 18, Tailwind CSS
- **Backend**: Next.js API Routes, Fastify WebSocket
- **Database**: PostgreSQL with Drizzle ORM
- **Auth**: NextAuth v5 (beta)
- **Storage**: MinIO (dev), S3/UploadThing (prod)
- **Analytics**: PostHog Cloud
- **Rate Limiting**: Arcjet
- **Package Manager**: Bun (primary), supports pnpm/npm
- **Build Tool**: Turborepo, tsup for CLI

## Getting Started

### Initial Setup

```bash
# Clone and install dependencies
git clone https://github.com/arturict/afc-stack.git
cd afc-stack
bun install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Start Docker services (PostgreSQL, MinIO, etc.)
docker compose -f compose.dev.yml up -d

# Generate and run database migrations
bunx drizzle-kit generate
bunx drizzle-kit migrate
```

### Running the Project

```bash
# Start all services in development mode
bun run dev

# Start specific apps
bun run dev --filter=web        # Next.js app only
bun run dev --filter=ws         # WebSocket server only

# Build for production
bun run build

# Run production build
bun run start
```

### Available Scripts

- `bun run dev` - Start all apps in development mode
- `bun run build` - Build all packages and apps
- `bun run lint` - Lint all code
- `bun run lint:fix` - Auto-fix linting issues
- `bun run typecheck` - Type-check all TypeScript
- `bun run test` - Run all tests
- `bun run format` - Format code with Prettier
- `bun run clean` - Clean all build artifacts and dependencies
- `bun run create` - Interactive CLI to create new project
- `bun run add:websocket` - Add WebSocket support to existing project
- `bun run db:generate` - Generate database migrations
- `bun run db:migrate` - Run database migrations
- `bun run db:studio` - Open Drizzle Studio (database GUI)
- `bun run changeset` - Create a new changeset for versioning

### Project Structure

```
afc-stack/
├── apps/
│   ├── web/                    # Next.js frontend application
│   │   ├── src/
│   │   │   ├── app/           # App Router pages and API routes
│   │   │   │   ├── api/       # API endpoints
│   │   │   │   ├── (auth)/    # Auth-related pages
│   │   │   │   └── ...        # Other routes
│   │   │   ├── components/    # React components
│   │   │   ├── lib/           # Utility functions
│   │   │   ├── styles/        # Global styles
│   │   │   └── env.ts         # Environment variable validation
│   │   └── package.json
│   └── ws/                     # WebSocket server (optional)
│       ├── src/
│       │   └── server.ts      # Fastify WebSocket server
│       └── package.json
├── packages/
│   └── db/                     # Shared database package
│       ├── src/
│       │   ├── schema.ts      # Drizzle ORM schemas
│       │   └── index.ts       # Database client exports
│       └── package.json
├── cli-templates/              # Template files for CLI
│   ├── base/                  # Base template files
│   └── extras/                # Optional feature templates
├── .github/
│   ├── workflows/             # GitHub Actions workflows
│   ├── copilot-instructions.md # This file
│   └── ...                    # Other GitHub config
├── .env.example               # Environment variables template
├── compose.dev.yml            # Docker Compose for development
├── turbo.json                 # Turborepo configuration
├── tsconfig.base.json         # Base TypeScript config
└── package.json               # Root workspace config
```

## Common Workflows

### Adding a New Feature

1. **Create a branch**:

    ```bash
    git checkout -b feature/your-feature-name
    ```

2. **Make changes** following the code style guidelines

3. **Test your changes**:

    ```bash
    bun run lint        # Check for linting errors
    bun run typecheck   # Verify TypeScript types
    bun run test        # Run tests
    bun run build       # Ensure it builds
    ```

4. **Create a changeset** (for user-facing changes):

    ```bash
    bun run changeset
    # Select packages, version bump type, and write description
    ```

5. **Commit and push**:

    ```bash
    git add .
    git commit -m "feat: add your feature"
    git push origin feature/your-feature-name
    ```

6. **Create a Pull Request** on GitHub

### Adding a New API Route

1. Create file at `apps/web/src/app/api/[route-name]/route.ts`:

```typescript
import { NextResponse } from "next/server";
import { z } from "zod";
import { db, tableName } from "@ac/db";
import arcjet from "@arcjet/next";

// Define request schema

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arturict) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
