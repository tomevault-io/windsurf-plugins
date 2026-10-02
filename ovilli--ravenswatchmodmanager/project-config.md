---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

Hybrid monorepo with two parallel toolchains:

- **Python CLI** (`src/rsmm/`, entry point `./rsmm` at repo root) — all mod install / lifecycle logic. Stdlib-only at runtime (`pyproject.toml` declares no `dependencies`). Installed editable via `pip install -e .`.
- **TypeScript pnpm workspace** (`apps/*` + `packages/*`) — Tauri 2 desktop shell, Hono API, Next.js site, Astro docs, shared `@rsmm/*` packages. Orchestrated by Turbo (`turbo.json`).
- **Native loader DLL** (`src/loader/`, Windows-only) — `winhttp.dll` proxy + MinHook + Lua 5.4 VM injected into Ravenswatch for Lua-scripted mods. Built with CMake. Texture/asset overrides work without it. The Lua SDK (`src/loader/lib/rsmm.lua` + the generated `engine_gen.lua`/`events_gen.lua`, plus the `src/loader/lua/rsmm/*.lua` submodules it requires) is **disk-loaded from `<game>/rsmm/lib/`, not embedded in the DLL** — a Lua-only change ships via `rsmm install-loader` (or a straight file copy), no rebuild.

The desktop app does **not** reimplement the CLI — it bundles the Python CLI as a PyInstaller sidecar (`apps/desktop/src-tauri/binaries/rsmm-<triple>[.exe]`) and shells out via Tauri's `shell:allow-execute`. See `scripts/build-sidecar.py` for the bundle definition (every data file the frozen CLI needs must be in `add_data_args` or it will crash on a fresh user install).

## Common commands

| Task | Command |
|------|---------|
| Install Python CLI editable | `pip install -e .` (after `python3 -m venv .venv && source .venv/bin/activate`) |
| Install JS deps | `pnpm install` |
| Desktop app (Tauri dev) | `pnpm dev` (= `turbo run dev --filter=desktop`). **Not `npm run dev`** — that makes corepack fetch the pinned `pnpm@9.12.0`, and turbo swallows corepack's confirm prompt, so the run stops dead after `! Corepack is about to download …` with no window and no error. `export COREPACK_ENABLE_DOWNLOAD_PROMPT=0` in your shell rc; it recurs on a fresh clone or an nvm version switch. |
| Desktop app w/ local CLI | `pnpm --filter desktop dev:with-cli` (puts repo root on PATH so it uses `./rsmm` not the bundled sidecar) |
| API server (`:3001`) | `pnpm api:dev` |
| Website (`:3000`) | `pnpm www:dev` |
| Docs site (`:4321`) | `pnpm docs:dev` |
| Item / talent / ability / map editor | `rsmm editor [--tab items\|talents\|abilities\|map]` (local page); the same editors run in the browser at `docs.rsmm.me/editor/` |
| Lint TS | `pnpm lint` / `pnpm lint:fix` (Biome) |
| Lint Python | `ruff check .` (config in `pyproject.toml` — `src/loader/third_party` excluded) |
| Type-check TS | `pnpm check-types` |
| All Python tests | `pnpm test:python` (= `python -m pytest`) |
| Fast Python tests (~2s) | `pnpm test:python:fast` (= `pytest -n auto -m "not slow"`; heavy corpus-scan tests are auto-marked `slow`) |
| Single Python test | `pytest tests/test_apply_restore.py::test_name` |
| Loader Lua SDK spec | `lua tests/lua/rsmm_spec.lua` (also wrapped as `pytest tests/test_loader_lua.py`) — engine-faithful mocks; extend it when touching `rsmm.lua` engine-call paths |
| TS tests (vitest) | `pnpm test:ts` — `@rsmm/schemas` + `@rsmm/api-client` + `desktop` + `www` |
| Regen symbol artifacts | `rsmm symbols gen` (after editing `data/symbols.json`; CI runs `--check`) |
| Local Postgres | `pnpm db:up` then `pnpm db:push` (Drizzle) |
| Publish a loader/SDK update (no app release) | `gh workflow run publish-loader.yml -f notes="what changed"` (the signing key is a repo secret; `scripts/publish_loader.sh` is what it runs). **If the run's last step fails, the channel is already live and only the stamp is missing** — read `loader.manifest.json` off the `loader` release, set `data/loader_version.json` to its `loader_version`, take `data/changelog.json` from the `changelog` release, and commit both, or main will claim an older loader than users are served |
| Build PyInstaller sidecar | `python scripts/build-sidecar.py` (CI replicates this inline in `.github/workflows/release.yml` — keep both in sync) |
| Build loader DLL (Win) | `src\loader\build.bat` |
| Build loader DLL (Linux→Win, MinGW) | `src/loader/build.sh` |
| Bump versions for release | `python scripts/bump-version.py patch` (or `minor`/`major`/explicit `0.1.12`) — updates all 4 version files atomically |
| Mirror cooked assets to `data/uncooked/` | `python scripts/extract_uncooked.py` (textures → PNG, everything else copied; sound banks come from `audio_pairs()`, not the CSV). **The mirror is pruned to match**: anything the run did not write is deleted, so a renamed asset cannot leave its old spelling behind (that is what made the 2026-09-22 cipher fix break `test_poi.py` with a GUID collision). `--prune-dry-run` to preview, `--no-prune` to skip; a `--limit`/`--filter`/errored run never prunes, since it only describes a slice. `Audio/extracted/` (extract_audio.py) and live `.gen.txt` sidecars (decode_gen_sidecars.py) are exempt |
| Unpack sound banks to playable audio | `python scripts/extract_audio.py` (`--wav`, `--filter <Bank>`) — needs `pip install fsb5`; writes `data/uncooked/Audio/extracted/<Bank>/<Sample>.ogg` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ovilli/RavenswatchModManager](https://github.com/Ovilli/RavenswatchModManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
