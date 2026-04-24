---
trigger: always_on
description: **NEVER use `chrome-devtools_*` tools directly in the main conversation.**
---

# AGENTS.md

## ⚠️ CRITICAL: Chrome DevTools = Subagent ONLY

**NEVER use `chrome-devtools_*` tools directly in the main conversation.**

Chrome DevTools dumps massive snapshots that exhaust context. Always spawn a subagent:

```
Task(subagent_type="explore", description="Debug via Chrome DevTools", prompt="...")
```

---

## Project Overview

**opencode-next** - Next.js 16 rebuild of OpenCode web application.

Turborepo monorepo transitioning from SolidJS to Next.js 16+ with React Server Components. Uses effect-atom based reactive World Stream for state management, with ai-elements chat UI.

**Architecture Philosophy:**
- **Core owns computation, React binds UI** - Smart boundary pattern (ADR-016)
- **World Stream is THE API** - Push-based reactive state via `createWorldStream()` (ADR-018)
- **Router is DELETED** - 4,377 LOC removed, it was solving the wrong problem

**Why Next.js 16?**
- Flat RSC hierarchy (eliminates 13+ nested providers)
- Better mobile patterns (React hooks map to scroll behavior)
- 30-40% code reduction with ai-elements
- React is 10x more common than SolidJS (easier hiring)

---

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Runtime** | Bun | Fast, 10x faster installs |
| **Testing** | Vitest | Isolated tests, proper ESM support |
| **Framework** | Next.js 16 canary | RSC, App Router, Turbopack |
| **Monorepo** | Turborepo | Incremental builds |
| **Language** | TypeScript 5+ | Type safety |
| **Linting** | oxlint | Fast Rust-based linter |
| **Formatting** | Biome | Fast Prettier replacement |
| **Chat UI** | ai-elements | Battle-tested React chat components |
| **Styling** | Tailwind CSS | Utility-first CSS |

---

## Development Commands

```bash
# Setup
bun install

# Dev
bun dev
bun build

# Type check (MANDATORY - checks full monorepo)
bun run typecheck          # Runs: turbo type-check

# Code quality
bun lint
bun format
bun format:fix

# Testing (use Vitest, NOT bun test - better isolation)
bun run test              # Runs: vitest run
bun run test:watch        # Runs: vitest
bun run test:coverage     # Runs: vitest run --coverage
```

**CRITICAL:** 
- Always `bun run typecheck` from repo root before committing
- Never use `bun test` directly - Vitest has proper isolation, Bun test leaks state

---

## Conventions

### Changesets (Non-Negotiable)

**Every changeset MUST include:**

1. **A relevant quote from pdf-brain** - Search for wisdom related to your change
2. **Sick ASCII art** - Creative, memorable

```bash
# Example changeset:
---
"@opencode-vibe/core": minor
---

feat(api): add session caching layer

```
    ╔═══════════════════════════════════════╗
    ║  🧠 CACHE ME OUTSIDE, HOW BOW DAH 🧠  ║
    ╠═══════════════════════════════════════╣
    ║     ┌─────────┐                       ║
    ║     │ REQUEST │──┐                    ║
    ║     └─────────┘  │  ┌───────┐         ║
    ║                  ├──│ CACHE │         ║
    ║     ┌─────────┐  │  └───────┘         ║
    ║     │ BACKEND │──┘                    ║
    ║     └─────────┘                       ║
    ╚═══════════════════════════════════════╝
```

> "The purpose of abstraction is not to be vague, but to create
> a new semantic level in which one can be absolutely precise."
> — Dijkstra, via pdf-brain

Adds LRU cache for session lookups, reducing backend calls by 80%.
```

**ASCII Art Inspiration:**

```
# The Swarm 🐝
       \   /
    `. _\|/_ .'
    - ( o bg) -    bzzzz
    .' /|\  `.    
       / | \      ╭─────────────╮
      🐝 🐝 🐝    │ HIVE MIND   │
     🐝 🐝 🐝 🐝  │ ACTIVATED   │
    🐝 🐝 🐝 🐝 🐝 ╰─────────────╯

# The Phoenix Refactor 🔥
          ,//
         ///
        ////
       /////
    ,///////
    ////////
     '////'
       '||'
        ||  FROM THE ASHES
        ||  OF LEGACY CODE
       /||\  WE RISE
      //||\\

# The Crab Rave 🦀
    (\/) (°,,°) (\/)
      RUST SAYS:
    "MEMORY SAFE, BB"
    
    ╱|、
  (˚ˎ 。7  
   |、˜〵          
   じしˍ,)ノ

# The Kraken Release 🦑
       ___
    .-'   '-.
   /  .-=-.  \   RELEASE
  |  /     \  |    THE
  | |  O O  | |   KRAKEN
   \|  ___  |/
    '.___.'
   /||||||||\
  //||||||||\\
```

### TDD (Non-Negotiable)

```
RED → GREEN → REFACTOR
```

**Every feature. Every bug fix. No exceptions.**

1. **RED** - Write failing test first
2. **GREEN** - Minimum code to pass
3. **REFACTOR** - Clean up while green

**Bug fixes:** Write test that reproduces bug FIRST, then fix. Prevents regression forever.

**NO DOM TESTING.** If the DOM is in the mix, we already lost.

- `renderHook` and `render` from `@testing-library` are code smells
- Test pure functions and hooks logic directly
- Test state management (Zustand stores) in isolation
- Test API/SDK integration with mocks
- Use E2E tests (Playwright) for actual UI verification

**USE VITEST, NOT BUN TEST.** Bun test has poor isolation - Zustand stores and singletons leak state between tests causing flaky failures.

### Fix Broken Shit (Non-Negotiable)

```
FIND IT → FIX IT → DON'T BLAME OTHERS
```

**If you encounter broken code, fix it. No excuses.**

1. Pre-existing type errors? Fix them.
2. Failing tests unrelated to your task? Fix them or file a cell.
3. Broken imports? Fix them.
4. Dead code? Delete it.

**What NOT to do:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhooks/opencode-vibe](https://github.com/joelhooks/opencode-vibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
