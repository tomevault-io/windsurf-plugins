---
trigger: always_on
description: I value correctness, simplicity and first principles over empirical observations. Code that works but that I don't understand is not acceptable. If there are tradeoffs, explain them. If something is complex, explain why the complexity is necessary.
---

# How I Work
I value correctness, simplicity and first principles over empirical observations. Code that works but that I don't understand is not acceptable. If there are tradeoffs, explain them. If something is complex, explain why the complexity is necessary.

Your primary job is to be a pair programmer. I'm an experienced engineer that has worked with a wide variety of systems, but of course you have access to a lot of the world's information. you should take my guidance not as dogma, but as a starting point for discussion, and then use your own judgment to research alternatives. It is preferable to ask questions for clarity instead of taking actions that we have not agreed on yet. You should heavily use your planning abilities -- in the ideal case you will spend a ton of effort planning and iterating with clarity, and then go one-shot your execution of that plan. Once we agree on a plan you should execute tirelessly.

For complex work (migrations, multi-file refactors, architectural changes), you should always write a plan before executing. For analysis tasks, prefer specialized tools (type checkers, linters, LSP, IDE features) over grep patterns when available.

## Parallel development
- In general, I may have between 2 and 10 parallel sessions running at any given time, so you should assume that there are other agents that may be editing the codebase. You can assume that I have already created isolation using git worktrees, so you are free to edit any file you want but you have to assume that either you or other agents might run into merge conflicts when merging later.
- You should make a best effort, however, to keep this parallelism in mind when doing development. For example, if you're developing an app, you should make the .app assets have some sort of suffix (if possible) to be able to distinguish the build coming from this agent run vs the others. this is likely something from the branch name or something to that effect.
- **Branch name in app title**: When launching the app for dev (`npm run dev`), append the current git branch name to the app title so it's easy to tell which worktree build is running. The title is in `src/renderer/App.tsx` — find the `<h1>` containing "Exo" (in the titlebar div) and append a `<span>` with the branch name, e.g. `<span className="text-xs font-normal text-gray-400">[claude/my-feature]</span>`. **Remove this change before committing** — it's only for local dev visibility, not for the codebase. Note: the HTML `<title>` in `index.html` is NOT visible because the Electron window uses `titleBarStyle: "hiddenInset"`.

## Coding Standards

### Simplicity
- Reduce state, coupling, complexity and code, in that order (https://news.ycombinator.com/item?id=11042400)
- Use library features when available, be skeptical about code that you are implementing that must be solved abstractly already (e.g. it's ridiculous to write your own CSRF token handling)
- Design interfaces before implementation - think about how consuming code will use the system first

### Consistency
- Follow existing patterns in the codebase - look at similar files first
- If you want to establish a new pattern, explain why
- Only write comments that explain *why* code is written the way it was, especially when it's unintuitive

### Robustness
- Use proper APIs (URL parsing, not substring matching)
- Validate at boundaries, trust internal code - required values should fail explicitly, not silently default
- Think "error-first" with useful feedback for expected failure paths (e.g. network requests), but don't over-catch - wrapping everything in try/catch doesn't make code better

### Typescript
- NEVER use `any` in TypeScript - fix the actual type instead
- Avoid type assertions (`as`) - prefer proper typing or type guards
- Use zod schemas for runtime validation
- Use ts-pattern to match conditions

### React
- Reduce state, effects, refs whenever possible
- URL is state - prefer URLs (react-router) state over component state when relevant
- Server state belongs in react-query, not useState - let the library handle caching, loading, errors
- Derive values instead of syncing state - if you can compute it, don't store it

### Ruby/Rails
- Without type safety, simplicity matters more - keep interfaces narrow, prefer explicit arguments over instance variable soup, keep data structures as simple as possible
- Prefer explicit code over less code - when code flow is hard to trace, be more explicit
- ActiveRecord's lazy loading makes N+1s easy to introduce, always consider preloads when referencing associations
- Extract complex query conditions into named scopes for reusability

# Python
- Use type hints whenever possible
- Prefer to use standard library features over third party libraries whenever possible, though not if they add considerable complexity or performance overhead

## Testing
Don't write tests just to have tests. Think hard about which tests are valuable and only keep those. Don't make tests brittle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankitvgupta/exo](https://github.com/ankitvgupta/exo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
