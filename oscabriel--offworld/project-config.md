---
trigger: always_on
description: **Offworld** — One skill for your whole stack. Builds a persistent clone map, generates right-sized references, installs one routing skill across all agents.
---

## OVERVIEW

**Offworld** — One skill for your whole stack. Builds a persistent clone map, generates right-sized references, installs one routing skill across all agents.

Turborepo monorepo: CLI (`ow`), web app (offworld.sh), docs, TUI.

## ARCHITECTURE

### Single-Skill Model

```
~/.local/share/offworld/skill/offworld/
├── SKILL.md           # Static routing skill
├── assets/
│   └── map.json       # Canonical clone map
└── references/
    ├── tanstack-router.md
    └── ...
```

- **One global skill** (static, never changes)
- **Canonical map** at `assets/map.json`
- **Per-repo references** under `references/`
- **Project map** at `.offworld/map.json` for scoped routing

### Data Flow

```
ow project init
  -> parse manifests
  -> resolve deps to repos
  -> clone repos
  -> generate references
  -> update assets/map.json
  -> write .offworld/map.json
  -> symlink single offworld skill
```

## STRUCTURE

```
offworld/
├── apps/
│   ├── cli/         # CLI (offworld / ow command)
│   ├── web/         # Web app - TanStack Start + Convex + WorkOS
│   ├── docs/        # Astro Starlight documentation
│   └── tui/         # OpenTUI terminal app
├── packages/
│   ├── sdk/         # Core logic (clone, generate, sync, agents)
│   ├── types/       # Zod schemas + TypeScript types
│   ├── backend/     # Convex functions + schema
│   └── config/      # Shared tsconfig
```

## WHERE TO LOOK

| Task                | Location                        | Notes                         |
| ------------------- | ------------------------------- | ----------------------------- |
| Add CLI command     | `apps/cli/src/handlers/`        | Handler + route in `index.ts` |
| Add SDK function    | `packages/sdk/src/`             | Export from `index.ts`        |
| Add Zod schema      | `packages/types/src/schemas.ts` | Export type from `types.ts`   |
| Add Convex function | `packages/backend/convex/`      | Query/mutation/action         |
| Add web route       | `apps/web/src/routes/`          | TanStack Router file-based    |
| Add web component   | `apps/web/src/components/`      | shadcn/ui + Tailwind          |
| Add agent support   | `packages/sdk/src/agents.ts`    | Agent registry                |

## KEY FILES

### CLI (`apps/cli/`)

- `src/cli.ts` — Entry point
- `src/index.ts` — Router definition (trpc-cli + @orpc/server)
- `src/handlers/*.ts` — Command implementations

### SDK (`packages/sdk/`)

- `src/config.ts` — Config load/save, path utilities
- `src/clone.ts` — Git clone/update/remove
- `src/index-manager.ts` — Global/project map management
- `src/generate.ts` — AI reference generation
- `src/agents.ts` — Agent registry (6 agents)
- `src/sync.ts` — Convex client for push/pull
- `src/auth.ts` — WorkOS token management
- `src/repo-source.ts` — Parse repo input (URL, owner/repo, local)
- `src/manifest.ts` — Dependency parsing (package.json, etc.)
- `src/reference-matcher.ts` — Match deps to installed references

### Backend (`packages/backend/convex/`)

- `schema.ts` — Tables: reference, repository, pushLog, user
- `references.ts` — CRUD for references
- `admin.ts` — Admin functions
- `github.ts` — GitHub API queries
- `validation/` — Push arg + content + GitHub validators

### Web (`apps/web/`)

- `src/routes/index.tsx` — Landing page
- `src/routes/explore.tsx` — Browse shared references
- `src/routes/_github/$owner_.$repo/` — Repo detail page
- `src/routes/admin.tsx` — Admin dashboard
- `src/components/home/` — Landing page components
- `src/components/repo/` — Repo display components

## CONVENTIONS

- **Linting**: Oxlint + Oxfmt (NOT eslint/prettier)
- **Package manager**: Bun
- **Imports**: `@/` → `apps/web/src/` (web only)
- **TypeScript**: Strict mode
- **Schemas**: Zod in `packages/types`, infer types

## COMMANDS

```bash
bun install              # Install deps
bun run dev              # Start all apps
bun run dev:web          # Web app only
bun run dev:server       # Convex backend only
bun run build            # Build all
bun run build:cli        # Build CLI + link globally
bun run check            # Oxlint + Oxfmt
bun run typecheck        # TypeScript check
bun run test             # Run tests
```

## CLI COMMANDS

| Command              | Handler       | Description                   |
| -------------------- | ------------- | ----------------------------- |
| `ow pull <repo>`     | `pull.ts`     | Clone + generate reference    |
| `ow generate <repo>` | `generate.ts` | Force local AI generation     |
| `ow push <repo>`     | `push.ts`     | Upload to offworld.sh         |
| `ow list`            | `list.ts`     | List managed repos            |
| `ow rm <repo>`       | `remove.ts`   | Remove repo + reference       |
| `ow init`            | `init.ts`     | Interactive global setup      |
| `ow project init`    | `project.ts`  | Scan deps, install references |
| `ow config *`        | `config.ts`   | Config management             |
| `ow auth *`          | `auth.ts`     | WorkOS authentication         |

## DATA PATHS

| Purpose      | Location                                                 |
| ------------ | -------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oscabriel/offworld](https://github.com/oscabriel/offworld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
