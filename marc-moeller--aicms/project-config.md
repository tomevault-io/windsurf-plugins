---
trigger: always_on
description: This is the one doc to read before changing code in this repo. It distills the
---

# AGENTS.md — canonical engineering reference

This is the one doc to read before changing code in this repo. It distills the
architecture, the data model, and the sharp edges that have bitten past sessions.
The `README.md` keeps the run/commands/env quickstart and points here.

**AI First CMS** — a conversational, AI-native CMS. Users manage websites through
a chat agent that can read/write pages, run tools, and publish sites, instead of a
traditional admin UI.

---

## Tech stack

- **Monorepo**: Turborepo + npm workspaces. Internal packages use `@ai-first-cms-mvp/*` (`workspace:*`).
- **Frontend** (`apps/web`): Next.js, React 19, TailwindCSS 4, shadcn/ui — port **3051**.
- **API** (`apps/server`): Hono + ORPC (type-safe RPC) — port **3052**, RPC at `/rpc`.
- **Public site** (`apps/public-site`): Astro — renders the published customer sites (port 4000).
- **DB**: PostgreSQL (Docker) via Drizzle ORM. **Two logical databases** — see below.
- **Agent LLM**: OpenAI-compatible provider configured through server environment variables.

## Repo map

```
apps/
├── web/          # Next.js admin dashboard (3051)
├── server/       # Hono + ORPC handler entrypoint (3052) — apps/server/src/index.ts
└── public-site/  # Astro renderer for published sites (blocks + raw-HTML)
packages/
├── api/          # ORPC routers (src/routers/*.ts) + services (llm, mcp, authz, security-events)
│                 #   src/index.ts = the procedure ladder (auth middleware)
├── db/           # Drizzle clients + schema. src/index.ts exports db/dbContent/dbControl
│                 #   src/schema/{content,control,access-tokens}.ts
├── executor/     # Sandboxed side-effect runner (validation, egress allowlist, concurrency)
├── types/        # Shared types incl. src/component-registry.ts (COMPONENT_REGISTRY)
└── env/          # @t3-oss/env-nextjs validation — src/server.ts, src/web.ts
```

## Per-site overlays (`sites/`)

Client/site-specific material (theme, imported assets, seed/content scripts,
docs, and reference files) lives in a separate **private** repo cloned into
`sites/<name>/`, which is gitignored. The CMS repo itself must never contain
client files.

Use `node scripts/site-link.mjs <name>` from the repo root to symlink private
overlay assets into the paths the build expects. Each overlay provides a
`site-link.json` array of `{ "source": "...", "target": "..." }` mappings:
`source` is relative to `sites/<name>/`; `target` is relative to this repo root.
Use `--copy` instead of symlinks for environments that need a copy fallback, and
`--unlink` to remove only links created by the same mapping.

---

## The two-database rule (read this first)

There are **two physical Postgres databases** on one instance, each with its own
role + connection string. They are exported from `packages/db/src/index.ts`:

| Client | Database | Holds |
|--------|----------|-------|
| `dbControl` | `cms_control` | **Crown jewels** — identity, credentials, authorization, audit. |
| `dbContent` (aliased as **`db`**) | `cms_content` | Site-scoped content — everything keyed by a site. |

`db` is a back-compat alias for `dbContent` (the bulk of the app is content-side).
**Control-table callers MUST import `dbControl` explicitly.**

- **Control tables** (`packages/db/src/schema/control.ts`): `organizations`, `users`,
  `orgMembers`, `sessions`, `apiTokens`, `humanActions`, `webauthnCredentials`,
  `webauthnChallenges`, `securityEvents`, `capabilityNonces`, `actionPlans`,
  `actionJobs`, `killSwitches`, `auditAnchors`.
- **Content tables** (`packages/db/src/schema/content.ts`): `sites`, `pages`, `people`,
  `intents`, `pagePeople`, `pageHierarchy`, `pageLinks`, `forms`, `formSubmissions`,
  `jobs`, `pageRevisions`, `siteSnapshots`, `assets`, `pageVectors`,
  `agentConversations`, `agentMessages`, `siteConfig`, `designTokens`, `blockStyles`,
  `stylePresets`, `componentStyles`, `themes`, `siteThemes`, `themeChanges`,
  `contentTypes`, `contentEntries`, `deployments`, `siteHosting`, `redirects`.

**Client-selection constraint:** both clients are constructed with the *full* combined
schema object (only so `.query.X` typings resolve). So `db.select().from(humanActions)`
(a control table on the content client) **compiles fine and fails at runtime** — the
role simply can't see that table. The compiler will not save you today.
→ Always match the client to the table. Cross-DB foreign keys do **not** exist:
`sites.organization_id` and every `*.site_id`/`user_id` on the "other" side is a plain
`text` logical reference, validated at the app tenant guard — never `.references()`.

---

## Auth / procedure ladder

Defined in `packages/api/src/index.ts`. Pick the **most restrictive** one that still
works — never hand a broad procedure a job a scoped one can do.

| Procedure | Guarantees | Use for |
|-----------|-----------|---------|
| `publicProcedure` | none | truly public: `auth.*`, passkey ceremonies, `healthCheck`, `form.submit`. |
| `authedProcedure` | valid session | account-level reads not tied to one site. |
| `siteProcedure` | `siteId` present in context | rare; prefer `authedSiteProcedure`. |
| `authedSiteProcedure` | session **AND** `siteId` **AND** tenant membership | **default for all site-scoped work.** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Marc-Moeller/aicms](https://github.com/Marc-Moeller/aicms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
