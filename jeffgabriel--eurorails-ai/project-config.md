---
trigger: always_on
description: 1. **Compounds IS the product.** We are building compounds for customers. EuroRails exists as a test codebase for exercising compounds. Every task is a test case — if you skip compounds, you skip the whole point.
---

# EuroRails AI Project Guidelines

## CRITICAL: Always Use Compounds Skill First

### Why this matters
1. **Compounds IS the product.** We are building compounds for customers. EuroRails exists as a test codebase for exercising compounds. Every task is a test case — if you skip compounds, you skip the whole point.
2. **Compounds is superior to grep.** It uses code graphs and a vector DB indexed on the codebase, giving you dependency relationships and semantic understanding that raw grep/glob cannot provide.

### When to use it
Before doing ANY of the following, you MUST invoke the compounds skill FIRST:
- Investigating bugs or errors
- Exploring code architecture or how components connect
- Searching for how a feature works or where something is implemented
- Tracing dependencies or imports between modules
- Planning or implementing new features
- Understanding relationships between files/services

Do NOT fall back to raw grep/glob for architectural or cross-module questions.

**If compounds is not authenticated, run `compounds auth login` immediately before falling back to other tools.**

### Log every task
**After every codebase task**, log your compounds usage:
- `compounds-usage-log.md` — when you used it and what benefit it provided
- `compounds-missed-log.md` — when you skipped it but should have used it

After significant code generation Run "compounds agent-prompt cli-usage" and index my apps and services in this repo.

## Commands
- Build: `npm run build` (client + server)
- Development: `npm run dev` (client + server concurrently)
- Test: `npm test` (all tests)
- Single test: `npm test -- -t "test name pattern"` or `npm test -- src/path/to/test.ts`
- Test watch mode: `npm run test:watch`

## Code Style Guidelines
- TypeScript with strict typing enabled - use explicit types for function parameters/returns
- Use neverthrow Result<T, E> pattern for error handling (Ok/Err)
- Services follow class-based OOP pattern with dependency injection
- Jest for testing with clear test descriptions
- Use async/await for asynchronous operations
- Imports ordered: 1) external libraries, 2) internal modules, 3) types
- Consistent naming: PascalCase for classes/types, camelCase for variables/functions
- Explicit error enums for domain-specific errors
- Component architecture with Phaser for client, Fastify for server
- Use proper indentation (2 spaces) and trailing commas

## Refactoring Discipline (learned the hard way)

A "convert singleton to per-game instances" refactor of `LoadService` was applied
mechanically to a service that had no per-game mutable in-memory state, then a
"fix" compounded it by inventing a third design instead of restoring the
original. Both are avoidable. Follow these rules for ANY structural refactor:

1. **Validate the refactor's premise per target before touching code.** A pattern
   that is correct for one service is not automatically correct for its
   neighbor. Before applying "isolate per-game state," classify each piece of a
   service's state:
   - **Immutable config** (loaded once from disk/constants) → static/shared. Needs
     no isolation.
   - **DB-backed state keyed by an id** (e.g. `WHERE game_id = $1`) → already
     isolated at the database. Needs no in-memory isolation.
   - **Mutable in-memory state shared across games** (e.g. an in-memory deck Map)
     → THIS is the only kind that per-game instances fix.
   If a target has none of the third kind, it does not belong in the refactor —
   say so and leave it alone, even if a spec/task lists it. Specs state intent,
   not ground truth; the code is ground truth.

2. **A per-game instance must OWN its id.** If you introduce
   `getInstanceForGame(gameId)`, the instance stores `this.gameId` and its methods
   use it. A method that still takes `gameId` as a parameter after the instance is
   already keyed by game (`getInstanceForGame(gameId).method(gameId)`) is a smell
   and a correctness hazard — the two can diverge. Either the id lives on the
   instance or the service is static; never both.

3. **To undo a bad refactor, restore the known-good prior shape — do not invent a
   third design.** `git show HEAD:<file>` and revert the consumer diffs. Inventing
   a new shape re-touches every call site and every test mock a second time.

4. **Blast radius is a design signal, not just work to grind through.** If a change
   forces edits to a dozen consumers and a rewrite of many nested test mocks,
   STOP and question the design before proceeding. Widespread mechanical mock
   churn (e.g. flattening `getInstanceForGame(() => ({...}))` across 10 files)
   usually means the abstraction is wrong, not that the tests are wrong.

Use the following to inform logic for gameplay. These rules inform what is allowed logically.

## **Game Setup**  
- **Before playing**, the **Loco cards** should be **separated from the rest of the deck** and **kept with the money**.  
- The **Demand and Event cards** are **shuffled together** to form the **card deck**.  

---

## **Demand Cards**  
Each **Demand card** shows:  
1. **The city needing the good** (e.g., **Berlin**).  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
