---
trigger: always_on
description: This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.
---

# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Add durable project-specific notes here as they are discovered through real work.

## Running the stack

`docker compose up -d --build` brings up six containers: `webui` (FastAPI on :8000),
`db` (Postgres on :5432), `postgrest` (:3000), `searxng` (:8080), a local `qdrant`
(:6333) and `scheduler` (no ports). The build is slow and the image is large —
`requirements.txt` pulls torch and layoutparser.

`.env` is gitignored and is not in the repo; compose reads it via `env_file:`. It is also
not copied into the image (see the `COPY` lines in `Dockerfile`), so the container's
environment comes entirely from compose. That absence is load-bearing: it is why
`get_settings()`'s `load_dotenv(override=True)` cannot clobber the overrides compose sets.

`scheduler` runs the same image as `webui` with `command: python -m nlp_pillars.scheduler`.
Any *new* service reusing this image must set `healthcheck: disable: true` as it does — the
Dockerfile's `HEALTHCHECK` curls `localhost:8000/health`, which exists only under uvicorn,
so a non-webui service inherits a probe that can never pass and sits permanently
"unhealthy" while working fine.

### Running a second stack alongside an existing one

Container names, host ports and volume names are all fixed, so a plain `up -d` in a second
worktree evicts the first. Use `-p <project>` plus an uncommitted override that renames
`container_name`, every volume `name:`, and the ports. Ports need the `!override` YAML tag —
Compose *appends* list entries by default, so a plain `ports:` in an override republishes
the original host port too and collides anyway.

## The database is self-hosted

Postgres + PostgREST run in this compose file. The hosted Supabase project was reaped and
its host no longer resolves; `SUPABASE_URL` / `SUPABASE_KEY` are left in `.env` only so the
old project stays reachable if it is ever recovered.

Three env vars are overridden on `webui` in `docker-compose.yml`, and all three are load-
bearing — the comments there explain why. The trap: there are **two** database clients and
they resolve the URL differently. `webui/services/postgrest_client.py` reads
`POSTGREST_URL` then `SUPABASE_URL`, but `nlp_pillars/db.py::get_client()` reads
`SUPABASE_URL` **only**. Setting `POSTGREST_URL` alone silently fixes half the app.

`SUPABASE_KEY` is sent as an `Authorization: Bearer` token on every request and cannot be
disabled (`get_client()` raises if it is empty). PostgREST therefore **must** have
`PGRST_JWT_SECRET` set: with no secret, a request carrying a token fails with
`500 PGRST300 "Server lacks JWT secret"` rather than being served anonymously. The secret
and the `web_anon` token in compose are a matched pair — change them together.

Schema is applied automatically on **first** start only, from `docker-entrypoint-initdb.d`
(`db/init/01-roles.sh`, `schema.sql` mounted as `02-schema.sql`, `db/init/03-grants.sql`).
Editing `schema.sql` does nothing to an existing volume; migrate by hand or recreate
`nlp_pg_data`. Never `docker compose down -v` casually — that volume is the real data, and
`qdrant_data` is shared with any other worktree running this compose file.

## Uploaded PDFs are retained, and are the only copy

A file upload stores `papers.url_pdf = file:///app/data/uploads/<hash>.pdf` and the podcast
agent dereferences that path to get the paper body, so the file is real data, not a cache.
It lives in the `nlp_uploads` named volume (mounted at `/app/data`, created and chowned in
the `Dockerfile` so the non-root `appuser` can write into a fresh volume) and
`upload_service.py` deletes it only when the upload never reached the database. Do not
"tidy" that directory and do not move it under `.cache/`. There is deliberately **no**
retention or cleanup policy yet — retained PDFs accumulate at ~1-5 MB/paper.

Papers added by URL keep the http URL in `url_pdf` and are re-downloaded on demand, so
only the file-upload path depends on this.

## Which Qdrant the app talks to

`webui` uses `QDRANT_URL` from `.env`, which points at a managed Qdrant Cloud cluster.
The local `qdrant` service is kept only as an offline option and is **not** what the app
uses — do not re-add a `QDRANT_URL` override to the `webui` service in `docker-compose.yml`.
`depends_on: qdrant` on `webui` is therefore cosmetic and is left in place deliberately.

`nlp_pillars/vectors.py::ensure_collections()` creates the single `nlp_pillars` collection
(1536-dim, cosine) **and** the `pillar_id` keyword payload index. It runs from
`VectorSearchTool.__init__`, so constructing an `Orchestrator` is enough to create both.
The cloud cluster is a small free tier — do not bulk-load it.

That index is not optional. The cloud cluster runs Qdrant **strict mode**
(`unindexed_filtering_retrieve: false`), which answers a filtered query on an unindexed
payload key with `400 Bad Request: Index required but not found`. Every read in
`vectors.py` filters by `pillar_id` — that is the namespace isolation — so without the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabomsambo/nlp-learning-workflow](https://github.com/gabomsambo/nlp-learning-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
