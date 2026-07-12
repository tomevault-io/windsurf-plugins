---
trigger: always_on
description: Use **`python3`**, not `python`, in all documented commands and examples — macOS
---

# OpenCheck — development notes for Claude

## Local commands (macOS)

Use **`python3`**, not `python`, in all documented commands and examples — macOS
ships Python 3 as `python3` and has no bare `python` on the PATH (`python …`
fails with `command not found`). The same applies to any one-off scripts and the
test suite below.

## After every commit: post the local run commands

After making **any** git commit during a session, post (in the chat) the commands
the user needs to bring the stack up locally on the branch just committed to, so
they can test immediately. The workspace is mounted from the user's disk, so the
commits already exist locally — the user **checks out** the branch, they don't
fetch/pull from origin.

Template (fill in `<branch>`):

```
cd ~/code/opencheck
rm -f .git/*.lock 2>/dev/null            # clear any leftover sandbox lock files
git checkout <branch>

# Backend (one terminal):
cd backend && uv sync && uv run uvicorn opencheck.app:app --reload --port 8000

# Frontend (another terminal):
cd frontend && npm install && npm run dev
```

Notes to add when relevant: uvicorn `--reload` picks up backend changes
automatically, but the Vite dev server must be **restarted** to pick up new files
or `vite.config.ts` / `.env.local` changes; `.env.local` already proxies the API
to `http://127.0.0.1:8000`; `uv sync` / `npm install` are only needed when
dependencies changed but are harmless to run otherwise.

---

## Architecture overview

- **Backend**: FastAPI, split into `backend/opencheck/routers/` (health, search, lookup, export).
- **Frontend**: React + Tailwind, split into `frontend/src/components/` (icons, risk, export, cdd).
- **Sources**: each adapter lives in `backend/opencheck/sources/<name>.py`, registered in `sources/__init__.py`.
- **BODS mapping**: each adapter has a corresponding `map_<name>()` function in `bods/mapper.py`, exported from `bods/__init__.py`.

---

## Open Knowledge Format (OKF) bundle — `okf/`

OpenCheck ships an **[OKF v0.1](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
knowledge bundle** at `okf/` — a directory of markdown files with YAML
frontmatter that lets humans and AI agents understand the project, its data
sources, the BODS/LEI standards, and the API. OKF is "metadata as code": every
concept has a required `type` field, cross-links are plain markdown links, and
`index.md` / `log.md` are reserved filenames (see the spec §3–§9).

Structure: `overview.md`, `architecture.md`, `glossary.md` (project);
`standards/` (BODS v0.4, LEI/GLEIF anchoring); `api/` (one concept per
endpoint); `sources/` (one **Data Source** concept per registered adapter);
`licensing/matrix.md`.

**Two halves:**

- **Hand-authored** narrative concepts (project / standards / api). Edit these by
  hand.
- **Auto-generated** from the live registry: `sources/*.md`, `sources/index.md`,
  `licensing/matrix.md`, `licensing/index.md`. **Do not hand-edit these** — they
  are produced by the generator below and pull `SourceInfo` + `licensing.classify`.

**Tooling (in `backend/scripts/`):**

- `generate_okf.py` — the "enrichment agent". Regenerates the auto concepts from
  the registry. `--check` validates OKF conformance **and** that the generated
  concepts are in sync with the registry (timestamp lines are ignored in the
  drift comparison). Run it (without `--check`) and commit after adding/changing
  a source.
- `generate_okf_viz.py` — renders the whole bundle to a self-contained
  `okf/viz.html` (Cytoscape graph + rendered markdown; CDN-loaded, no backend).
  Regenerate after editing concepts.

**CI:** `.github/workflows/vendored-enum-drift.yml` has an `okf` job that installs
the backend and runs `generate_okf.py --check`, so a stale bundle (e.g. a new
source not regenerated) fails the build — alongside the vendored-enum drift jobs.

---

## Phase 8 — Licensing & AuraDB deferral (recorded 2026-06-07)

### Demo data licences

The `data/demo/` graph is assembled from two freely-shareable published
BODS v0.4 datasets. The combined graph is freely usable in talks,
blog posts, and derivative works under the most restrictive of the two
licences, OGL v3.0:

| Dataset | Licence |
|---|---|
| UK PSC (Companies House via Open Ownership) | OGL v3.0 |
| GLEIF L1 + L2 (GLEIF via Open Ownership) | CC0 1.0 |

Both licences are permissive and compatible. OGL v3.0 requires
attribution; CC0 does not. Pipeline code
(`bods-uk-psc-pipeline`, `bods-gleif-pipeline`) is AGPL-3.0 but is
**not** included in OpenCheck — OpenCheck only reads their published
BODS output. No AGPL obligations apply to OpenCheck.

Full attribution wording and source URLs: `data/demo/LICENCES.md`.

### AuraDB / hosted Neo4j — explicitly parked

**Decision (2026-06-07):** Do **not** move to a hosted Neo4j AuraDB
instance or adopt any embedded graph DB (Kuzu, Memgraph, MemGQL) as a
dependency of OpenCheck's runtime at this time.

**Rationale:** The demo use-case (curated 9-entity set, one-off
build, slides + local Neo4j Docker) is fully served by the current
stack: SQLite extraction → BODS JSON-Lines → `bods-neo4j` CSV → local
Neo4j. Adding a hosted graph DB introduces cost, network dependency,
and operational complexity before any evidence that DuckDB + the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StephenAbbott/opencheck](https://github.com/StephenAbbott/opencheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
