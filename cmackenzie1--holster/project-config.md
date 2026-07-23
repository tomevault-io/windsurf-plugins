---
trigger: always_on
description: A Paperless-ngx clone built on Cloudflare Workers, R2, and PostgreSQL via Hyperdrive.
---

# Paperless Worker

A Paperless-ngx clone built on Cloudflare Workers, R2, and PostgreSQL via Hyperdrive.

## Tech Stack

- **Framework**: TanStack React Start (SSR-capable React framework)
- **Routing**: TanStack Router (file-based routing)
- **Runtime**: Cloudflare Workers
- **Database**: PostgreSQL via Hyperdrive (Drizzle ORM)
- **Storage**: Cloudflare R2 for document files
- **Styling**: Tailwind CSS 4.0

## Project Structure

```
src/
├── components/        # React components (Header, etc.)
├── db/
│   ├── index.ts      # DB connection factory (createDbFromHyperdrive)
│   └── schema/       # Drizzle ORM schemas
│       ├── documents.ts
│       ├── files.ts
│       ├── tags.ts
│       ├── correspondents.ts
│       └── incoming-emails.ts
├── routes/           # File-based routes (TanStack Router)
│   ├── __root.tsx    # Root layout
│   ├── index.tsx     # Dashboard
│   ├── documents.$id.tsx
│   ├── api.upload.ts
│   ├── api.files.$key.ts
│   ├── api.documents.$id.ts
│   ├── api.documents.$id.tags.ts
│   ├── api.tags.ts
│   └── api.tags.$id.ts
├── router.tsx        # Router configuration
└── routeTree.gen.ts  # Auto-generated route tree
drizzle/              # Database migrations
```

## SSR Architecture: Server vs Client Code

This is an SSR application. **Cloudflare bindings (HYPERDRIVE, R2) are only available in server-side code.**

### Server-Side Only (Can Access Bindings)

Use these patterns when you need database or R2 access:

**1. Server Functions (`createServerFn`)**
```typescript
import { createServerFn } from "@tanstack/react-start";
import { env } from "cloudflare:workers";

const getDocuments = createServerFn({ method: "GET" }).handler(async () => {
  const db = createDbFromHyperdrive(env.HYPERDRIVE);
  return db.select().from(documents);
});
```

**Server Functions with Input Validation**

Use `createMiddleware` with `inputValidator` - NOT `.validator()` on the server function:

```typescript
import { createServerFn, createMiddleware } from "@tanstack/react-start";

const myMiddleware = createMiddleware({ type: "function" })
  .inputValidator((data: { id: string }) => data)
  .server(({ next }) => next());

const getDocument = createServerFn({ method: "GET" })
  .middleware([myMiddleware])
  .handler(async ({ data }) => {
    // data.id is typed
  });

// Call with: getDocument({ data: { id: "123" } })
```

**2. Route Loaders**
```typescript
export const Route = createFileRoute("/")({
  component: Dashboard,
  loader: () => getDocuments(), // Executes on server
});
```

**3. API Route Handlers**
```typescript
import { env } from "cloudflare:workers";

export const Route = createFileRoute("/api/upload")({
  server: {
    handlers: {
      POST: async ({ request }) => {
        // Access env.HYPERDRIVE, env.R2 here
        return json({ success: true });
      }
    }
  }
});
```

### Client-Side Only

- React component interactivity (useState, useEffect, event handlers)
- Browser APIs (localStorage, fetch to API routes)
- Client-side navigation

### Hydrated (Runs Both)

- React components render on server first, then hydrate on client
- Route components with `useLoaderData()` - data comes from server

## Cloudflare Bindings

Configured in `wrangler.jsonc`:

| Binding | Type | Description |
|---------|------|-------------|
| `HYPERDRIVE` | Hyperdrive | PostgreSQL connection proxy |
| `R2` | R2 Bucket | Document file storage (`paperless-documents`) |
| `EMAIL` | SendEmail | Email sending capability (for future use) |
| `CF_VERSION_METADATA` | WorkerVersionMetadata | Worker version ID and tag (git SHA) |

**Environment Variables**:

| Variable | Description |
|----------|-------------|
| `ALLOWED_EMAIL_SENDERS` | Comma-separated list of allowed sender email addresses for email import. Empty = allow all. |

Access via:
```typescript
import { env } from "cloudflare:workers";
// Use env.HYPERDRIVE, env.R2, env.ALLOWED_EMAIL_SENDERS, etc. directly
```

## Database

**ORM**: Drizzle with PostgreSQL dialect

**Key Tables**:
- `documents` - Document metadata (title, content, ASN, dates)
- `files` - File records linked to documents (R2 object keys, hashes)
- `tags` - Categorization tags with colors
- `document_tags` - Many-to-many join table
- `correspondents` - Sender/source entities
- `incoming_emails` - Email import audit log (sender, status, documents created)

**Commands**:
```bash
pnpm db:generate  # Generate migrations from schema changes
pnpm db:push      # Push schema to database
pnpm db:studio    # Open Drizzle Studio
```

## R2 Storage Pattern

Files are stored with keys like: `documents/{timestamp}/{filename}`

```typescript
// Upload
await env.R2.put(objectKey, file, {
  httpMetadata: { contentType: file.type }
});

// Retrieve
const object = await env.R2.get(objectKey);
```

## Email Import

The worker supports importing documents via email. Emails are processed by the `email()` handler in `server.ts`.

**How it works:**
1. Email arrives at configured address (set up via Cloudflare Email Routing)
2. Sender is checked against `ALLOWED_EMAIL_SENDERS` (rejected if not allowed)
3. Raw email is stored in R2 at `emails/{timestamp}/{sender}.eml`
4. Attachments are extracted using `postal-mime`
5. Each attachment becomes a document with its filename as the title

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmackenzie1/holster](https://github.com/cmackenzie1/holster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
