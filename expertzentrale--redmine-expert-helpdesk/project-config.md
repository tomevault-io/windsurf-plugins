---
trigger: always_on
description: These instructions apply to the whole repository. They mirror `CLAUDE.md` (the fuller
---

# GitHub Copilot instructions — redmine_expert_helpdesk

These instructions apply to the whole repository. They mirror `CLAUDE.md` (the fuller
companion doc — read it for deeper architecture notes); keep the two in sync when either changes.

## What this is

A **Redmine plugin** (`redmine_expert_helpdesk`) that turns Microsoft 365 **or IMAP** mailboxes into a
helpdesk: incoming mail becomes tickets (or journal replies), agents reply to customers from
the ticket page, and contacts/SLA/phishing detection layer on top. Requires Redmine 5.0+.

The plugin is developed **inside the parent `redmine-expert` deployment repo** (two levels up,
`../../`), which packages plugins, themes and Docker/Kubernetes config. **This repo is NOT a
Redmine checkout** — there is no Gemfile, `rake`, or `config/database.yml` at `../../`. Base
Redmine is supplied by a Docker image; the app only exists inside the running container
(`/usr/src/redmine`), so host-level `bundle exec rake` does not work.

## Hard rules (must follow)

- **Never use `docker exec`** to run commands inside a running container (hard project rule).
  You **may** use `docker-compose` to manage the stack.
- Because the plugin is **`COPY`'d into the image** (see `Dockerfile.dev`), not volume-mounted,
  a plain `restart` does **not** pick up edited source or new routes. Rebuild with
  `docker-compose -f docker-compose.yml up --build` (run from the parent repo root `../../`).
- **Every change must update `CHANGELOG.md` (EN, authoritative)** and its German mirror
  **`CHANGELOG.de.md`**; for user-facing changes also **`README.md` (EN) and `README.de.md` (DE)**.
  Keep each EN/DE pair in sync. GitHub release notes are generated from `CHANGELOG.md`.
- **Language policy:** write **code comments in English** (legacy German comments may remain until
  that code is next touched). **i18n stays bilingual** (`config/locales/en.yml` + `de.yml`) and the
  plugin UI is German.
- **Never edit a shipped migration.** When changing schema, add the next sequential number in
  `db/migrate/` (`001_...` onward).
- Don't hard-code secrets/API keys. Local dev credentials live in `.dev.env` (git-ignored).
- **Don't develop features on `main`.** Every feature, bugfix or schema change goes on a
  `type/short-desc` branch and reaches `main` via a squash-merged PR (see *Git workflow* below).

## Development workflow (Docker)

Run everything from the **parent repo root** (`../../`).

```bash
# Start / rebuild the local stack (MariaDB + Redmine + phpMyAdmin) — Redmine on :3000
docker-compose -f docker-compose.yml up --build
```

- **Migrations run automatically** on build via `REDMINE_PLUGINS_MIGRATE=1` (see
  `docker-compose.yml`). Manual equivalent (in the container/build context, not the host):
  `bundle exec rake redmine:plugins:migrate NAME=redmine_expert_helpdesk RAILS_ENV=production`.
- A DB dump in `../../redmine_db_dump/` is auto-imported on first MariaDB start.
- To inspect Redmine core behavior, use the Redmine source on GitHub — do **not** `docker exec`.

## Tests

MiniTest (`ActiveSupport::TestCase`); `test/test_helper.rb` loads Redmine's own test helper, so
tests **require a Redmine environment** (the container / a Redmine checkout — they cannot run
standalone here). Inside a Redmine environment:

```bash
bundle exec rake redmine:plugins:test NAME=redmine_expert_helpdesk RAILS_ENV=test
bundle exec ruby -Itest plugins/redmine_expert_helpdesk/test/unit/sla_test.rb
```

The REST API (see `API.md`) can be smoke-tested live against the local stack over HTTP from the
host — source `.dev.env` for `REDMINE_URL` + `REDMINE_API_KEY`, then `curl` with an
`X-Redmine-API-Key` header. Use a `project_id` with the Helpdesk module enabled.

## Git workflow (feature branches)

`main` is the protected integration branch — **release tags are cut only from merged `main`**.
Do not build features directly on it.

- **Branch per unit of work**, named `type/short-desc` where `type` matches the Conventional-Commit
  style used in the CHANGELOG: `feat/…`, `fix/…`, `chore/…`, `docs/…`, `refactor/…`, `test/…`
  (e.g. `feat/sla-priority-overrides`, `fix/graph-token-refresh`). Branch off the latest `main`.
- **Commit messages** follow the same Conventional-Commit prefixes; keep commits focused and
  include the CHANGELOG/README updates the change requires (see the CHANGELOG hard rule above).
- **Open a PR** into `main`. CI (`ci.yml`, `docker-image.yml`) runs on PRs and **must pass** before
  merge. Prefer a short self-review / description linking the change to its CHANGELOG entry.
- **Squash-merge** the PR — one commit per feature keeps `main` history linear and readable. Delete
  the branch after merge.
- **Exception — trivial fixes may commit straight to `main`**: docs/typo/CHANGELOG-only tweaks that
  touch no code and no schema. Anything touching Ruby/JS, migrations, i18n or behavior needs a
  branch + PR.
- **Never rewrite published history** (`main`, or any branch someone else has pulled). Rebase only
  your own un-pushed local commits.

## Releases (tag-driven)

GitHub Releases are produced only by pushing a semver tag, never on normal pushes/PRs. The plugin
version is the **single source of truth in `init.rb`** (`version '...'`): bump + commit it, then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [expertZentrale/redmine_expert_helpdesk](https://github.com/expertZentrale/redmine_expert_helpdesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
