---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md — Deploy Center (Server)

Guidance for Claude Code (claude.ai/code) when working in this repository.
**Audience:** AI agent only. Project users should read [`docs/README.md`](./docs/README.md).

**Last Updated:** 2026-05-24 (v3.0.0 GA)

---

## 📋 Project Identity

**Deploy Center** — enterprise-grade self-hosted CI/CD deployment platform.
Monorepo with `server/` (Node.js + Express + TypeScript + Sequelize) and
`client/` (React 19 + TypeScript + Material-UI + Vite).

- **Current version:** v3.0.0 (server) / v3.0.0 (client) — released 2026-05-24.
- **Release history:** [`docs/CHANGELOG.md`](./docs/CHANGELOG.md).
- **Roadmap:** [`docs/ROADMAP.md`](./docs/ROADMAP.md) — every feature has a
  stable `F-NNN` ID mapped to a target version. **Do not guess release dates;
  trust this file.**
- **Per-version specs:** [`docs/versions/`](./docs/versions/) (v3.0 → v5.0).
- **Tech stack & exact versions:** authoritative source is
  [`package.json`](./package.json) + [`client/package.json`](../client/package.json).
  Read those, don't trust restated values in any doc.

---

## 📁 Documentation Location Convention (MANDATORY)

> **⚠️ Strict rule:** every documentation file in this project lives under
> `server/docs/`. Do **not** create `.md` files in the root, in `client/`, or
> anywhere else.

### Allowed paths

| File type                              | Required location                |
|----------------------------------------|----------------------------------|
| Master Roadmap                         | `server/docs/ROADMAP.md`         |
| Changelog (release history)            | `server/docs/CHANGELOG.md`       |
| Per-version specs                      | `server/docs/versions/vX.Y-*.md` |
| User guides                            | `server/docs/guides/*.md`        |
| Architecture / API / Design docs       | `server/docs/*.md`               |
| Screenshots & assets                   | `server/docs/screenshots/`       |

### Allowed exceptions

- `CLAUDE.md` — stays in `server/CLAUDE.md` (AI-agent instructions).
- `README.md` — root entry point for GitHub.
- `LICENSE.md` — root (npm + GitHub auto-detect).
- **GitHub community files** (`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`,
  `SECURITY.md`, `SUPPORT.md`, `AUTHORS.md`) live in `server/.github/` — the
  GitHub UI reads them from there with higher priority than the root and
  surfaces them in Insights / Security / Community tabs.

### When you create new documentation

1. **Check first:** is there an existing file to update instead of creating new?
2. **If new:** put it under `server/docs/` (or an appropriate subfolder).
3. **If you find docs in the wrong place:** move them to `server/docs/` and
   tell Sabry.

---

## 🛠️ Path Aliases (server `tsconfig.json`)

Use these in imports — never use deep relative paths like `../../../Models/`:

- `@Controllers/` → `server/src/Controllers/`
- `@Services/` → `server/src/Services/`
- `@Models/` → `server/src/Models/`
- `@Middleware/` → `server/src/Middleware/`
- `@Routes/` → `server/src/Routes/`
- `@Utils/` → `server/src/Utils/`
- `@Types/` → `server/src/Types/`
- `@Database/` → `server/src/Database/`
- `@Config/` → `server/src/Config/`
- `@Migrations/` → `server/src/Migrations/`

---

## 🎯 Coding Conventions (summary)

Full rules in [`docs/CODING_STANDARDS.md`](./docs/CODING_STANDARDS.md).

### Naming

| Construct                  | Convention                        | Example                  |
|----------------------------|-----------------------------------|--------------------------|
| Classes / Types            | PascalCase                        | `DeploymentService`      |
| Interfaces                 | `I` + PascalCase                  | `IUserAttributes`        |
| Enums                      | `E` + PascalCase                  | `EDeploymentStatus`      |
| Functions / methods / vars | camelCase                         | `getUserById`            |
| Constants                  | UPPER_SNAKE_CASE                  | `MAX_RETRIES`            |
| DB columns                 | PascalCase                        | `CreatedAt`, `ProjectId` |
| Class-method handlers      | PascalCase (`public Handle =`)    | `Authenticate`           |
| Service-class files        | PascalCase.ts                     | `QueueService.ts`        |
| Utility files              | camelCase.ts                      | `logger.ts`              |

### Hard rules

- **No `any`** — use explicit types or proper generics.
- **No `console.log`** in production code — always Winston `Logger`.
- **No raw SQL** except for complex aggregations; everything else through Sequelize.
- **All passwords:** bcrypt (10 rounds minimum).
- **All secrets at rest:** AES-256-GCM with a unique IV per record
  (reuse [`Utils/EncryptionHelper.ts`](./src/Utils/EncryptionHelper.ts)).
- **All API endpoints:** protected by `AuthMiddleware` unless explicitly public.
- **Sensitive endpoints:** also gated by `RoleMiddleware` with explicit roles.
- **All rate-limited routes:** wrap with `RateLimiterMiddleware.ApiLimiter`
  (or `AuthLimiter` / `DeploymentLimiter` as appropriate). CodeQL flags
  unrate-limited routes.
- **TypeScript:** `tsc --noEmit` must pass with zero errors before commit.
- **ESLint:** zero errors before commit.

---

## 🚫 Never Do


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FutureSolutionDev/Deploy-Center-Server](https://github.com/FutureSolutionDev/Deploy-Center-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
