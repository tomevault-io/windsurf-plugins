---
trigger: always_on
description: Read `PLAN.md` first. It is the agreed scope and roadmap. If anything here
---

# Instructions for Claude (or any agent) working on this repo

Read `PLAN.md` first. It is the agreed scope and roadmap. If anything here
contradicts `PLAN.md`, `PLAN.md` wins.

## What this project is trying to solve

Two specific problems in the Python documentation ecosystem drive all design
decisions here. Understand these before touching anything.

**Problem 1 — Sphinx couples building and rendering.**
In Sphinx, parsing docstrings and rendering HTML happen in the same step. To
update a template (e.g. for accessibility), you must rebuild every project
from source. Papyri solves this by splitting the pipeline:

1. `papyri gen` — run by the library maintainer. Produces a self-contained
   *DocBundle* (the IR) from the project source.
2. Rendering — a separate, stateless step that consumes the IR. Updating the
   renderer never touches the original source.

**Problem 2 — Documentation is fragmented across domains.**
Every library lives on its own subdomain with no shared cross-linking.
Papyri's model: maintainers publish DocBundles; a single rendering service
ingests many bundles and serves them from one place with real cross-package
links (conda-forge model).

The **local viewer** (`viewer/`) is the current reference implementation of
the rendering side. It is used for development and debugging, and is being
designed with the centralized service in mind. The hosted service runs as a
long-running Node.js server on a VPS. The storage layer is kept behind
abstractions (`BlobStore` / `GraphDb` / `RawStore`) so the backend can be
swapped later, but only the filesystem + SQLite implementations exist. An
earlier Cloudflare Workers (R2 + D1) target was abandoned because ingest
latency on it was far too high.

## Repo purpose, short version

- **`papyri gen`**: run per project, by each library maintainer in their own
  CI or build environment. Produces a self-contained DocBundle directory under
  `~/.papyri/data/<pkg>_<ver>/`.
- **`papyri pack`**: packs a DocBundle directory into a `.papyri` artifact
  (gzip-compressed CBOR). The artifact is the canonical shipping unit.
  `papyri unpack` is the inverse — it explodes a `.papyri` artifact back into
  a JSON DocBundle directory for inspection.
- **`papyri upload`**: ships a `.papyri` file, a `.zip` containing one, or a
  DocBundle directory to a viewer instance whose `/api/bundle` endpoint (HTTP
  `PUT`) runs the TypeScript ingest pipeline server-side to wire bundles into
  the cross-linked graph. Auth: `$PAPYRI_UPLOAD_TOKEN` / `--token`; endpoint:
  `$PAPYRI_UPLOAD_URL` / `--url` (default `http://localhost:4321/api/bundle`).
- **`ingest/`**: TypeScript `papyri-ingest` package — the canonical
  ingestion engine, invoked by the viewer's upload endpoint. There is no
  `papyri ingest` Python CLI; do not add one.
- **`viewer/`**: TypeScript web renderer (Astro + React islands). Runs as a
  long-running Node.js server (`@astrojs/node`, `output: "server"`) for both
  local dev (`pnpm dev`) and the hosted VPS deployment (`pnpm build` + `pnpm
  serve`). When building the viewer, think about what the hosted service will
  need.
- There is no Python-side rendering. Do not add any.

## Audience

- **Right now**: contributors to papyri itself.
- **Eventually**: Python library maintainers who publish DocBundles to a
  central service.

When writing docs, code, or CLI help text, speak to contributors first.
Don't design features for the hosted service yet — design so that a hosted
service *could* be built later without a breaking change to the IR.

## Ground rules for changes

0. **Pre-production: prefer deleting dead code over keeping it.** Nothing here
   is shipped to real users yet, there are no published bundles or external
   consumers to keep compatible, and we rebuild everything from the raw
   archives when the IR changes (see "Storage invariant" in `PLAN.md`). So when
   a change makes code, a CBOR tag, a schema entry, a render branch, or a CSS
   block unreachable, **delete it** rather than leaving it for
   backwards-compatibility. Don't add compat shims, legacy-format readers, or
   "just in case" fallbacks for old data — there is no old data that matters.
1. **Stay inside scope.** Before adding or fixing anything, check `PLAN.md`.
   If a task is not in the open work or follow-ups, stop and ask the user.
2. **Small focused PRs.** One logical change per commit.
3. **Don't add Python-side rendering, a `papyri ingest` CLI, or the
   JupyterLab extension.** Dangling references to `render.py`, `rich_render`,
   `textual`, `ipython`, `jlab`, `install`, `browse`, or `serve` should be
   deleted, not restored.
4. **Python 3.13+.** `requires-python = ">=3.13"`. CI runs on 3.14.
   Don't add shims for anything older than 3.13.
5. **Verify locally before committing.** At minimum:
   ```
   pip install -e .
   papyri gen examples/papyri.toml --no-infer
   # then run a viewer instance and:
   papyri upload ~/.papyri/data/papyri_<version>
   python -m pytest
   ```
   Run `python -m pytest` (not bare `pytest`) so the editable install's
   interpreter is used. If `papyri.db` complains about schema, `rm -rf
   ~/.papyri/ingest/` and re-upload to a fresh viewer instance.
6. **Enable the pre-commit hook — required before your first commit.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyter/papyri](https://github.com/jupyter/papyri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
