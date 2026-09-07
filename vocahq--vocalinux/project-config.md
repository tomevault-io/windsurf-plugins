---
trigger: always_on
description: Voice dictation for Linux: GTK 3 tray app (Python) plus a Next.js marketing site in `web/`. Default speech engine is **whisper.cpp** (`pywhispercpp`); OpenAI Whisper, Vosk, and a user-configured remote API are optional. Do not invent features, user counts, or privacy claims.
---

# AGENTS.md — Vocalinux

Voice dictation for Linux: GTK 3 tray app (Python) plus a Next.js marketing site in `web/`. Default speech engine is **whisper.cpp** (`pywhispercpp`); OpenAI Whisper, Vosk, and a user-configured remote API are optional. Do not invent features, user counts, or privacy claims.

## Critical: git worktrees for every branch and PR

Never create a branch, commit, or open a pull request in the primary checkout. Always use a linked git worktree so the main working tree stays on `main` and stays clean. Do not `git switch` / `git checkout` a feature branch in the primary directory, and do not leave it dirty.

```bash
git fetch origin
git worktree add /tmp/vocalinux-<task> -b <type>/<short-name> origin/main

# All edits, commits, and `gh pr create` happen inside that worktree.

git worktree remove /tmp/vocalinux-<task>
git worktree prune
```

Rules:

- One worktree per branch, one branch per PR
- Place worktrees **outside** the primary working tree (`/tmp/vocalinux-<task>` or a sibling directory such as `../.worktrees/vocalinux-<task>`)
- Never run two tasks in the same worktree
- Never commit directly to `main`
- Clean up the worktree after the PR is pushed

## Toolchain

| Piece | Current |
|---|---|
| Python | `>=3.11` (`requires-python` in `pyproject.toml`; CI: 3.11–3.14). Floor, classifiers, CI matrix and `install.sh` must agree — `tests/test_python_version_policy.py` |
| GTK | GTK 3 via distro `python3-gi` (PyGObject). Never pip-install it |
| uv | `>=0.12,<0.13` (`[tool.uv]` in `pyproject.toml`). `uv.lock` is the source of truth; CI runs `uv sync --locked` / `uv run --locked`, which fails on drift |
| just | https://just.systems or distro package `just` |
| Format / lint / types | Black + isort (line length 100), flake8 (`E9,F63,F7,F82` only), mypy `src/` (targets 3.11). The three linters live in the `lint` dependency group, not the `dev` extra, so CI can install them without building the project |
| Website | Next.js / TypeScript in `web/` — see `web/AGENTS.md` |

Two virtualenvs, on purpose:

- **`.venv/`** — uv's, for dev tooling. `just deps` syncs it; every other Python `just` recipe uses `uv run --no-sync` so it does not prune extras that `just deps-all` installed. No activation needed.
- **`venv/`** — what `install.sh` builds for the app itself, from the *system* Python so distro `gi` is importable.

Never run `install.sh` from an activated `.venv`: it drops an inherited `VIRTUAL_ENV` from `PATH` and picks `$SYSTEM_PYTHON` (default `/usr/bin/python3`) precisely because a venv built from uv's interpreter cannot see distro PyGObject.

## Setup

```bash
./install.sh --dev                 # system deps + venv + editable install + tests
# non-interactive / no TTY:
./install.sh --dev --auto
./install.sh --dev --auto --no-rebuild-whispercpp   # skip cmake/Vulkan rebuild

source venv/bin/activate
```

Manual venv (must see distro `gi`):

```bash
uv venv --system-site-packages --python /usr/bin/python3
source venv/bin/activate
# exclude pygobject from uv sync/export:
#   --no-install-package pygobject / --no-emit-package pygobject
uv pip install -e ".[dev,vad]"
```

`install.sh` flags: `--engine=whisper_cpp|whisper|vosk|remote_api`, `--test`, `--skip-models`, `--venv-dir=PATH`. Default engine is `whisper_cpp`.

## Commands

```bash
just lint          # flake8 (critical) + black --check + isort --check
just format        # black + isort
just typecheck     # mypy src/
just test          # pytest -v
just test-cov      # pytest --cov=src --cov-report=html
just deps          # sync .venv with dev+vad extras and the lint group
just deps-all      # also whisper/vosk/docs; later recipes use --no-sync so they keep it
just lock          # regenerate uv.lock + requirements/*.txt
just lock-check    # fail if uv.lock is stale vs pyproject.toml
just model-checksums  # refresh pinned model digests after adding a model
just appimage      # build the AppImage in its pinned base image (needs docker)
just appimage-boot fedora:42   # boot that AppImage in a distro container
just aur-gate      # build the AUR PKGBUILD on current Arch (needs docker)
just verify-release  # check a published release as published (needs gh)
just pre-commit    # pre-commit run --all-files
just run-debug     # vocalinux --debug
just run-source-debug
```

```bash
pytest
pytest tests/test_command_processor.py
pytest tests/test_command_processor.py::TestCommandProcessor::test_initialization
pytest -m "not slow"
pytest -m "not integration"
python -m vocalinux.main --debug
```

Website: `web/AGENTS.md`, `web/PRODUCT.md`, `web/DESIGN.md`. Do not duplicate site commands here.

## Dependencies (uv)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VocaHQ/vocalinux](https://github.com/VocaHQ/vocalinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
