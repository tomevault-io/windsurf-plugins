---
trigger: always_on
description: Guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

Guidance to Claude Code when working with code in this repository.


## Architecture Overview

Big-AGI is a Next.js 15 application with a sophisticated modular architecture built for professional AI interactions.

### Development Commands

Dev servers may be already running on ports 3000, 3001, 3002, or 3003 (not always this app - other projects may occupy these ports). Never start or stop dev servers, let the user do it.

```bash
# Validate (~5s, safe while dev server runs, do NOT use `next build` ~45s for same checks)
tsc --noEmit --pretty && npm run lint # Type check (~3.5s) + ESLint (~2s)
eslint src/path/to/file.ts           # Lint specific file

# Full build (~60s+, only when suspecting runtime/bundle issues)
npm run build  # next build runs compile+lint+types but stops at first type-error file; tsc shows all at once

# Database & External Services
# npm run supabase:local-update-types   # Generate TypeScript types
# npm run stripe:listen                 # Listen for Stripe webhooks
```

For AI protocol development (model listing, live API requests/responses, parameter probing), real vendor API keys are in `.env.api-keys` if present (Anthropic, OpenAI, Gemini, etc., one VENDOR_API_KEY per line). Use them for empirical verification; never commit or echo the values.

### Git/GitHub remotes

The `gh` command is available to interact with GitHub from the terminal, but **NEVER PUSH TO ANY BRANCH**. The user manages all 'write' git operations.
- `opensource` -> `enricoros/big-AGI` (public, default branch: `main`, MIT) - community issues/PRs/releases
- `private` -> `big-agi/big-agi-private` (private, default branch: `dev`) - main dev repo with `dev`->`staging`->`prod` pipeline
- **Always use `git mv` instead of `mv`** when renaming or moving files - preserves git history tracking
- **NEVER run `git stash`** - it causes work loss
- **Commit subjects**: `Area: terse imperative` (e.g. `LLMs: OpenAI: ...`, `Build: ...`), single line, no body unless needed, and no `Co-Authored-By` trailer

**Branch contents:**
- `main` is the open-source build: local-first, BYO-keys, full AIX and provider coverage
- `dev` extends `main` with the hosted/cloud layer: auth, Zync sync, Cloud Fabric, Stripe, multi-tenant, admin pages, it's the way to go for users, the best user experience of any multi-model chat application
- Cloud/auth/sync code stays on `dev`; non-cloud improvements (UX, AIX, model support, bug fixes) can land on either branch

**Branch workflow:**
- `dev` is rebased on top of `main` (never merged) - `main` changes flow into `dev` on the next rebase, no manual forward-port needed
- Never `git merge` between the two branches - breaks the linear topology
- Backporting `dev` -> `main` is a re-implementation, never a cherry-pick - keep `main`-side edits minimal/additive so the existing `dev` version lands cleanly on rebase; split into small commits when natural
- Rebasing `dev` onto `main`: work on a scratch branch (never `private/dev` directly); only files `main` changed since the merge-base can conflict - forecast with `git diff --name-only $(git merge-base private/dev opensource/main) opensource/main`
- Resolve that rebase per-conflict: keep `dev` where it diverges, UNION where `main` only added (never blanket `-X theirs`/`-X ours` - they drop `main`'s additions). Check no `<<<<<<<` markers survive before each `--continue`

### Core Directory Structure

You are started from the root of the repository (i.e. where the git folder is or scripts should be run from).
**ISSUE ALL COMMANDS FROM THE ROOT, OMITTING 'cd' COMMANDS. DO NOT CHAIN CD AND OTHER COMMANDS**
**NEVER RUN COMPOUND `cd` COMMANDS LIKE `cd some-folder && command` - ONLY RUN `command` FROM THE ROOT, ALWAYS.**
The directory structure is as follows:

```
/app/api/          # Next.js App Router (API routes only, mostly -> /src/server/)
/pages/            # Next.js Pages Router (file-based, mostly -> /src/apps/)
/src/
├── apps/          # Feature applications (self-contained modules)
├── modules/       # Reusable business logic and integrations
├── common/        # Shared infrastructure and utilities
└── server/        # Backend API layer with tRPC
/kb/               # Knowledge base for modules, architectures
```

### Key Technologies

- **Frontend**: Next.js 15, React 18, Material-UI Joy, Emotion (CSS-in-JS)
- **State Management**: Zustand with localStorage/IndexedDB (single cell) persistence
- **API Layer**: tRPC with TanStack React Query for type-safe communication
- **Runtime**: Edge Runtime for AI operations, Node.js for data processing

### "Apps" Architecture Pattern

Each app in `/src/apps/` is a self-contained feature module:
- Main component (`App*.tsx`)
- Local state store (`store-app-*.ts`)
- Feature-specific components and layouts
- Runtime configurations

Example apps: `chat/`, `call/`, `beam/`, `draw/`, `personas/`, `settings-modal/`

### Modules Architecture Pattern

Modules in `/src/modules/` provide reusable business logic:
- **`aix/`** - AI communication framework for real-time streaming
- **`beam/`** - Multi-model AI reasoning system (scatter/gather pattern)
- **`blocks/`** - Content rendering (markdown, code, images, etc.)
- **`llms/`** - Language model abstraction supporting 20+ vendors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enricoros/big-AGI](https://github.com/enricoros/big-AGI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
