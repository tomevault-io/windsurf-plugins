---
trigger: always_on
description: validates them up front, so a missing key surfaces as an error inside the flow. In production they are Fly secrets
---

# VERDAD backend: guide for coding agents

VERDAD records Spanish- and Arabic-language US radio stations around the clock and runs the audio through a
five-stage pipeline (Gemini 2.5 + OpenAI embeddings) that flags, clips, analyzes, reviews and embeds suspected
mis/disinformation. Results land in Supabase (Postgres + pgvector) and are reviewed by journalists in
[verdad-frontend](https://github.com/publicDataWorks/verdad-frontend). Orchestration is Prefect 3 on Fly.io.

## Layout (what is not obvious from file names)

- Path-specific guidance lives in `.claude/rules/` (pipeline stages, recorders, Supabase SQL, prompts,
  tests); each file loads only when you read files it matches.
- `src/processing_pipeline/stage_{1..5}/{flows,tasks,executors,models}.py`: one package per stage.
  `flows.py` = Prefect flow (loop that fetches work from Supabase), `tasks.py` = steps, `executors.py` =
  the LLM call. Stage 4 is `executor.py` + `agents.py` (Google ADK agent pipeline) and stage 2 has no executor.
- `src/processing_pipeline/main.py`: production entrypoint. Reads `FLY_PROCESS_GROUP` and `serve()`s the matching
  Prefect deployment (`fly.processing_worker.toml` `[processes]` lists the 11 valid values). With no value it raises.
- `src/recording.py` (ffmpeg stream recorders) and `src/generic_recording.py` + `src/radiostations/` (Selenium/Chrome
  recorders for six web-only stations). Same `FLY_PROCESS_GROUP` dispatch.
- `config/stations.yaml` + `src/stations.py`: the station list and its loader/validator (`load_stations`,
  `stations_for`, `station_dicts`, and `python -m stations prefect-runs` for `scripts/start_recording.sh`).
- `src/utils.py`: `optional_flow`/`optional_task` decorators and `fetch_radio_stations()` (see gotchas).
- `src/processing_pipeline/supabase_utils.py`: the only DB access layer (`SupabaseClient`).
- `prompts/`: prompt sources, but the pipeline reads prompts from the `prompt_versions` table; `prompts/manifest.json`
  names each entry's files and semver version. `src/scripts/import_prompts_to_db.py` moves files to the DB
  (`import --from-manifest` or `import --version`, `list`, `diff`); `src/scripts/evaluate_prompt.py` measures a
  Stage 3 change on the eval sets in `prompts/eval/`. See `docs/PROMPT_MANAGEMENT.md`, `docs/PROMPT_EVALUATION.md`
  and `.claude/rules/prompts.md`.
- `supabase/`: `migrations/` is the source of truth (generated baseline of the live schema + one file per
  applied version); `supabase/database/sql/` is historical hand-applied SQL, see its README and the
  "Database schema and migrations" section of `docs/OPERATIONS.md`. `server/`: separate
  Express/TS app (Liveblocks auth, Resend email) with its own Dockerfile and `fly.server.toml`.
- `scripts/*.sh` + `Dockerfile.*` + `fly.*.toml`: deploy and cron. See `docs/OPERATIONS.md`. `scripts/ci/`: helpers
  for the prompt CI workflows (`.github/workflows/prompts-*.yml`, `prompt-evaluation.yml`).
- `docs/design-2024.md`: historical design doc (two-stage era). Do not treat it as current.

## Commands

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt   # needs ffmpeg on PATH
make check          # ruff check + pytest with coverage gate; what CI runs. Run before every commit.
make lint           # ruff check src tests scripts + scripts/check_rules.py
make test           # pytest (coverage gate = [tool.coverage.report] fail_under in pyproject.toml)
pytest tests/processing_pipeline/test_stage_3.py -k executor --no-cov     # one file/test, fast
make format         # ruff format; only run on files you are already changing (repo is not yet fully formatted)
python scripts/run_stage.py --stage 3 --snippet-id <uuid>                  # run one stage locally, no Prefect
PYTHONPATH=.:src python src/scripts/import_prompts_to_db.py import --from-manifest [--dry-run]   # what CI runs on merge
PYTHONPATH=.:src python src/scripts/import_prompts_to_db.py import --version 1.2.0 --description "..." [--dry-run]
PYTHONPATH=.:src python src/scripts/import_prompts_to_db.py diff [--stages stage_1/initial_detection ...] [--show-diff]   # prompts-vs-DB drift
```

- `pre-commit install` enables ruff on staged files; `hooks/install-hooks.sh` installs a lint+test pre-push hook.

## Environment variables

Complete list with one-line explanations: `.env.sample`. Everything is read with `os.getenv` at flow start; nothing
validates them up front, so a missing key surfaces as an error inside the flow. In production they are Fly secrets
(`fly secrets set -a <app>`); `PREFECT_API_URL` and `FLY_PROCESS_GROUP` come from the `fly.*.toml` files.

## How deploy works

- One Fly app per `fly.*.toml`, deployed manually: `fly deploy -c fly.processing_worker.toml` (there is no deploy CI
  for the apps; prompt changes are the exception, see the gotcha below).
  Each `[processes]` entry becomes a machine whose `FLY_PROCESS_GROUP` selects the Prefect deployment to serve.
- The `prefect` app runs the Prefect server plus a `cron` machine (supercronic): `scripts/restart_all.sh` every 6 hours
  cancels all flow runs, `fly machine restart`s `$FLY_MACHINE_IDS`, then re-triggers `start_recording.sh` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PublicDataWorks/verdad](https://github.com/PublicDataWorks/verdad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
