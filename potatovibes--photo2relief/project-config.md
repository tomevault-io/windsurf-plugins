---
trigger: always_on
description: Working notes and conventions for Claude Code. **`SPEC.md` is the source of truth** for architecture, API contract, parameters, and milestone acceptance criteria. If this file and SPEC.md conflict, SPEC.md wins; if reality and SPEC.md conflict, update the Decisions Log below and note it here.
---

# CLAUDE.md — Photo2Relief

Working notes and conventions for Claude Code. **`SPEC.md` is the source of truth** for architecture, API contract, parameters, and milestone acceptance criteria. If this file and SPEC.md conflict, SPEC.md wins; if reality and SPEC.md conflict, update the Decisions Log below and note it here.

## What this project is

Single-user, locally-hosted Docker web app: photo → monocular depth (Depth Anything family, swappable via model registry) → tunable heightmap → watertight relief mesh → binary STL/OBJ in **millimeters** for Autodesk Fusion CAM. Owner's machine: Windows desktop, RTX 3080, Docker; GPU compose is the primary run mode. Port 8090.

## How to work

- Build strictly in milestone order (SPEC §6, M1→M6). Do not start a milestone until the previous one's acceptance criteria pass. Update the status table below as you go.
- **Commit as you go, not just at milestone boundaries.** Make small, reviewable commits per logical unit *within* a milestone (e.g. registry, then adapters, then endpoint wiring, then tests) — don't batch a whole milestone into one commit. Conventional-commit style messages (`feat:`, `fix:`, `test:`, `docs:`, `chore:`). Run `ruff check .` + `ruff format .` and the fast test suite before each commit.
- Write the tests alongside the code, not after. `heightmap.py` and `meshing.py` are pure functions — keep them that way (no I/O, no globals) and hold them to ≥90% branch coverage.
- Never mark a milestone done on "it should work" — run the acceptance checks and paste evidence (test output, timing numbers, debug PNG paths) into the status table notes.
- Items needing the owner (e.g., the Fusion import check in M4) → add to **Owner checklist** below and continue with what's unblocked.

## Commands

```bash
uv sync                                          # deps
uv run uvicorn app.main:app --reload --port 8090 # dev server
uv run pytest                                    # fast tests
uv run pytest -m slow                            # + real-inference smoke test
uv run ruff check . && uv run ruff format .      # lint/format (run before every commit)
docker compose up --build                                                   # CPU
docker compose -f docker-compose.yml -f docker-compose.gpu.yml up --build   # GPU (primary)
```

## Releasing & version tags

Pushing a git tag matching `v*` triggers `.github/workflows/publish.yml`, which builds and pushes both the CPU (`:X.Y.Z`, `:latest`) and GPU (`:X.Y.Z-gpu`, `:latest-gpu`) images to `ghcr.io/potatovibes/photo2relief`. The tag name **is** the release version — nothing else sets it. Rules:

- **Tag only from `main`, with a clean tree, after the full suite and lint pass.** A tag is a public release artifact: `uv run pytest` (green) + `uv run ruff check . && uv run ruff format --check .` (clean) before tagging, same bar as a commit but non-negotiable.
- **Use SemVer `vX.Y.Z`** (the leading `v` is what the workflow's `tags: ['v*']` trigger matches — a tag without it silently publishes nothing):
  - **X** (major) — a breaking change to the API contract (`SPEC.md`), the exported mesh/units semantics, env-var config, or the Docker run interface.
  - **Y** (minor) — a backward-compatible feature (new param, new model in the registry, new endpoint).
  - **Z** (patch) — a backward-compatible bug fix, doc, or packaging change only.
- **Don't tag mid-milestone.** A release should sit on a coherent, acceptance-passing state. `v1.0.0` is gated on M6 (see status table). Before that, `v0.y.z` pre-releases are fine **and encouraged once** to smoke-test the publish pipeline end-to-end (verify both images appear in GHCR and run).
- **Tags are immutable — never move or force-push one.** If a release is broken, fix forward with the next patch tag (`v1.0.1`); never re-point `v1.0.0` at a new commit (it desyncs anyone who already pulled that image digest).
- **Bump the version in one place if/when one exists** (e.g. `pyproject.toml [project].version`) in the same commit you tag, so the code's self-reported version matches the tag. As of this writing there is no such field; add the bump step here if one is introduced.
- **How to cut a release:**
  ```bash
  git switch main && git pull            # clean, up to date
  uv run pytest && uv run ruff check . && uv run ruff format --check .
  git tag -a v1.0.0 -m "Photo2Relief v1.0.0"   # annotated tag
  git push origin v1.0.0                  # this line triggers the build
  ```
  Then confirm the run under the repo's **Actions** tab and that both image variants published. First-ever publish: set each GHCR package's visibility to **Public** once (it sticks).

## Conventions & hard rules

- Python 3.12, `uv`-managed. Type hints everywhere; pydantic models in `schemas.py` are the single definition of `ReliefParams` and all API payloads.
- **Never re-run depth inference on a parameter change.** Raw depth caches on `(image_hash, model_id, MAX_INFER_PX)` as `raw_depth_{model_id}.npy` in the session dir.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PotatoVibes/photo2relief](https://github.com/PotatoVibes/photo2relief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
