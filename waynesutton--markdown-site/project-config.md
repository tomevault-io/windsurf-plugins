---
trigger: always_on
description: Guidelines for preventing write conflicts when using React, useEffect, and Convex
---


# Preventing Write Conflicts in Convex with React

Write conflicts occur when two functions running in parallel make conflicting changes to the same table or document. This rule provides patterns to avoid these conflicts.

## Understanding Write Conflicts

According to [Convex documentation](https://docs.convex.dev/error#1), write conflicts happen when:

1. Multiple mutations update the same document concurrently
2. A mutation reads data that changes during execution
3. Mutations are called more rapidly than Convex can execute them

Convex uses optimistic concurrency control and will retry mutations automatically, but will eventually fail permanently if conflicts persist.

## Backend Protection: Idempotent Mutations

### Always Make Mutations Idempotent

Mutations should be safe to call multiple times with the same result.

**Good Pattern:**

```typescript
export const completeTask = mutation({
  args: { taskId: v.id("tasks") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const task = await ctx.db.get(args.taskId);

    // Early return if document doesn't exist
    if (!task) {
      return null;
    }

    // Early return if already in desired state
    if (task.status === "completed") {
      return null;
    }

    // Only update if state change is needed
    await ctx.db.patch(args.taskId, {
      status: "completed",
      completedAt: Date.now(),
    });

    return null;
  },
});
```

**Bad Pattern:**

```typescript
// This will cause conflicts if called multiple times rapidly
export const completeTask = mutation({
  args: { taskId: v.id("tasks") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const task = await ctx.db.get(args.taskId);
    // No check if already completed
    await ctx.db.patch(args.taskId, {
      status: "completed",
      completedAt: Date.now(),
    });
    return null;
  },
});
```

### Avoid Unnecessary Reads - Patch Directly

When you only need to update fields, patch directly without reading first. Database operations throw if the document doesn't exist.

**Good Pattern:**

```typescript
export const updateNote = mutation({
  args: { id: v.id("notes"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Patch directly without reading first
    // ctx.db.patch throws if document doesn't exist
    await ctx.db.patch(args.id, { content: args.content });
    return null;
  },
});
```

**Bad Pattern:**

```typescript
export const updateNote = mutation({
  args: { id: v.id("notes"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Reading the document first creates a conflict window
    const note = await ctx.db.get(args.id);
    if (!note) throw new Error("Not found");

    // When typing rapidly, multiple mutations fire
    // Each reads the same version, then all try to write, causing conflicts
    await ctx.db.patch(args.id, { content: args.content });
    return null;
  },
});
```

### Minimize Data Reads

Only read the data you need. Avoid querying entire tables when you only need specific documents.

**Good Pattern:**

```typescript
export const updateUserCount = mutation({
  args: { userId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Only query tasks for this specific user
    const tasks = await ctx.db
      .query("tasks")
      .withIndex("by_user", (q) => q.eq("userId", args.userId))
      .collect();

    await ctx.db.patch(args.userId, {
      taskCount: tasks.length,
    });

    return null;
  },
});
```

**Bad Pattern:**

```typescript
export const updateUserCount = mutation({
  args: { userId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Reading entire table creates conflicts with any task change
    const allTasks = await ctx.db.query("tasks").collect();
    const userTasks = allTasks.filter((t) => t.userId === args.userId);

    await ctx.db.patch(args.userId, {
      taskCount: userTasks.length,
    });

    return null;
  },
});
```

### Use Indexes to Reduce Read Scope

Always define and use indexes to limit the scope of data reads.

```typescript
// In schema.ts
tasks: defineTable({
  userId: v.string(),
  status: v.string(),
  content: v.string(),
}).index("by_user", ["userId"])
  .index("by_user_and_status", ["userId", "status"]),
```

## Frontend Protection: Preventing Duplicate Calls

### Use Refs to Track Mutation Calls

When mutations should only be called once per state change, use refs to track calls.

**Good Pattern:**

```typescript
export function TimerComponent() {
  const [session, setSession] = useState(null);
  const hasCalledComplete = useRef(false);
  const completeSession = useMutation(api.timer.completeSession);

  useEffect(() => {
    if (timeRemaining <= 0 && session && !hasCalledComplete.current) {
      hasCalledComplete.current = true;
      completeSession({ sessionId: session._id });
    }
  }, [timeRemaining, session, completeSession]);

  // Reset ref when starting new session
  const handleStartNewSession = async () => {
    hasCalledComplete.current = false;
    await startSession();
  };

  return <div>...</div>;
}
```

**Bad Pattern:**

```typescript
export function TimerComponent() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waynesutton/markdown-site](https://github.com/waynesutton/markdown-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
