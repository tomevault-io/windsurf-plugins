---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# StudentHub — Telegram Mini App + Web (Turborepo)

Open-source StudentHub platform — Telegram Mini App + browser web (via Telegram Login Widget) — for university students (curriculum charts, course offerings, diff notifications). **Vercel serverless** (Hono on Bun) + **Postgres is the only infra** (no Redis/MinIO/BullMQ). University data lives in a **git registry** (`packages/registry`), not DB.

**Versions:** mini-app `1.0.0-beta.1` (actively versioned); all **other** workspaces pinned to `1.0.0` (static, never bumped). • **Branch:** `main` • **Migrated from** legacy `4.1.9` Supabase — only `azad-malard / computer-engineering` (uni 1/major 1) kept, 291 profiles, 330 noted, 5676 passed, 98 professor votes. Other universities dropped (users go to `/setup`). See Migration notes below.

## Commit Convention (Required)

All commits **MUST** follow Conventional Commits:

```
<type>(<scope>): <short description>
```

Allowed types: `feat`, `fix`, `chore`, `docs`, `refactor`, `perf`, `test`, `style`, `build`, `ci`, `revert`.

- Scope is optional but recommended, e.g. `feat(api): ...`, `fix(mini-app): ...`, `chore(deps): ...`. Use `*` only for cross-cutting changes: `feat(*): ...`.
- Keep `type` and `scope` lowercase; description in imperative mood, no trailing period, max ~72 chars. Good: `feat(course): add prerequisites display` / Bad: `Feat(Course): Added prerequisites.`
- Breaking changes via `feat!:` / `fix!:` or `BREAKING CHANGE:` footer. One logical change per commit.
- Validated against `^(feat|fix|chore|docs|refactor|perf|test|style|build|ci|revert)(\(.+\))?: .+` — non-conforming commits will be rejected in review; squash-merge PRs must be reworded to conform.
- Examples: `feat: add validation for university and major consistency` / `fix(telegram): handle expired file links gracefully` / `chore(deps): bump next to 15.2.3` / `feat(*): migrate shared utils to new structure`

## Commands

```bash
docker compose up -d        # Postgres 17 only (5433)
pnpm install
cp .env.example .env        # DATABASE_URL must match POSTGRES_PORT
pnpm --filter @workspace/db generate
pnpm --filter @workspace/db migrate
pnpm --filter @workspace/db studio

pnpm --filter @workspace/db seed:clear              # drop + migrate
pnpm --filter @workspace/db seed:mock-data          # 700000000+ fake ids
pnpm --filter @workspace/db exec tsx ./scripts/migrate-malard.ts  # legacy → local (read-only Supabase pooler)
pnpm --filter @workspace/registry validate
pnpm --filter @workspace/registry build-index
```

## Architecture Decisions (do not undo)

- **Registry over DB:** Universities, majors, degrees, entry-year dirs (`[1400-1401]`/`1402`/`[1403-1405]`), charts, offerings, professors, archives, groups — all JSON in `packages/registry`, validated by CI (`registry.yml` now just `validate` on every push). Year dir detector accepts both range and single names.
- **DB rows reference registry slugs:** `university_profiles`, noted/passed, votes, uploads store `universitySlug/majorSlug` strings; API validates at read time, dangling slugs surface as “chart moved”.
- **Users = Telegram chat ids** (`bigint` PK). Auth: **Mini App** `tma <initData>` (stateless, auto-upsert, `withUser`); **Web** Telegram Login Widget / OIDC (`POST /auth/telegram/widget` → `id_token` or `hash` verified via `SHA256(bot_token)` + `JWKS https://oauth.telegram.org/.well-known/jwks.json` → `aud=app` JWT 30d, `Bearer` + `x-bypass-maintenance` header). `resolveInitData()` falls back to `window.Telegram.WebApp.initData`, `request.ts` sends `tma` or `Bearer`.
- **Notifications resumable + manual:** `completed_offering_diffs` + `notification_batches/messages` (`PENDING`), admin clicks `send-next` per message; never auto-send from CI.
- **Uploads no object storage:** `POST /me/uploads` streams to `TELEGRAM_UPLOADS_CHAT_ID` via `sendDocument`, only `file_id` stored (`uploads` PENDING). Admin PRs `archives.json`.
- **Maintenance gate:** `app_settings.maintenanceMode` cached 30s. `withUser` runs before `maintenanceGate`; if `SUPERADMIN` + `x-bypass-maintenance:1` → pass, else `503 { maintenance:true, canBypass:true }` for superadmins (client shows “ورود به عنوان سوپرادمین” → `sessionStorage.sh_bypass_maintenance=1` + reload). `request.ts` adds header, `server.ts` CORS allows it.
- **Intro/profile gating:** `AppBootstrap` holds splash 600ms, hydrates `/me`, then `localStorage`+`cloudStorage` intro check (`completed-introduce-v2-1` saved to both), `isProfileComplete` (5 fields) → `/welcome` / `/setup` / `/profile` (+ `rd` deep link). Web unauthenticated → `TelegramLoginWidget` (legacy `telegram-widget.js?22` + new `telegram-login.js?6`).

## Packages

- `packages/db` — Drizzle schemas: `users`, `university_profiles` (`bachelors-degree`, `currentSemesterCode=4051`), `noted_courses/passed_courses/failed_courses`, `professor_votes`, `uploads`, `feedback`, `chart_files`, `app_settings` etc. `isContributor` badge, `banned=false` for all after migration, `5725800953=SUPERADMIN`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taymakz/studenthub](https://github.com/taymakz/studenthub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
