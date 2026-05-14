---
trigger: always_on
description: AI-powered video sequence platform built with TanStack Start, optimized for edge deployment.
---

# CLAUDE.md

AI-powered video sequence platform built with TanStack Start, optimized for edge deployment.

## Architecture Overview

**Tech Stack:**

- **Runtime**: Bun (not Node.js)
- **Framework**: TanStack Start + TanStack Router + Vite
- **Database**: Turso (libSQL/SQLite) + Drizzle ORM
- **Workflows**: QStash (durable execution for AI tasks)
- **Storage**: Cloudflare R2 (S3-compatible)
- **Auth**: Better Auth
- **Styling**: Tailwind v4 + shadcn/ui
- **Testing**: Bun test

**Core Principles:**

- Database access ONLY in server handlers (never in components)
- Anonymous-first → upgrade to save work
- Team-based resources (sequences, styles, characters)
- Script-driven generation for consistency

**Data Model:**

```
teams
  ├── users (members)
  ├── sequences (videos)
  │   └── frames (scenes with metadata)
  └── libraries (styles, characters, vfx, audio)
```

---

## Setup

```bash
bun install
bun setup                          # Auto-configure local dev (SQLite + QStash)
bun db:setup                       # Migrate + seed database
```

**Daily workflow (2 terminals):**

- Terminal 1: `bun qstash:dev` (async job processing)
- Terminal 2: `bun dev`

**Before commit:** Lefthook auto-checks quality. Branch `123-feature` → commits tagged `#123`.

---

## Server Handler Pattern

All API routes use TanStack Start server handlers:

```typescript
// src/routes/api/example/$id.ts
import { createFileRoute } from '@tanstack/react-router';
import { json } from '@tanstack/react-start';
import { requireUser } from '@/lib/auth/action-utils';
import { handleApiError } from '@/lib/errors';

export const Route = createFileRoute('/api/example/$id')({
  server: {
    handlers: {
      POST: async ({ params, request }) => {
        try {
          // 1. Validate input
          const input = schema.parse(await request.json());

          // 2. Check auth/team permissions
          const user = await requireUser();

          // 3. Execute business logic (DB operations ONLY here)
          const record = await db.insert(table).values({
            ...input,
            teamId: user.teamId,
          });

          // 4. Trigger workflows for async AI tasks
          const { messageId } = await qstash.publishJSON({
            url: `${getQStashWebhookUrl()}/workflows/image`,
            body: { userId: user.id, teamId: user.teamId, ...input },
          });

          // 5. Return standardized response
          return json({ id: record.id, workflowRunId: messageId });
        } catch (error) {
          const handledError = handleApiError(error);
          return json(
            { success: false, error: handledError.toJSON() },
            { status: handledError.statusCode }
          );
        }
      },
    },
  },
});
```

---

## Workflow Pattern

**Triggering workflows (from server handlers):**

```typescript
// ❌ WRONG - Direct fetch() calls don't include QStash signatures
await fetch('/api/workflows/image', {
  method: 'POST',
  body: JSON.stringify(data),
});

// ✅ CORRECT - Use qstash.publishJSON() for proper signatures
const qstash = getQStashClient();
const { messageId } = await qstash.publishJSON({
  url: `${getQStashWebhookUrl()}/workflows/image`, // External URL QStash can reach
  body: { userId, teamId, prompt, ...params },
});
const workflowRunId = messageId;
```

**Implementing workflows (TanStack Start + serveMany):**

```typescript
// src/routes/api/workflows/$.ts - Register with serveMany
import { createFileRoute } from '@tanstack/react-router';
import { serveMany } from '@upstash/workflow/tanstack';

const handler = serveMany({
  image: generateImageWorkflow,
  motion: generateMotionWorkflow,
  storyboard: generateStoryboardWorkflow,
});

export const Route = createFileRoute('/api/workflows/$')({
  server: {
    handlers: {
      POST: async ({ request }) => {
        return handler.POST({ request });
      },
    },
  },
});

// Individual workflow (src/lib/workflows/image-workflow.ts)
export const generateImageWorkflow = async (
  context: WorkflowContext<ImageWorkflowInput>
) => {
  const input = context.requestPayload;
  validateWorkflowAuth(input); // Check userId/teamId passed through context

  const result = await context.run('generate-image', async () => {
    // Step logic - automatically retried on failure
    const image = await generateImage(input.prompt);

    // Update database directly
    await db
      .update(frames)
      .set({ thumbnailUrl: image.url })
      .where(eq(frames.id, input.frameId));

    return { imageUrl: image.url };
  });

  return result;
};
```

**Key principles:**

- Workflows handle their own state (no DB job tracking needed)
- Pass auth (userId/teamId) through workflow context
- Steps are durable - execution continues even if server restarts
- Update DB records directly in workflow steps

---

## Frame System

Frames are the core content unit - each represents one scene from script analysis.

**Frame Structure:**

- `thumbnailUrl` - Generated image
- `videoUrl` - Motion video (image-to-video)
- `metadata` - Complete `Scene` object (typed JSONB)

**Frame.metadata IS the Scene object** (no wrapper):

```typescript
// src/lib/ai/frame.schema.ts
frame.metadata = {
  sceneId: string,
  sceneNumber: number,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
