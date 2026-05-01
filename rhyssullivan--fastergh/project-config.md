---
trigger: always_on
description: Confect (Effect + Convex) and Effect-Atom patterns for type-safe RPC and reactive state management
---


# Confect & Effect-Atom Patterns

This codebase uses **Confect** (`@packages/confect`) for Effect + Convex integration and **Effect-Atom** (`@effect-atom/atom`) for reactive state management in React.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        React Components                      │
│  useAtomValue(atom) / useAtom(atom) / usePaginatedAtom()   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    RPC Client (effect-atom)                  │
│  guestbookClient.list.subscription({})                      │
│  guestbookClient.add.mutate                                  │
│  guestbookClient.listPaginated.paginated(pageSize)          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Convex Functions (RPC)                    │
│  factory.query / factory.mutation / factory.action          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Confect Context (Effect)                    │
│  ConfectQueryCtx / ConfectMutationCtx / ConfectActionCtx    │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 1: Confect (Backend - Convex Functions)

### 1.1 Schema Definition

Use Effect Schema with Confect's `defineTable` and `defineSchema`:

```typescript
// packages/database/convex/schema.ts
import { defineSchema, defineTable } from "@packages/confect/schema";
import { Schema } from "effect";

const UserSchema = Schema.Struct({
  name: Schema.String,
  email: Schema.String,
});

const PostSchema = Schema.Struct({
  title: Schema.String,
  content: Schema.String,
  authorId: Schema.String,
  published: Schema.Boolean,
});

export const confectSchema = defineSchema({
  users: defineTable(UserSchema).index("by_email", ["email"]),
  posts: defineTable(PostSchema)
    .index("by_authorId", ["authorId"])
    .index("by_published", ["published"]),
});

export default confectSchema.convexSchemaDefinition;
```

### 1.2 Context Setup

Create typed context tags for your schema:

```typescript
// packages/database/convex/confect.ts
import {
  ConfectMutationCtx as ConfectMutationCtxTag,
  type ConfectMutationCtx as ConfectMutationCtxType,
  ConfectQueryCtx as ConfectQueryCtxTag,
  type ConfectQueryCtx as ConfectQueryCtxType,
  ConfectActionCtx as ConfectActionCtxTag,
  type ConfectActionCtx as ConfectActionCtxType,
} from "@packages/confect/ctx";
import { type TablesFromSchemaDefinition } from "@packages/confect/schema";
import { confectSchema } from "./schema";

export { confectSchema };

type Tables = TablesFromSchemaDefinition<typeof confectSchema>;

export const ConfectQueryCtx = ConfectQueryCtxTag<Tables>();
export type ConfectQueryCtx = ConfectQueryCtxType<Tables>;

export const ConfectMutationCtx = ConfectMutationCtxTag<Tables>();
export type ConfectMutationCtx = ConfectMutationCtxType<Tables>;

export const ConfectActionCtx = ConfectActionCtxTag<Tables>();
export type ConfectActionCtx = ConfectActionCtxType<Tables>;
```

### 1.3 RPC Module Definition

Define type-safe RPC endpoints with Effect:

```typescript
// packages/database/convex/rpc/guestbook.ts
import { createRpcFactory, makeRpcModule } from "@packages/confect/rpc";
import {
  Cursor,
  PaginationOptionsSchema,
  PaginationResultSchema,
} from "@packages/confect";
import { Effect, Schema } from "effect";
import { ConfectMutationCtx, ConfectQueryCtx, confectSchema } from "../confect";

const factory = createRpcFactory({ schema: confectSchema });

const Entry = Schema.Struct({
  _id: Schema.String,
  _creationTime: Schema.Number,
  name: Schema.String,
  message: Schema.String,
});

class EmptyFieldError extends Schema.TaggedError<EmptyFieldError>()(
  "EmptyFieldError",
  { field: Schema.String },
) {}

const guestbookModule = makeRpcModule({
  list: factory.query({ success: Schema.Array(Entry) }, () =>
    Effect.gen(function* () {
      const ctx = yield* ConfectQueryCtx;
      const entries = yield* ctx.db.query("guestbook").order("desc").take(50);
      return entries.map((e) => ({
        _id: String(e._id),
        _creationTime: e._creationTime,
        name: e.name,
        message: e.message,
      }));
    }),
  ),

  listPaginated: factory.query(
    {
      payload: PaginationOptionsSchema.fields,
      success: PaginationResultSchema(Entry),
    },
    (args) =>
      Effect.gen(function* () {
        const ctx = yield* ConfectQueryCtx;
        const result = yield* ctx.db.query("guestbook").order("desc").paginate({
          cursor: args.cursor,
          numItems: args.numItems,
        });
        return {
          page: result.page.map((e) => ({
            _id: String(e._id),
            _creationTime: e._creationTime,
            name: e.name,
            message: e.message,
          })),
          isDone: result.isDone,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RhysSullivan/fastergh](https://github.com/RhysSullivan/fastergh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
