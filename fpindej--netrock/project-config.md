---
trigger: always_on
description: NETrock - .NET 10 API (Clean Architecture) + SvelteKit frontend (Svelte 5), fully dockerized.
---

# CLAUDE.md

NETrock - .NET 10 API (Clean Architecture) + SvelteKit frontend (Svelte 5), fully dockerized.

**The backend API is the core of the project.** It is a public-facing API designed to serve any client - multiple frontends, mobile apps, other backends, third-party integrations. The SvelteKit frontend is a fully functional reference client, not a throwaway. Treat every API change as if unknown consumers depend on it.

```
Frontend (SvelteKit :5173) → /api/* proxy → Backend API (.NET :8080) → PostgreSQL / MinIO
Backend layers: WebApi → Application ← Infrastructure → Domain + Shared
```

## Hard Rules

### Backend

- `Result`/`Result<T>` for all fallible operations - never throw for business logic
- `TimeProvider` (injected) - never `DateTime.UtcNow` or `DateTimeOffset.UtcNow`
- C# 13 `extension(T)` syntax for new extension methods
- Never `null!` - fix the design instead
- `ProblemDetails` (RFC 9457) for all error responses - never anonymous objects or raw strings
- `internal` on all Infrastructure service implementations
- `/// <summary>` XML docs on all public and internal API surface
- `System.Text.Json` only - never `Newtonsoft.Json`
- NuGet versions in `Directory.Packages.props` only - never in `.csproj`

### Frontend

- Never hand-edit `v1.d.ts` - run `pnpm run api:generate`
- Svelte 5 Runes only - never `export let`
- `interface Props` + `$props()` - never `$props<{...}>()`
- Logical CSS only: `ms-*`/`me-*`/`ps-*`/`pe-*` - never `ml-*`/`mr-*`/`pl-*`/`pr-*`
- No `any` - define proper interfaces
- Feature folders in `$lib/components/{feature}/` with barrel `index.ts`
- Use shadcn-svelte components (`pnpm dlx shadcn-svelte@latest add <name>`) - never build what shadcn already provides
- Pixel-perfect responsiveness - mobile, tablet, desktop, ultrawide, landscape and portrait
- Touch targets minimum 44px on all interactive elements
- Unified UX - reuse existing components and patterns so the app feels like one product, not five
- No overflow - dialogs, modals, and pages must never show scrollbars. Fit content to viewport.

### Cross-Cutting

- Security first - when convenience and security conflict, choose security. Deny by default, open selectively. Full PII compliance (`users.view_pii` permission, server-side masking, no PII in logs/URLs/errors).
- Atomic commits: `type(scope): imperative description` (Conventional Commits). No `Co-Authored-By` lines.
- No dead code - remove unused imports, variables, functions, files, and stale references in the same commit
- No em dashes - never use `—` anywhere (code, comments, docs, UI). Use `-` or rewrite the sentence.

## Delegation Rule

The top-level agent is an orchestrator. It does not write application code in `src/` - that goes to specialized agents.

**Default (application code in `src/`):**
- Delegate implementation to `backend-engineer`, `frontend-engineer`, or `fullstack-engineer`
- Run relevant reviewers in parallel after implementation completes
- Run `filemap-checker` after modifying files with known consumers

**Orchestrator handles directly (no delegation needed):**
- Documentation, configuration, and tooling files (`.claude/`, `CLAUDE.md`, `FILEMAP.md`, `.gitignore`, `docs/`, CI/CD)
- Quick answers, planning, research, and code review
- Commits, PRs, and git operations

**User override:** If the user explicitly asks to skip delegation ("do it yourself", "directly", "don't delegate", "just fix it"), the orchestrator implements directly regardless of scope.

## Agent Team

All application code in `src/` goes to specialized agents. User override is the only exception (see Delegation Rule). Run reviewers in parallel after every implementation.

| Agent | Role | When to use |
|---|---|---|
| `backend-engineer` | Implements .NET features | Task stays within `src/backend/` |
| `frontend-engineer` | Implements SvelteKit features | Task stays within `src/frontend/` |
| `fullstack-engineer` | Implements cross-stack features | Task touches both backend and frontend |
| `backend-reviewer` | Audits C# code (read-only) | Reviewing backend changes |
| `frontend-reviewer` | Audits Svelte code (read-only) | Reviewing frontend changes |
| `ux-designer` | Audits UI/UX design quality (read-only) | Checking responsiveness, visual consistency, theming |
| `security-reviewer` | Audits for vulnerabilities (read-only) | Auth, permissions, PII, tokens, middleware changes |
| `devops-engineer` | Implements infra changes | Dockerfiles, Aspire, CI/CD, health checks, env vars |
| `devops-reviewer` | Audits infra/deployment (read-only) | Dockerfiles, compose, Aspire, CI/CD changes |
| `test-writer` | Writes tests | Tests needed alongside implementation |
| `filemap-checker` | Verifies downstream updates (read-only) | After modifying files with known consumers |
| `tech-writer` | Writes documentation | READMEs, session docs, guides |
| `product-owner` | Proposes prioritized work items (read-only) | Deciding what to work on next, backlog review |

**Delegation patterns:**
- **New backend feature**: `backend-engineer` implements, then `backend-reviewer` + `security-reviewer` audit in parallel
- **New frontend feature**: `frontend-engineer` implements, then `frontend-reviewer` + `ux-designer` audit in parallel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fpindej/netrock](https://github.com/fpindej/netrock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
