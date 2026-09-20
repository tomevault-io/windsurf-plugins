---
trigger: always_on
description: Canonical, tool-agnostic guide for AI coding agents (and humans) working **on**
---

# AGENTS.md — pyobfus

Canonical, tool-agnostic guide for AI coding agents (and humans) working **on**
the pyobfus codebase. Tool-specific files defer here: `CLAUDE.md` imports this
file; Cursor / Windsurf / Aider / Codex read `AGENTS.md` natively.

> Looking to *use* pyobfus to protect your own code, not develop it? See the
> [`pyobfus-review`](skills/pyobfus-review/SKILL.md) skill (read-only: should
> we obfuscate this, and what breaks?), the
> [`pyobfus-protect`](skills/pyobfus-protect/SKILL.md) skill (do it, and verify
> the result), and the [`templates/ai-integration/`](templates/ai-integration/)
> rule files instead.

## What this project is

pyobfus is an **AST-based Python code obfuscator** — framework-aware presets,
reverse stack-trace mapping for AI-assisted debugging, and a machine-readable
JSON CLI. A transparent, open-source alternative to PyArmor. The repo ships
**two packages**:

- `pyobfus/` — the obfuscator (CLI + library). Published as `pyobfus`.
- `pyobfus_mcp/` — an MCP server exposing the tools to AI agents. Published as
  `pyobfus-mcp`.

Plus `pyobfus_pro/` (commercial, license-gated features) kept source-separated
from the Apache-2.0 core.

## Setup

Use the repository-local **`venv/`** on WSL/Linux. Do **not** use `.venv/`:
that directory is a Windows-side legacy environment and WSL cannot reliably run
its executables. Either activate `venv/` first, or call tools through
`venv/bin/...` directly.

```bash
python -m venv venv && source venv/bin/activate
pip install -e ".[dev]"
git config core.hooksPath .githooks   # once per clone — enables the pre-commit guard (PII + credentials)
```

## Build / test / lint — run before every commit

```bash
venv/bin/pytest tests/                 # core suite (run this and the two below separately)
venv/bin/pytest pyobfus_mcp/tests/     # MCP server suite
venv/bin/pytest integration_tests/     # end-to-end CLI
venv/bin/black pyobfus/                # format
venv/bin/ruff check pyobfus/           # lint
venv/bin/mypy pyobfus/                 # type check
```

Note: the core and MCP test roots are collected as **separate** pytest
invocations (CI runs them as separate jobs) — don't point one `pytest` at both
roots at once.

Tests must never read, write, or delete a developer's real pyobfus state under
`~/.pyobfus` (notably `trial.json` and `license.json`) or place generated run
counters in the real home directory. Bind module-level state paths and any
generated-code `HOME` / `USERPROFILE` lookup to pytest's per-test `tmp_path`.
The standard commands above must pass with the caller's normal HOME; requiring
`HOME=/tmp/...` is a test-isolation regression, not an accepted prerequisite.

**4th test root — `vscode-extension/`** (Node/npm, not pytest; independent
package, see `docs/VSCODE_EXTENSION_PLAN.md`):

```bash
cd vscode-extension
npm ci
npm run lint          # eslint
npm run typecheck     # tsc --noEmit
npm run pretest       # esbuild + compile tests to out/
PYOBFUS_PYTHON_PATH="$(cd .. && pwd)/venv/bin/python3" npm test
```

`npm test` needs a **resolvable interpreter with pyobfus actually
installed** for the real-contract integration tests (`test/suite/
integration.test.ts`) — without `PYOBFUS_PYTHON_PATH` set, interpreter
resolution falls back to a bare `python3`/`python` on PATH (the
`ms-python.python` extension isn't active inside the plain
`@vscode/test-electron` test profile), which on a fresh machine likely
doesn't have pyobfus installed and fails 4 of the tests with a "no module
named pyobfus" error — not a real bug, just a missing env var. WSLg (or
any real X server) is required for `@vscode/test-electron` to launch;
`xvfb-run -a npm test` works too if there's no display available.

CI runs this as a separate, path-filtered workflow
(`.github/workflows/vscode-extension-ci.yml`, sets the same env var to
`${{ env.pythonLocation }}/bin/python`), not as part of the Python
`ci.yml` jobs above.

Targets: Python **3.9–3.14** must all pass. (Python 3.8 was dropped in 0.5.0 —
EOL 2024-10 — which removed the old `astunparse`/`@requires_py39` flakiness;
`docs/PYTHON38_COMPATIBILITY.md` is retained only as historical record.)

## Self-dogfooding boundary

Use pyobfus itself for read-only analysis/reporting and exercise obfuscation on
maintained canary fixtures or freshly built wheels; do not replace the public
package sources with a self-obfuscated tree. A current checkout must not be the
only verifier of artifacts it produced: pair it with the latest public release,
normal Python tooling, independent schema validation, hosted attestations, and
fresh-environment install tests as appropriate.

Self-scan findings are initially audit evidence, not an automatic blocking
gate. Do not exclude the whole `pyobfus/` tree or weaken a general detection
merely to make the repository's own report green. Suppress intentional patterns
only through a reviewed, precise rule with a technical reason; claim SARIF
`baselineState` only after a complete comparison. See
[`docs/SELF_DOGFOODING_BEST_PRACTICES.md`](docs/SELF_DOGFOODING_BEST_PRACTICES.md)
for the staged policy and current baseline.

## Repository layout

```
pyobfus/            # core obfuscator: cli.py, config.py, core/, transformers/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhurong2020/pyobfus](https://github.com/zhurong2020/pyobfus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
