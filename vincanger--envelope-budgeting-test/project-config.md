---
trigger: always_on
description: This document covers advanced Wasp capabilities like Jobs, API Routes, and Middleware, along with performance optimization tips and common troubleshooting steps.
---

# 6. Advanced Features & Troubleshooting

This document covers advanced Wasp capabilities like Jobs, API Routes, and Middleware, along with performance optimization tips and common troubleshooting steps.

## Advanced Features ( [main.wasp](mdc:main.wasp) )

These features are configured in [main.wasp](mdc:main.wasp).

### Jobs and Workers

- Wasp supports background jobs, useful for tasks like sending emails, processing data, or scheduled operations.
- Jobs require a job executor like PgBoss (which requires PostgreSQL, see [3-database-operations.mdc](mdc:.cursor/rules/3-database-operations.mdc) ).
- Example Job definition in [main.wasp](mdc:main.wasp):
  ```wasp
  job emailSender {
    executor: PgBoss, // Requires PostgreSQL
    // Define the function that performs the job
    perform: {
      fn: import { sendEmail } from "@src/server/jobs/emailSender.js"
    },
    // Grant access to necessary entities
    entities: [User, EmailQueue]
  }
  ```
- Jobs can be scheduled or triggered programmatically from Wasp actions or other jobs.
- See [Wasp Jobs Documentation](mdc:https:/wasp-lang.com/docs/advanced/jobs).

### API Routes

- Define custom server API endpoints, often used for external integrations (webhooks, third-party services) where Wasp Operations are not suitable.
- Example API route definition in [main.wasp](mdc:main.wasp):
  ```wasp
  api stripeWebhook {
    // Implementation function in server code
    fn: import { handleStripeWebhook } from "@src/server/apis/stripe.js",
    // Define the HTTP method and path
    httpRoute: (POST, "/webhooks/stripe"),
    // Optional: Grant entity access
    entities: [User, Payment],
    // Optional: Apply middleware
    // middlewares: [checkStripeSignature]
    // Optional: Disable default auth check if webhook handles its own
    // auth: false
  }
  ```
- See [Wasp API Routes Documentation](mdc:https:/wasp-lang.com/docs/advanced/apis).

### Middleware

- Wasp supports custom middleware functions that can run before API route handlers or Page components.
- Useful for logging, custom authentication/authorization checks, request transformation, etc.
- Example Middleware definition in [main.wasp](mdc:main.wasp):
  ```wasp
  // Define the middleware itself
  middleware checkAdmin {
    fn: import { checkAdminMiddleware } from "@src/server/middleware/auth.js"
  }

  // Apply it to a page or API route
  page AdminDashboardPage {
    component: import { AdminDashboard } from "@src/features/admin/AdminDashboardPage.tsx",
    auth: true, // Ensure user is logged in first
    middlewares: [checkAdmin] // Apply custom admin check
  }

  api adminAction {
      fn: import { handleAdminAction } from "@src/server/apis/admin.js",
      httpRoute: (POST, "/api/admin/action"),
      auth: true,
      middlewares: [checkAdmin]
  }
  ```
- See [Wasp Middleware Documentation](mdc:https:/wasp-lang.com/docs/advanced/middleware).

## Performance Optimization

- **Operation Dependencies:** Use specific entity dependencies (`entities: [Task]`) in your Wasp operations ([main.wasp](mdc:main.wasp)) to ensure queries are automatically refetched only when relevant data changes.
- **Pagination:** For queries returning large lists of data, implement pagination logic in your server operation and corresponding UI controls on the client.
- **React Optimization:**
  - Use `React.memo` for components that re-render often with the same props.
  - Use `useMemo` to memoize expensive calculations within components.
  - Use `useCallback` to memoize functions passed down as props to child components (especially event handlers).
- **Optimistic UI Updates (Actions):**
  - For actions where perceived speed is critical (e.g., deleting an item, marking as complete), consider using Wasp's `useAction` hook (from `wasp/client/operations`) with `optimisticUpdates`.
  - This updates the client-side cache (affecting relevant `useQuery` results) *before* the action completes on the server, providing instant feedback.
  - **Use Sparingly:** Only implement optimistic updates where the action is highly likely to succeed and the instant feedback significantly improves UX. Remember to handle potential server-side failures gracefully (Wasp helps revert optimistic updates on error).
  - Example:
    ```typescript
    import { useAction, useQuery } from 'wasp/client/operations';
    import { deleteTask, getTasks } from 'wasp/client/operations'; // Assuming these exist
    import type { Task } from 'wasp/entities';

    function TaskList() {
      const { data: tasks } = useQuery(getTasks);

      // Use useAction when optimistic updates are needed
      const { execute: deleteAction, isExecuting } = useAction(deleteTask, {
        optimisticUpdates: [
          {
            // Specify the query to update optimistically
            getQuerySpecifier: getTasks,
            // Function to update the query cache
            updateQuery: (oldTasks, args) => {
              // args contains { taskId: number } passed to deleteAction
              return oldTasks.filter(task => task.id !== args.taskId);
            }
          }
        ]
      });

      const handleDelete = async (taskId: number) => {
        try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincanger/envelope-budgeting-test](https://github.com/vincanger/envelope-budgeting-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
