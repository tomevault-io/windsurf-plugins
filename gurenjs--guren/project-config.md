---
trigger: always_on
description: Guren is a Laravel-inspired fullstack TypeScript framework running on Bun. It combines Hono for HTTP handling, Drizzle ORM for database operations, and Inertia.js for seamless frontend integration.
---

# Guren Framework

## Overview
Guren is a Laravel-inspired fullstack TypeScript framework running on Bun. It combines Hono for HTTP handling, Drizzle ORM for database operations, and Inertia.js for seamless frontend integration.

**Status:** Alpha (v0.2.x) - Breaking changes expected.

## Monorepo Structure

```
packages/
├── core/           # Framework entry point, aggregates other packages
├── server/         # HTTP server (Hono), routing, controllers, middleware, auth
├── orm/            # ORM abstraction with Drizzle adapter, Model API
├── cli/            # CLI commands (make:*, db:*, routes:types, AI agent commands)
├── testing/        # Testing utilities for controllers and HTTP
├── create-app/     # Project scaffolding tool
└── inertia-client/ # Frontend React + Inertia.js integration

examples/
└── blog/           # Reference application

web/                # Documentation site
```

## Development Commands

```bash
# Build all packages (required after code changes)
bun run build

# Run tests
bun run test:bun      # Framework unit tests
bun run test:examples # Example app tests
bun run test          # Full test suite

# Type checking
bun run typecheck

# Development server (blog example)
bun run dev

# Database
bun run db:up         # Start PostgreSQL container
bun run db:down       # Stop container
bun run db:migrate    # Run migrations
bun run db:seed       # Run seeders
```

## Build Order & Troubleshooting

`bun run build` executes packages sequentially in dependency order:
`testing → orm → server → openapi → cli → core → create-app → inertia`

**Stale `.d.ts` issue:** If DTS build fails (e.g., `@guren/core` cannot find `@guren/server` types), old `dist/` artifacts are likely interfering. Run:
```bash
bun run build:clean   # rm -rf packages/*/dist && bun run build
```

**Rule:** Always use `bun run build:clean` instead of `bun run build` when:
- Building after switching branches
- Building after pulling large changes
- DTS build fails with "could not find declaration file" errors

## Package-Specific Builds

```bash
bun run build:server  # Build @guren/server
bun run build:orm     # Build @guren/orm
bun run build:cli     # Build @guren/cli
# etc.
```

## AI Agent Commands

Commands designed for AI coding agents to understand, validate, and generate code:

```bash
# Project introspection
bunx guren context              # Project context map (markdown)
bunx guren context --json       # Project context map (JSON)
bunx guren model:list           # List models with relationships
bunx guren model:list --format json  # Models as JSON

# Integrity checking
bunx guren check                # Validate route↔controller↔page consistency
bunx guren check --json         # Check results as JSON
bunx guren doctor --next        # Doctor report + actionable next steps

# Code generation
bunx guren guidelines           # Auto-generate project-specific coding guidelines
bunx guren guidelines -o .claude/rules/project-guidelines.md  # Write to file
bunx guren make:feature Post --fields "title:string,body:text,published:boolean"  # CRUD scaffold
bunx guren make:feature Post --fields "title:string,body:text" --test  # With test file
```

## Coding Conventions

### TypeScript
- **Strict mode** enabled (`strict: true`)
- **ES2022** target with ESNext modules
- **Bundler** module resolution
- Use **Bun native APIs** where applicable
- **No CommonJS** - ESM only

### File Organization
- Test files: `*.test.ts` alongside source files
- Index exports: Each package has `src/index.ts` as main entry
- Type declarations: Generated via tsup build

### Naming
- **Classes:** PascalCase (e.g., `UserController`, `PostModel`)
- **Files:** kebab-case for utilities, PascalCase for classes
- **Variables/functions:** camelCase
- **Constants:** UPPER_SNAKE_CASE for true constants

### Imports
```typescript
// Use package aliases
import { Controller } from '@guren/server'
import { Model } from '@guren/orm'

// Relative imports within same package
import { helper } from './utils'
```

## Architecture Patterns

### Controllers
```typescript
import { Controller } from '@guren/core'
import { z } from 'zod'
import { pages } from '@/.guren/pages.gen'

const CreatePostSchema = z.object({
  title: z.string().min(1),
  body: z.string().min(1),
})

const PostIdParamSchema = z.object({
  id: z.coerce.number().int().positive(),
})

export class PostController extends Controller {
  async index() {
    const posts = await Post.all()
    return this.inertia(pages.posts.Index, { posts })
  }

  async show() {
    const { id } = this.validateParams(PostIdParamSchema)
    const post = await Post.findOrFail(id)  // throws 404 automatically
    return this.inertia(pages.posts.Show, { post })
  }

  async store() {
    const data = await this.validateBody(CreatePostSchema)  // throws 422 on failure
    const user = await this.auth.userOrFail()  // throws 401 if unauthenticated
    const post = await Post.create({ ...data, authorId: user.id })
    return this.redirect('/posts')
  }
}
```

**Controller validation helpers** (accepts any Zod-like schema with `safeParse`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gurenjs/guren](https://github.com/gurenjs/guren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
