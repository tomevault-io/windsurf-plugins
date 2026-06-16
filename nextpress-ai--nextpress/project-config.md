---
trigger: always_on
description: - No classes/OOP — functions, factories, functional composition. Extend via composition not modification. ES6+, 400 LOC/file max unless absolutely necessary.
---

# Agent Coding Rules

## Core

- No classes/OOP — functions, factories, functional composition. Extend via composition not modification. ES6+, 400 LOC/file max unless absolutely necessary.
- Factory pattern: `createUser()` returns `{ activate, deactivate, ... }` — plain object with methods, not `class User`
- Names: `verbNoun`, `isActive`, `CONSTANTS`, `kebab-case.ts`
- Domain folders with barrel `index.ts`: `/users/get-users.ts, update-users.ts → index.ts`
- JSDoc for all public APIs (explain WHY not what). Comments only where intent unclear
- Critical/missing-dep errors → `throw` (language-native). Runtime/user errors → handle gracefully using project's error handling + logging patterns
- Code for humans: readability over cleverness. Many small functions > one monolith

## Patterns

- Single responsibility. DI via params (unified context object). Guard clauses early return. Object lookup > switch
- Contract-based boundaries: types define expected shapes, enforced at module boundaries
- Prefer `type` over `interface`, ban `enum`. Explicit return types on public functions
- Co-locate standalone modules. Abstract only when reused. Extraction before 3 uses = premature
- No barrel re-exports across domains — circular deps, slow type-checking
- `safeTry` over try/catch. `.filter().map()` over for loops
- `{ name, email }` not `(name, email, ...)` basically named params in an object vs positional unless param is single.
- Defensive: anticipate failure. Critical harm → throw fast. Known failure paths → handle gracefully

## React

- State logic → hooks. Components pure presentational. Containers handle state, pure components render props
- Prop drilling max 2 levels. Use context/state management beyond
- Check `data?.length > 0` before `.map()`. `useMemo`, `useCallback`, `React.memo`
- Build complex UIs from small, focused components

## Philosophy

- **Plan first** → systematize → implement. Never rush. Break complex tasks into phases
- **Follow my strategy exactly**, suggest before deviating. I approve changes
- **Occam's Razor**: simplest solution that works. Add complexity only when needed
- **Existing patterns first**: work with what's there. Don't rewrite when fix works
- **Reflect after tasks**: what worked, what failed, what learned. QA your own work
- **Multi-agent collaboration**: delegate to other agents where possible
- **Document decisions**: record reasoning behind changes, tradeoffs, impacts
- **Agree on tradeoffs before committing**: I approve tradeoffs before they hit the codebase
- **Keep code taste matching mine**: your code style should blend with existing
- **Be brief**: conciseness over verbosity in communication
- **Not demo/prototypes**: real thing or don't do it. One clear way to do something

### Think Outside the Box

When the obvious approach isn't working after 2 attempts, stop and reconsider. The repeated failure is a signal — either escalate to me for direction, or find a creative alternative. Don't tunnel-vision on one approach. If you're patching the same thing twice, the fix is probably wrong. Ask: "what am I assuming that might be wrong?"

### Think in Systems

Every change ripples. Before implementing, map the blast radius:

- What contracts (types, APIs, DB schemas) does this break or require changes to?
- What downstream consumers depend on the current behavior?
- Does this stabilize or destabilize the system? A fix that introduces new edge cases is worse than the bug.
- Is the tradeoff worth it? Document it. Get approval before committing.
- Trace the full call chain, not just the immediate function. A change in `utils.ts` affects every caller.

### Think in First Principles

When a problem recurs or fixes don't stick, don't apply more patches. Break it down:

- What is the base assumption? Is it actually true? (e.g., "safeTry preserves Result nesting" — it doesn't)
- Can we isolate and test the assumption independently?
- Divide and conquer: split the problem into independently verifiable pieces. Prove each piece works before composing.
- If the foundation is wrong, no amount of upper-layer fixes will hold. Fix the foundation first.

## Libraries

- `slang-ts` (safeTry/result), `z-fetch` (fetch), `regist` (regex)
- Understand library edge cases: read source enough to know when behavior diverges from types
- Check Context7 MCP for library docs. Need more? Ask me. Source in node_modules if you want to inspect directly
- agent-browser CLI for frontend debugging. `agent-browser -h` for usage
- MCPs/tools/agents available → use them. Skills available → load them when relevant. Ask me if unsure how
- Use caveman skill if available, but not its ultra version!

## Boundaries — DO NOT CROSS WITHOUT APPROVAL

- **Servers**: you never start. Ask me
- **DB**: all db commands/decisions → ask me first
- **Git**: ask before any git command
- **.env**: never read/edit. Env vars → config module only, never `process.env` direct
- New env var needed: suggest → I approve → I add to config + .env myself
- **/backend, /nile, frontend/api**: never touch without approval
- **Installs/stack changes**: never without permission
- **Project PM** (not npm) for package commands

## Safety
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextpress-ai/nextpress](https://github.com/nextpress-ai/nextpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
