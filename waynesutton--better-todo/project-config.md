---
trigger: always_on
description: Security guidelines for the better-todo Convex application. Activate with @sec-check to audit code for vulnerabilities, scan for PII exposure, and verify auth patterns.
---


# Security Guidelines for better-todo

This document covers security patterns specific to the better-todo application, a Convex-powered todo and notes app with Clerk authentication.

## App-Specific Security Context

### Authentication Provider

- **Auth**: Clerk (via `ctx.auth.getUserIdentity()`)
- **User ID**: `identity.subject` (Clerk subject ID)
- **Config**: `convex/auth.config.ts`

### Sensitive Tables

| Table              | Contains PII               | Auth Required | Notes                                      |
| ------------------ | -------------------------- | ------------- | ------------------------------------------ |
| `users`            | email, firstName, lastName | Yes           | User profile data                          |
| `todos`            | No                         | Yes           | User content                               |
| `notes`            | No                         | Yes           | User content                               |
| `fullPageNotes`    | No                         | Yes           | May be shared publicly via `isShared` flag |
| `aiChats`          | No                         | Yes           | AI conversation history                    |
| `pomodoroSessions` | No                         | Optional      | Supports guest users                       |
| `streaks`          | No                         | Yes           | User progress tracking                     |
| `statistics`       | No                         | No            | Aggregate counts only                      |

## 1. Authentication Patterns for better-todo

### Standard Auth Check Pattern

All mutations and queries that access user data must verify identity:

```typescript
// REQUIRED pattern for all user-data functions
const identity = await ctx.auth.getUserIdentity();
if (!identity) {
  throw new Error("Not authenticated"); // For mutations
  // OR return []; / return null; for queries
}
const userId = identity.subject;
```

### Ownership Verification Pattern

Use indexed queries to verify ownership:

```typescript
// GOOD: Index-based ownership check
const todo = await ctx.db
  .query("todos")
  .withIndex("by_user", (q) => q.eq("userId", userId))
  .filter((q) => q.eq(q.field("_id"), args.todoId))
  .unique();

if (!todo) {
  throw new Error("Todo not found or unauthorized");
}

// BAD: Fetch then check (exposes existence)
const todo = await ctx.db.get(args.todoId);
if (todo?.userId !== userId) throw new Error("Forbidden");
```

### Pomodoro Sessions (Guest Support)

Pomodoro allows guests, so auth is optional:

```typescript
const identity = await ctx.auth.getUserIdentity();
const userId = identity?.subject; // May be undefined for guests

const session = await ctx.db
  .query("pomodoroSessions")
  .withIndex("by_user", (q) =>
    userId ? q.eq("userId", userId) : q.eq("userId", undefined),
  )
  .filter((q) => q.eq(q.field("_id"), args.sessionId))
  .unique();
```

## 2. Public vs Private Content

### Shared Notes Security

Full-page notes can be shared publicly via `shareSlug` and `isShared` flag:

```typescript
// Public query - no auth required but must check isShared
export const getNoteBySlug = query({
  handler: async (ctx, args) => {
    const note = await ctx.db
      .query("fullPageNotes")
      .withIndex("by_shareSlug", (q) => q.eq("shareSlug", args.slug))
      .first();

    // CRITICAL: Only return if explicitly shared
    if (!note || !note.isShared) {
      return null;
    }

    return note;
  },
});
```

### Statistics (Aggregate Data)

The `statistics` table stores aggregate counts and is safe for public access:

```typescript
// OK: Aggregate stats don't expose user data
export const getDatabaseStats = internalQuery({
  handler: async (ctx) => {
    // Returns counts only, no user-specific data
    return {
      totalTodos: allTodos.length,
      totalNotes: allNotes.length,
    };
  },
});
```

## 3. Function Registration Guide

### Public Functions (api.\*)

Use for user-facing operations with auth checks:

- `todos.ts`: All functions (getTodosByDate, createTodo, updateTodo, etc.)
- `notes.ts`: All functions
- `fullPageNotes.ts`: Most functions
- `folders.ts`: All functions
- `aiChats.ts`: getAIChatByDate, addUserMessage, clearChat, deleteChat
- `pomodoro.ts`: All functions (with optional auth for guest support)
- `search.ts`: searchAll
- `stats.ts`: getUserStats

### Internal Functions (internal.\*)

Use for backend operations and scheduled tasks:

- `aiChats.ts`: addAssistantMessage, getAIChatInternal
- `fullPageNotes.ts`: getSharedNoteMetadata
- `streaks.ts`: updateStreak
- `stats.ts`: getDatabaseStats

### Public Without Auth (by design)

- `fullPageNotes.ts`: getNoteBySlug, checkSlugAvailability (shared content)
- `stats.ts`: getStats action (aggregate public stats)

## 4. Schema Security Notes

### Users Table

```typescript
users: defineTable({
  userId: v.string(),      // Clerk subject (not PII)
  email: v.string(),       // PII - never expose in public queries
  firstName: v.string(),   // PII - never expose in public queries
  lastName: v.string(),    // PII - never expose in public queries
}).index("by_userId", ["userId"]),
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/better-todo](https://github.com/waynesutton/better-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
