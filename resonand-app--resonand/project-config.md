---
trigger: always_on
description: Instructions for coding agents working in this repository. Read this before touching anything.
---

# AGENTS.md

Instructions for coding agents working in this repository. Read this before touching anything.

## What Resonand is

A self-hosted archive for personal audio recordings: it keeps the original byte-for-byte,
transcribes it through a provider you choose, and lets you search inside every transcript you own.
A single container — Python 3.12 + FastAPI + SQLite/FTS5 on the back, React 19 + TypeScript + Vite
on the front, one in-process job worker. Licence AGPL-3.0.

**Status: `0.1.0` is released**, as `ghcr.io/resonand-app/resonand` for `linux/amd64` and
`linux/arm64`. Somebody is running this, so a change reaches an archive that already exists: a
migration runs on somebody's data, a setting rename breaks somebody's `.env`, and an endpoint that
moves breaks whatever they wrote against it. `CHANGELOG.md` says what the number promises -- a
patch fixes, a minor may break and says how -- and the version is set by `scripts/release.sh`
rather than by hand, because it is written in six places and four of them are generated.
`README.md` is the product statement, `VISION.md` the principles, `ROADMAP.md` what is deliberately
*not* in the first version.

## Repository map

How the tree is organised, not what is in it — a file listing rots, the shape does not.

| Path | How it is organised |
|---|---|
| `backend/` | The `resonand` Python package, a test suite mirroring it, and the whole toolchain description (`pyproject.toml`, `uv.lock`) |
| `frontend/` | The interface (`src/`) beside the vendored design system (`design-system/`), which sits outside `src/` on purpose (`DEC-21`) |
| `docs/` | The committed plans. `docs/internal/` is local-only |
| `deploy/` | What an operator needs: the compose file, `.env.example`, and a README of their own |
| `Dockerfile` | Multi-stage. The bundle and the package come out of one image |
| `.pre-commit-config.yaml`, `.github/workflows/ci.yml` | The quality gate, described once and run in both places |

### `backend/resonand/`

Layered, and the dependency runs one way:
`core` ← `db` / `acl` / `media` / `transcription` ← `api` / `jobs` / `cli`.

- `core/` — primitives with no dependencies of their own: settings (pydantic-settings, every
  variable prefixed `RESONAND_`), ids, time, text, enums.
- `db/` — the engine and its pragmas, the ORM models, Alembic wiring, and one module per
  aggregate. Named repositories, but they are application services: they resolve permissions and
  enforce invariants, not just rows (`REV-S3`).
- `acl/` — one query. Every read and write resolves permissions through it.
- `api/` — composition, dependencies, security, presenters, and `routes/` with one module per
  resource group.
- `media/` — everything that shells out to ffmpeg/ffprobe.
- `transcription/` — the provider boundary: a contract, a registry, and one implementation.
- `jobs/` — the in-process queue and its worker.
- `cli/` — the Typer app `pyproject.toml` installs as `resonand`.
- `migrations/` — the Alembic tree, shipped inside the package so a container migrates itself.
- `archive.py` — export and re-import, and the only module at the package root: it reaches across
  `db` and `media` and answers to the CLI alone, so it belongs to no one layer. Its test mirrors
  it at `tests/test_archive.py`.

`backend/tests/` mirrors this layout, one directory per package.

### `frontend/`

- `design-system/` — the signed-off visual language (`DEC-8`). `tokens/*.css` is the source of
  truth for every value; `components/` is grouped by family, each shipping a `.prompt.md` saying
  when *not* to use it; `index.ts` is the only public entrance; `guidelines/` are standalone
  specimen cards a designer can open with no bundler in the way.
- `src/api/` — the hand-written client. `contract/` beside it holds the committed OpenAPI
  document and the types generated from it; nothing in there is edited by hand, which is why
  ESLint, Prettier and coverage each exclude it with one glob.
- `src/app/` — the spine: routing, session, keyboard commands, URL state. `shell/` is the chrome
  those modules are wired into, reached from nowhere else; `hooks/` the ones any feature may use.
- `src/features/` — one folder per view, holding its data-bound components and view logic. A
  folder directly under `features/` is a route and owns a `*View.tsx`; a folder nested inside one
  is a part of that view rather than a destination — `settings/administration/` is a tab.
- `src/components/` — composites used by more than one feature, plus the pair that is tested as
  one: `FilterBar` and the `BulkBar` that replaces it have to be the same height.
- `src/player/` — the player, which outlives every navigation and so lives in the shell.
- `src/i18n/` — i18next setup, `en/*.json`, and the hooks that hand a component its copy. Every
  user-visible string is here.
- `src/test/` — the tests whose subject is the repository itself, the msw handlers in `api/`, and
  in `support/` the modules that exist only to be imported by a test.
- **Tests under `src/` live in a `tests/` folder inside the folder they cover**, so a listing
  shows the thing and not the thing plus its tests. They reach their subject with `../`, which

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [resonand-app/resonand](https://github.com/resonand-app/resonand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
