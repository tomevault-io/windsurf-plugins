---
trigger: always_on
description: This file contains build, test, lint commands and coding standards for the ClickLens project. It serves as the authoritative reference for all agentic coding agents working in this repository.
---

# AGENTS.md - Codebase Guidelines for ClickLens

This file contains build, test, lint commands and coding standards for the ClickLens project. It serves as the authoritative reference for all agentic coding agents working in this repository.

---

## Build, Lint, and Test Commands

### Development & Production

```bash
bun run dev              # Start Next.js dev server (http://localhost:3000)
bun run build           # Build for production (.next/standalone)
bun run start           # Start production server
```

### Code Quality

```bash
bun run lint            # Run ESLint (Next.js + TypeScript rules)
bun run lint --fix      # Auto-fix linting issues
```

### Unit Tests (Bun Test)

```bash
bun run test                    # Run all tests in src/
bun run test src/path/to/file.test.ts  # Single test file
bun run test --filter "pattern"        # Filter by test name
bun run test:coverage             # With coverage report
```

Test files: `.test.ts`/`.test.tsx`, co-located with source.

### End-to-End Tests (Playwright)

```bash
bun run test:e2e       # Run all E2E tests (chromium, firefox, webkit)
bun run test:e2e:ui    # Playwright UI mode
```

### Pre-commit Hooks

- Husky + lint-staged: auto-lint `*.{ts,tsx,js,jsx}`, test `*.test.{ts,tsx}` on commit

---

## Code Style Guidelines

### TypeScript

- **Strict mode** enabled (`strict: true`)
- **Target**: ES2017, **Module**: ESNext with bundler resolution
- **Path alias**: `@/*` → `./src/*`
- **Exclude**: `**/*.test.ts`, `**/*.spec.ts`, `src/test-setup.ts`

### Imports (ES Modules Only)

Order (top to bottom, alphabetize within groups):

1. External packages
2. Next.js core (`next/server`, etc.)
3. Internal libs (`@/lib/...`)
4. UI components (`@/components/...`)
5. Relative imports from same directory

**Example:**

```typescript
import { NextRequest, NextResponse } from "next/server";
import { getSessionClickHouseConfig } from "@/lib/auth";
import { createClient } from "@/lib/clickhouse";
import { formatQueryError } from "@/lib/errors";
import { StatCard } from "@/components/shared/StatCard";
import { validateSqlStatement } from "./validator";
```

### Naming Conventions

| Element             | Convention       | Example                               |
| ------------------- | ---------------- | ------------------------------------- |
| Components          | PascalCase       | `StatCard.tsx`, `QueryBar`            |
| Functions/variables | camelCase        | `getSessionConfig`, `handleSubmit`    |
| Constants           | UPPER_SNAKE_CASE | `MAX_ROWS`, `QUERY_TIMEOUT_MS`        |
| Types/interfaces    | PascalCase       | `QueryResult`, `UserSession`          |
| Test files          | `name.test.ts`   | `route.test.ts`, `Component.test.tsx` |
| Directories         | kebab-case       | `clickhouse/`, `query-analytics/`     |

### React/Next.js Patterns

- **Default**: Server components (no `"use client"` directive)
- **Add `"use client"`** only when using: state (`useState`, `useEffect`), browser APIs (`window`), event handlers, React context
- **App Router**: `app/` with layouts/pages, API routes in `app/api/`, loading.tsx, error.tsx
- **Styling**: Tailwind CSS utility classes only. Use Radix UI primitives. No inline styles or CSS modules.
- **Semantic tokens**: `foreground`, `background`, `primary`, `border`, etc.

### API Routes (Next.js App Router)

**Standard pattern:**

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  try {
    // 1. CSRF/authentication
    const csrfError = await requireCsrf(request);
    if (csrfError) return csrfError;

    // 2. Authorization
    const authError = await checkPermission("canExecuteQueries");
    if (authError) return authError;

    // 3. Validation
    const body = await request.json();
    if (!body.sql) {
      return NextResponse.json(
        { success: false, error: formatHttpError(400) },
        { status: 400 },
      );
    }

    // 4. Main logic
    const result = await processQuery(body.sql);
    return NextResponse.json({ success: true, data: result });
  } catch (error) {
    // 5. Centralized error handling
    const formatted = formatQueryError(error, undefined, true);
    return NextResponse.json(
      { success: false, error: formatted },
      { status: 500 },
    );
  }
}
```

**Error response format:**

```typescript
{
  success: false;
  error: {
    code: number;           // HTTP status or error code
    type: string;           // "SYNTAX", "SCHEMA", "PERMISSION", etc.
    message: string;        // Technical (sanitized in prod)
    userMessage: string;    // User-friendly
    hint?: string;          // Optional help
  };
}
```

**Streaming:** Use NDJSON for large datasets (>1000 rows) with backpressure handling. Headers: `Content-Type: application/x-ndjson; charset=utf-8`.

### Error Handling

- Use `formatQueryError` from `@/lib/errors` for ClickHouse errors
- Categorizes errors: SYNTAX, SCHEMA, TYPE, PERMISSION, RESOURCE, FUNCTION, SYSTEM, NETWORK, UNKNOWN
- Sanitizes sensitive data in production (file paths, IPs, credentials, stack traces)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ntk148v/clicklens](https://github.com/ntk148v/clicklens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
