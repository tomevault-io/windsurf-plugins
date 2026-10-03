---
trigger: always_on
description: - When talking to the user, sacrifice grammar for concision. Drop articles, punctuation, capitalization — whatever shaves words without losing meaning.
---

Session rules:

- When talking to the user, sacrifice grammar for concision. Drop articles, punctuation, capitalization — whatever shaves words without losing meaning.
- If the next action is non-destructive and naturally follows from the user's request, do it proactively. Do not wait for approval or turn-by-turn feedback.
- Keep moving through safe next steps until the task is done or you hit a real blocker.
- Greenfield codebase, no users to preserve — rip things out and reshape data structures freely; ship the cleanest version of the change. Hold every line to production quality: rigorous code, thoughtful UX, real product thinking. Scope stays narrow; craft stays high.
- No overengineering. Pick the simplest approach that fits the codebase and the task.
- Product feel matters: the app should feel fast, snappy, and local-first. Prefer proper caching, preloading, optimistic updates, and reuse of warm UI/runtime state so users rarely see loading states.
- Use `better-result` as a typed workflow toolkit, not just an error wrapper. Model rich product/domain outcomes in `Result.ok(...)`; reserve `Result.err(...)` for recoverable operation failures callers must handle. Prefer `Result.gen`, `Result.await`, `andThen`, `mapError`, `tryRecover`, `tap`/`tapBoth`, and boundary `.match(...)`/`matchError(...)` over scattered `isOk()`/`isErr()` branching, nullable sentinels, booleans, or thrown domain exceptions.
- Never use `try/catch`.
- React `useEffect` is banned. Use loaders, server functions, event handlers, subscriptions, or derived state instead.
- For API and server boundary validation, prefer shared Zod schemas and use `drizzle-zod` where DB-backed shapes should stay in sync with Drizzle schema.
- For schema migrations, edit the Drizzle schema first and run `pnpm --filter @garden/db db:generate`. Do not hand-write raw schema SQL. Raw SQL is only acceptable for data migrations or hand-authored data repair/backfill steps that Drizzle cannot generate.
- Only invoke skills once you understand the problem surface — never preemptively. First read the relevant files, confirm what the task actually touches, then decide if a skill applies. If you invoke a skill, state which one and why it fits the verified surface.
- When listing available skills to the user, do it as a check — name only the ones whose trigger conditions match what you've already verified about the task. Do not dump the full skill list or claim a skill is relevant before you've looked at the code.
- The `better-result` skill applies when you've confirmed the change touches TypeScript Result workflows, error handling, domain outcomes, callback statuses, retries, serialization, or tagged errors. Read its bundled `opensrc/better-result` source before non-trivial changes. Don't invoke it for unrelated edits (docs, config, non-TS files).
- Don't start, stop, kill, or restart local servers casually. If you genuinely need a running server (e.g. to verify a fix end-to-end, smoke-test a UI change, or reproduce a bug), reuse an already-running port first; only start your own when nothing's listening. Always tear down anything you started.
- Use the dedicated root `pnpm dev` orchestration for local development. Executor connector APIs and MCP Durable Objects are part of the Garden web Worker; there is no separate connector or MCP-proxy service to boot.
- Do not use `git stash` by default. Multiple agents may be working concurrently on this branch and a stash hides their in-flight work from them. If the user explicitly asks to stash, do it. Otherwise, stage clean commits by path with `git add <files>` and let unrelated dirty files stay in the working tree.
- Commit after every major change. Stage only the files relevant to that change with `git add <files>` and write a focused commit message — don't batch unrelated work into one commit, and don't leave large completed work uncommitted.

TanStack docs:

- This repo has TanStack skill docs installed inside `node_modules/@tanstack/**/skills/**/SKILL.md`.
- Read those before changing TanStack Start, Router, auth guards, loaders, server functions, or related routing patterns.
- Useful commands:
  - `sed -n '1,260p' node_modules/@tanstack/router-core/skills/router-core/SKILL.md`
  - `sed -n '1,260p' node_modules/@tanstack/start-client-core/skills/start-core/SKILL.md`
  - `sed -n '1,260p' node_modules/@tanstack/router-core/skills/router-core/auth-and-guards/SKILL.md`
  - `find node_modules/@tanstack -path '*/skills/*/SKILL.md' | sort | sed -n '1,260p'`

Terminology:

- "Context menu" or "explore menu" refers to the inner rail (the context rail tied to the active tab), not the sidebar rail.

Cloudflare Agents SDK / MCP storage:

- Don't monkey-patch SDK manager methods (`MCPClientManager.restoreConnectionsFromStorage`, etc.). The SDK methods reference `this.sql`/`this.storage` — destructuring or reassigning them silently breaks `this` binding and the failure surfaces deep inside an alarm dispatch with `Cannot read properties of undefined (reading 'sql')`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flow-Research/garden](https://github.com/Flow-Research/garden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
