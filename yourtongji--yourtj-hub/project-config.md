---
trigger: always_on
description: Operating guide for anyone (human or AI agent) changing this repository.
---

# AGENTS.md — yourtj-hub

Operating guide for anyone (human or AI agent) changing this repository.

Before changing anything, read this file, [`docs/README.md`](docs/README.md),
[`docs/development/README.md`](docs/development/README.md), and the product/architecture/operations
documents directly affected by the request. Use the repository `$yourtj-development` skill for
implementation, testing, review, CI, or PR work.

---

## 1. What this is

yourtj-hub is the monorepo for the Tongji university campus forum platform (brand: yourtj, distinct from
the archived YourTJ-Platform). The forum is the core product — a **direct modification of the upstream
GooseForum, keeping the single-binary deployment**. Unified auth (built-in OIDC Provider), search (Meilisearch), and
cross-platform points (credit, `Planned`) are shared infrastructure subdomains. Database, search, and structure may all
be changed, but the "Go + Vue in one binary, frontend go:embed into the binary" deployment shape is kept.

- Forum: **Go 1.26 + Gin + Vue 3 + Tailwind**, at `apps/gooseforum` (fork of upstream; module path
  `github.com/YourTongji/YourTJ-Hub/apps/gooseforum`, diverged from upstream's `github.com/leancodebox/GooseForum`).
- Backend layers (upstream structure): `app/bundles` (utilities) → `app/models` (GORM models) →
  `app/service` (business) → `app/http/controllers/{api,forum}` (JSON API + GoHTML three-mode rendering).
- Frontend: `apps/gooseforum/resource` (Vue 3 + Vite, site/admin dual entry), built output
  `resource/static/dist` go:embed; GoHTML templates in `resource/templates` keep server-side rendering (three-mode).
- Database: **PostgreSQL is the default deployment database** (`deploy/config.toml.example`
  `[db.default] connection = "postgres"`); SQLite stays the local development/test default
  (`apps/gooseforum/config.toml`, in-memory tests); the file db (`[db.file]`) is fixed SQLite.
  MySQL is **not supported**.
- Search: **Meilisearch** (`config.toml [meilisearch]`, optional); aggregate search (topics/users/
  categories, pinyin/initials) landed (issue #22); event-driven index sync, rebuildable projection.
- Mobile: **Flutter** (`apps/mobile`, melos workspace, Riverpod, **Partial**).
- Auth: GitHub OAuth (goth) + **built-in OIDC Provider** (`/api/oauth`, authorization code + PKCE S256,
  RS256 id_token, opaque access tokens, numeric `sub` = users.id); TOTP 2FA and session management
  (`jti` + `user_sessions`) in place.
- Contract: **Partial** — `packages/api-contract/openapi.yaml` is the controlled contract center for
  password login, login public-key retrieval, TOTP login verification and account management, logout,
  mobile OIDC exchange, session management (list/revoke/revoke-all), topic writing, forum core
  interactions (post create/update/delete/window/revisions, topic status/delete/like/bookmark/watch,
  post like/bookmark, follow-user, report), user account and identity (captcha, user-card,
  profile/email/username/avatar/badge settings, upload-avatar, change-password, OAuth
  bindings/unbind), notifications/unread/chat, forum moderation workbench, the admin console
  (`/api/admin/*`: user/role/category/moderator management, topic/post moderation, agent
  administration, operation records, traffic overview, page settings, site settings, data
  import/export), account
  registration/password recovery (`/api/register`, `/api/forgot-password`, `/api/reset-password`),
  the six-operation Agent forum API, course catalog reads + review write/moderation, the Wiki domain
  (public tree/namespaces/home + admin read-only tree + sync/status|sync|sync/runs +
  sync/webhook-secret + asset CDN + webhook; the legacy in-forum write/revision/rollback/diff/editor
  and namespace-CRUD endpoints were retired with the GitHub-SSoT model),
  and the PK scheduler (14 ops:
  courses-by-major/optional-types/courses-by-nature/course-details/course-search/courses-by-time/
  latest-update/course-info-sync/course-review-brief),
  the user content lifecycle (my-content/deleted-content lists, content-restore/batch-delete/
  purge/event, account-close), aggregate search (`/api/forum/search`) and public
  site statistics,
  the sticker library (`GET /api/forum/stickers` public enabled list + `/api/admin/`
  `stickers`/`sticker-save`/`sticker-delete`/`sticker-import` admin CRUD and zip pack
  import; token `[:sticker:name:]` expanded server-side at render for posts/replies and
  client-side in chat bubbles (web segmented renderer) and mobile (markdown pre-expansion +
  inline image spans) via the public list API, MADR 0030),
  with lint/bundle, generated TypeScript types, fixtures, and route-level HTTP tests;
  paths are split per domain under `paths/`. Route coverage (issue #277) is complete:
  `route-coverage.json` knownUncovered is empty — every non-excluded `/api` route has a
  contract operation; new routes must be added to the contract or the exclusion list.
- Points: forum-local ledger mechanics are `Current`, durable reward delivery is `Partial`;
  cross-platform credit settlement is `Planned`. See `docs/product/credit-and-escrow.md`.

## 2. Repository layout & boundary rules

```
apps/
  gooseforum/  The forum itself (upstream fork; module path github.com/YourTongji/YourTJ-Hub/apps/gooseforum)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YourTongji/YourTJ-Hub](https://github.com/YourTongji/YourTJ-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
