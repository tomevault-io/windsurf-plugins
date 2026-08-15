---
trigger: always_on
description: bmad-loop is a deterministic Python orchestrator that drives unattended BMAD-method dev loops by spawning coding-CLI sessions (claude, codex, gemini, copilot, antigravity, opencode) inside terminal multiplexers (tmux; psmux on Windows). **The control loop contains no LLM calls — hard rule.** Orchestration is deterministic Python; LLMs run only inside disposable coding-CLI sessions. Never move orchestration into an LLM. User-facing overview: [README.md](README.md); behavior reference: [docs/FEATU
---

# AGENTS.md — bmad-loop

bmad-loop is a deterministic Python orchestrator that drives unattended BMAD-method dev loops by spawning coding-CLI sessions (claude, codex, gemini, copilot, antigravity, opencode) inside terminal multiplexers (tmux; psmux on Windows). **The control loop contains no LLM calls — hard rule.** Orchestration is deterministic Python; LLMs run only inside disposable coding-CLI sessions. Never move orchestration into an LLM. User-facing overview: [README.md](README.md); behavior reference: [docs/FEATURES.md](docs/FEATURES.md).

## Dev environment

```bash
uv sync --all-extras          # setup (extras: tui, non-linux, opencode)
uv run pytest -q              # test suite (-n auto to parallelize)
uv run pytest tests/test_engine.py -q   # single file
uv run pyright                # typecheck — same pinned version CI runs
trunk fmt                     # format changed files
trunk check                   # lint changed files, as CI does — run before every push (pre-push hook enforces it)
trunk check --all             # whole-repo lint; the only way to catch untouched files (e.g. after a linter bump)
```

- Never `pip install`; uv owns the environment. Dependency changes: edit pyproject.toml, run `uv lock` (CI uses `uv sync --locked` and fails on a stale lock).
- Typecheck: `uv run pyright`. The version is pinned exactly in the `dev` group (pyproject.toml) and CI runs that same command, so there is one pin and no drift — bump it deliberately, never with `uv lock --upgrade`.
- Windows local runs require `PYTHONUTF8=1` (tests/conftest.py raises UsageError otherwise). Python floor: 3.11.

## Hard invariants

Things that break silently. Never violate; when in doubt, read the named module's docstring.

- No LLM calls in the orchestrator control loop.
- Sessions complete only on hook Stop events or window death — never on LLM prose. Never add another completion path. Post-session state is re-verified deterministically (`verify.py`).
- `sprintstatus.advance()` is the orchestrator's sole write path to sprint-status.yaml (the dev skill flips only spec frontmatter; the engine mirrors it onto the board pre-verify). Legal phase transitions live only in `statemachine.py`.
- All git subprocess calls in `src/bmad_loop` go through the `_run_git` chokepoint in `verify.py` (timeouts, `LC_ALL=C`) — no bare subprocess git; `tests/test_portability_guard.py` enforces it. Tests, `scripts/`, and CI workflows deliberately spawn their own git: a harness must not depend on the artifact it validates.
- Every new policy field needs an entry in `src/bmad_loop/data/settings/core.toml` (a sync test enforces defaults/options match `policy.py`). New core env vars register in `envvars.py`; plugin-owned env-var families stay with their plugin.
- Version strings are stamped only by `scripts/sync_version.py` from `src/bmad_loop/__init__.py` — never hand-edit pyproject.toml, module.yaml, marketplace.json, or uv.lock versions.
- Canonical module skills live in `src/bmad_loop/data/skills/`; their copies in the gitignored `.claude/skills/` and `.agents/skills/` are seeded forks (`scripts/seed_skills.py`) — editing a fork loses work on reseed (drift-tested locally; the test skips in CI). Other skills in those dirs are BMAD-installed, not seeded. (The `.agents/` directory is unrelated to this file.)
- `bmad-loop <cmd> --json` output is a schema-versioned contract (`machine.py`): one JSON object on stdout, nothing else.
- `ExitCode` allocation is closed (`cli.py`): OK=0, FAILURE=1, USAGE=2, INTERRUPTED=130; 3–129 and 131+ stay unallocated.
- tmux/POSIX argv construction is quarantined in `adapters/tmux_base.py` + `tmux_backend.py`; `tests/test_portability_guard.py` enforces it.
- Live/E2E tests must consume zero LLM tokens.

## Architecture

Two orthogonal seams: **which CLI** (adapter axis: `adapters/base.py` `CodingCLIAdapter`, TOML profiles in `src/bmad_loop/data/profiles/`, user overlay `.bmad-loop/profiles/*.toml` — a new coding CLI is a TOML profile plus `bmad-loop probe-adapter`, not Python; a CLI needing its own adapter **class** registers one in `adapters/registry.py` (`register_adapter`, `bmad_loop.adapters` entry point), selected by the profile's `adapter` field) and **which transport** (mux axis: `adapters/multiplexer.py` `TerminalMultiplexer` registry; selection: `BMAD_LOOP_MUX_BACKEND` env > policy `[mux] backend` > platform default > first available match > fallback — full 5-step precedence in [docs/multiplexer-backends.md](docs/multiplexer-backends.md)).

| Module                                     | Role                                                                                        |
| ------------------------------------------ | ------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bmad-code-org/bmad-loop](https://github.com/bmad-code-org/bmad-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
