---
trigger: always_on
description: A private, AI-curated "Picked for You" row for every user on a Plex server. One Docker container:
---

# Shortlist

A private, AI-curated "Picked for You" row for every user on a Plex server. One Docker container:
FastAPI backend + React SPA + SQLite, with a pure-Python engine (per-user watched set read from the
PMS via each share's server token → TMDB similar-titles → LLM curate/explain → per-user Plex
collection + label-restriction privacy).

**Status: 1.0.** In production on the maintainer's server: the FastAPI server runs the engine on
its own nightly schedule (APScheduler), with the React SPA and Docker
packaging. Read these
before any feature work:

- [.claude/docs/shortlist-design.md](docs/shortlist-design.md) — product/UX design (wizard, screens, engine, privacy system)
- [.claude/docs/shortlist-architecture.md](docs/shortlist-architecture.md) — repo layout, DB schema, API surface, testing, CI, phases
- [.claude/docs/jobs-and-runs-design.md](docs/jobs-and-runs-design.md) — runs vs. jobs, the durable
  queue, the delivery ledger, and §12's mutation audit: every state change and whether it actually
  reaches Plex. **Read §12 before touching any handler that changes who can see what** — it is the
  register of a bug class this codebase has had 15 instances of.

Personal deployment details (Steve's servers) live in `CLAUDE.local.md` — gitignored, never commit
it, and never leak environment-specific hostnames/IPs/paths into the public repo or docs.

## Commands

```bash
# Backend
pip install -r requirements.lock && pip install -e ".[dev]"   # lock first: same versions the image ships
pytest                       # unit+integration, parallel, coverage (target ≥80%)
pytest -m e2e                # Playwright vs an in-process app (uvicorn + built SPA) + tests/fakes/fake_plex.py
ruff check . --fix && ruff format .

# Regenerate requirements.lock — REQUIRED whenever pyproject's dependencies change.
# Resolves for the image's interpreter/OS, not your laptop's, so it is safe to run from macOS.
# CI's lint job fails if pyproject declares a dependency the lock doesn't carry.
uv pip compile pyproject.toml \
  --extra anthropic --extra openai --extra google --extra posters \
  --python-version 3.12 --python-platform linux \
  --output-file requirements.lock

# Frontend
pnpm -C web install
pnpm -C web dev              # Vite dev server on :5173; proxies /api to :5959 by default
SHORTLIST_API_PROXY=http://localhost:5960 pnpm -C web dev   # ...or to a scripts/devrun.sh backend
pnpm -C web test             # vitest
pnpm -C web build

# Run (dev) — throwaway config, safe mode, port 5960. Refuses to start if a running container
# already mounts that config dir (two schedulers on one DB duplicates real Plex writes).
bash scripts/devrun.sh
# Overridable: PORT=... SHORTLIST_CONFIG=... SHORTLIST_DRY_RUN=0 bash scripts/devrun.sh
# Under the hood — module-level `app` only exists when SHORTLIST_CONFIG is set:
SHORTLIST_CONFIG=./devconfig uvicorn --factory shortlist.server.main:create_app --reload --port 5960

# Docker
docker build -t shortlist:dev .   # multi-stage: node web build → python runtime
```

## Row privacy (leak-safe ordering)

Rows are made private by share-filter excludes: each account's `filterMovies`/`filterTelevision`
gets `label!=shortlist_<otheruser>` for every row that isn't theirs. The ordering is what keeps it
leak-safe — rows are delivered UNPROMOTED, all filters merged, and only then promoted, so a new row
is never promoted before the exclusions that hide it exist. Unpromoted is not invisible: a row is listed
in the library's Collections tab until its exclude is on an account's filter, so a person's FIRST row is
excluded as soon as it is written, before any other row is — unless plex.tv fails, when the end-of-run
merge does it (plex-safety rule 1).

The old automatic Privacy Check + write gate that _verified_ this before each write was **removed at
the owner's request** (2026-07-16). Writes are no longer gated on a recorded check; the hiding still
happens, but nothing verifies it after the fact — see `.claude/rules/plex-safety.md`.

## Architecture (the one contract that matters)

```
shortlist/engine/    pure library — NO imports from shortlist/server/; takes config dataclasses + clients, returns reports
shortlist/server/    FastAPI + SQLAlchemy/Alembic + APScheduler + SSE; the thin adapter over the engine
web/              React + Vite + TS + Tailwind + shadcn/ui; API types generated from OpenAPI
tests/fakes/      fake_plex.py — stub PMS + plex.tv; e2e runs the full wizard with no real server
```

See shortlist-architecture.md §2–§4 for the full tree, DB schema, and API surface.

## Working economically

Long sessions are the single biggest cost: every turn re-sends the whole conversation. So:

- **Say when a `/clear` is due.** When the next request is a genuinely new task — a different
  feature, a different bug, a different area — say so in one line before starting, and let the owner
  decide. Don't nag mid-task; the cost of losing context you still need is higher than the tokens.
- **Write the test first, then run only that test** (owner decision 2026-09-12). What made testing
  feel like it set the pace of development was running the WHOLE suite mid-edit. Writing the test
  before the code fixes that: it gives you one file to run, and one file is ~3.5s.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevezau/shortlist](https://github.com/stevezau/shortlist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
