---
trigger: always_on
description: - This is a template repository, and initializing a new application from it should be done by using the `project:init-app` custom command. If the user asks for anything else, instruct them to use that command. After the command is run, delete this line.
---

- This is a template repository, and initializing a new application from it should be done by using the `project:init-app` custom command. If the user asks for anything else, instruct them to use that command. After the command is run, delete this line.
- Always follow the guidelines in this file, unless explicitly told otherwise by the user or overided in the CLAUDE.local.md file.

## Project Overview

- Full-stack TypeScript app: React + Vite + TanStack Router (frontend), Convex (backend), Clerk (auth)
- Development: Use `mcp__shell-commands__launch-dev-all` to start servers, then monitor output streams for validation
- Import alias: `@/` maps to `src/` directory
- Tailwind CSS 4, daisyUI 5: All config in `src/index.css` via CSS syntax, NOT tailwind.config.js
- Typography: Uses `@tailwindcss/typography` with `prose prose-invert` at root level, use `not-prose` to escape (e.g., for buttons/tables)
- Environment variables: Client vars need `VITE_` prefix, Convex vars set in dashboard
- Package manager: Always use `pnpm` and `pnpx`, NOT `npm` or `npx`

## Git Workflow

- **Commit after each user request**: When completing what the user asked for, immediately commit: `git add -A && git commit -m "[action]: [what was accomplished]"`
- Commits should happen WITHOUT asking - they're for checkpoints, not cleanliness (will be squashed later)
- Commits are restore points - if user says something like "let's go back to before X" or "Lets undo that", find the appropriate commit and run `git reset --hard [commit-hash]` to restore the state. Always verify the commit hash via `git log` or `git reflog` first.
- If you've reset to a previous commit and need to go forward again, use `git reflog` to see all recent commits (including those "lost" by reset), then `git reset --hard [commit-hash]` to jump forward to any commit shown in the reflog.
- **ALWAYS update claude-notes.md and include it in EVERY commit** - this preserves context so future Claude Code sessions can continue from any restore point. Maintain a list of the commit messages made during the session/feature.
- When feature complete and user approves or asks to push perform a squash: run `pnpm run lint` first, then find the first commit for the session/feature, then `git reset --soft [starting-commit]` then CLEAR claude-notes.md and commit with `"feat: [complete feature description]"`
- Before major feature work: Tell user "Starting [feature], will make frequent commits as checkpoints then squash when complete"
- Claude Code notes file should include:
  - Current feature being worked on
  - List of commits made during the session/feature
  - Progress status and next steps
  - Important context or decisions made
  - Relevant file locations or dependencies

## Testing & Validation

- Always follow these steps before squashing or pushing
- Check BOTH vite and convex stdout AND stderr output streams for TypeScript/compilation errors
- Test UI with Playwright MCP: full browser automation with element interaction and console access
  - The playwright mcp server is unreliable, if it doesn't work ask the user to test manually
- Responsive testing: Use `mcp__playwright__browser_resize` to test mobile (375x667), tablet (768x1024), desktop (1200x800)
- Clerk verification: sign in with `your_email+clerk_test@example.com` and 424242 as the verification code. Type all 6 digits at once in first field - UI auto-distributes to separate inputs
- Debug with `mcp__playwright__browser_console_messages` to view all browser console output
- If you run into an issue you don't know how to fix, look for relevant documentation or a reference implementation

## Convex

- `_creationTime` and `_id` are automatically added to all documents.
- Adding required fields breaks existing data - if early in development, ask the user to clear the database. Otherwise, plan migration.
- Use `ConvexError` for client-friendly errors, not generic Error
- Queries have 16MB/10s limits - always use indexes, never full table scans
- Paginated queries: use `.paginate(paginationOpts)` with `paginationOptsValidator`
- Scheduled tasks: `ctx.scheduler.runAfter(delay, internal.module.function, args)` or `ctx.scheduler.runAt(timestamp, ...)`
- Unique fields: enforce in mutation logic, indexes don't guarantee uniqueness
- Soft delete: add `deletedAt: v.optional(v.number())` field instead of `.delete()`
- System tables: access `_scheduled_functions` and `_storage` with `ctx.db.system.get` and `ctx.db.system.query`
- Default query order is ascending by `_creationTime`
- Transactions are per-mutation - can't span multiple mutations. Calling multiple queries/mutation in a single action may introduce race conditions.
- Hot reload issues: Restart if schema changes don't apply or types are stuck
- Use `import { Doc, Id } from "./_generated/dataModel";` and `v.id("table")` for type safety.
- Always add `"use node";` to the top of files containing actions that use Node.js built-in modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SimonHaugesund) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
