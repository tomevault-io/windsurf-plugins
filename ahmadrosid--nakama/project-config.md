---
trigger: always_on
description: Agent platform built to work with your team — not replace them. Multi-tenant monorepo; orgs are flat tenants, each profile has a **soul** (identity, style, instructions, memory).
---

# nakama — Agent Context

Agent platform built to work with your team — not replace them. Multi-tenant monorepo; orgs are flat tenants, each profile has a **soul** (identity, style, instructions, memory).

## Dev

- Bun 1.3+: `bun install`, `bun run`, `bun test`
- Servers: `bun run dev:server` | `dev:web` | `dev:cli`
- Layout: `apps/{server,web,cli}`, channel workers in `apps/platform/{telegram,whatsapp,discord,automation}`

## LLM cassette tests (MSW)

For live provider tests: record one real HTTP call, commit the cassette, replay offline thereafter. Helper: `apps/server/src/testing/llm-msw-cassette.ts` (`withMswCassette`). Cassettes live in `apps/server/src/testing/cassettes/`. Name live tests `*.llm.test.ts`.

```bash
bun test path/to/foo.llm.test.ts                 # replay (default when cassette exists)
LLM_VCR_MODE=record bun test path/to/foo.llm.test.ts  # re-record (needs provider API key)
```

## GitHub

Use `gh` for issues, PRs, checks, reviews, releases, and any GitHub URL. Do not use the API, browser, or scraping.

## Browser automation

Use `agent-browser` cli to do browser automation, screenshot etc. Run the docker first when you need to debug with first installation, for just quick test or screenshot use local dev server that already running.

## Docker

One container: API + web + platform workers. Data at `/nakama/data` (`NAKAMA_CONFIG_DIR`). Dashboard: http://localhost:4310

```bash
# Prebuilt
docker pull ghcr.io/ahmadrosid/nakama:latest
docker run -d -p 4310:4310 -v nakama-data:/nakama/data --name nakama ghcr.io/ahmadrosid/nakama:latest

# Build from source (uses buildx; default linux/amd64 -t nakama)
./scripts/docker-build.sh
docker run -d -p 4310:4310 -v nakama-data:/nakama/data --name nakama nakama

# Fresh start (removes container, volume, image)
./scripts/docker-reset.sh
./scripts/docker-build.sh
docker run -d -p 4310:4310 -v nakama-data:/nakama/data --name nakama nakama
```

## Multi-tenancy

Orgs isolate profiles, sessions, automations, tasks, tools, MCP, skills, usage (`org_id` — see `packages/db/sql/schema.sql`, `migrateTenantOrgScope`).

| Role | Can |
|---|---|
| Platform admin | Orgs (`/v1/platform/orgs`), profiles/tools/MCP/skills |
| Org admin | Members/invites (`/v1/orgs/{orgId}/members`) |
| Org member | Chat, agents, automations/tasks |
| Org viewer | Read chat only — no agent invoke / mutations |

**Org context:** every authed call except `/v1/auth/*` and `/v1/platform/*` needs `X-Org-Id` (`@nakama/client`) or `active_org_id` cookie (`POST /v1/auth/active-org`). Middleware: `org-middleware.ts`; guards: `org-guards.ts`.

**Onboard:** setup → `POST /v1/auth/setup`; more orgs → platform admin; invite → `/v1/orgs/{orgId}/invites` + `POST /v1/auth/accept-invite`; switch → `OrgSwitcher.tsx` / `client.setActiveOrg()`.

| Change | Where |
|---|---|
| Org CRUD / invites / members | `apps/server/src/services/org-service.ts` |
| Platform org routes | `apps/server/src/http/routes/platform-orgs.ts` |
| Member routes | `…/routes/org-members.ts` |
| Auth / active-org | `…/routes/auth.ts` |
| DB types / SQLite | `packages/db/src/{types.ts,adapters/sqlite.ts}` |
| Contracts | `packages/core/src/contract.ts` |
| Client `X-Org-Id` | `packages/client/src/client.ts` |
| Web auth / switcher | `apps/web/src/context/auth-context.tsx`, `OrgSwitcher.tsx` |

## System prompt

Merged in `agent-service` `resolveProfileSystemPrompt` → `generateReply` (`provider.generateChat` / `streamChat`):

| Change | File | Fn |
|---|---|---|
| Chat structure (USER.md, tools, timezone, channels) | `packages/agent/src/chat-prompt.ts` | `buildChatSystemPrompt` |
| Soul content | `packages/core/src/soul/compose.ts` | `composeSoulSystemPrompt` |
| Skills catalog / matched / agent-browser | `packages/core/src/skills/compose.ts` | `composeSkillsCatalog`, `composeMatchedSkillsPrompt`, `composeAgentBrowserCapabilityPrompt` |
| Per-turn context (date, etc.) | `packages/agent/src/chat.ts` | `generateReply` |

## Soul (`packages/core/src/soul/`)

Path: `~/.nakama/orgs/{orgId}/profiles/{profileId}/` (`getProfileSoulDir`). Load: `loadSoulStack()`; inject: `composeSoulSystemPrompt()`.

| File | Role |
|---|---|
| `SOUL.md` | Identity |
| `STYLE.md` | Voice |
| `INSTRUCTIONS.md` | Operating rules |
| `MEMORY.md` | Cross-session facts |

## Tools (`packages/core/src/tools/`)

| Tool / skill | Notes |
|---|---|
| `update-profile-memory` / `archive-profile-memory` | MEMORY.md ↔ memory-archive/ |
| `save-artifact` | Persist under `artifacts/` |
| `knowledge_base_search` / `web_search` / `email` | KB, web, mailbox |
| `search_files` / `ripgrep` | File/content search |
| `bash` | Super Bot — profile workspace shell |
| `sub_agent` | Opt-in same-profile delegate (not repo coding) |
| `coding-agent` | Codex / Claude Code / OpenCode via `bash` |
| `agent-browser` | Opt-in browser CLI; needs host install — `docs/website/agent-browser.md` |
| `create-profile` | Super Bot only, confirm-first — `apps/server/src/tools/super-bot-tools.ts` |
| Composio | Org toolkits + per-user OAuth — `docs/website/composio.md` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahmadrosid/nakama](https://github.com/ahmadrosid/nakama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
