---
trigger: always_on
description: <!-- BEGIN issue-flow (managed: do not edit this block) -->
---

<!-- BEGIN issue-flow (managed: do not edit this block) -->
# Issue-flow best practices


## Running python

**Respect the project's existing toolchain first.** If this project already
documents how to run Python and manage dependencies — in its `README`,
`AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, `environment.yml`, `pyproject.toml`,
`Makefile`, CI config, etc. — **follow that**, even where it conflicts with the
defaults below. These rules describe issue-flow's *default* assumptions, not a
mandate to override a project that has already chosen differently.

The one tool-neutral principle: **don't call bare `python ...`** — invoke Python
through the project's environment (its runner, or an activated virtualenv/conda
env) so scripts and tests see the right interpreter and dependencies.

### If the project uses conda

When the project documents a conda environment, run **all** Python commands —
scripts **and `pytest`** — inside the **activated conda environment**. Do **not**
substitute `uv run`.

```bash
# Either activate the environment first…
conda activate <env-name>
python run_script.py
pytest

# …or run one-off commands inside it:
conda run -n <env-name> pytest
```

### If the project uses uv (issue-flow's default)

For projects scaffolded fresh (and this is the default when nothing else is
documented), use `uv`:

```bash
# ❌ BAD: bare interpreter
python run_script.py

# ✅ GOOD: through uv
uv run run_script.py
```

**Package management with `uv`**

- Install, synchronize, and lock dependencies with `uv`; don't reach for `pip`,
  `pip-tools`, or `poetry` in a uv-managed project.

```bash
# Add or upgrade dependencies
uv add <package>

# Remove dependencies
uv remove <package>

# Reinstall all dependencies from the lock file
uv sync

# Run a script with the right environment
uv run script.py
```

### Other toolchains (plain venv / pip / poetry)

If the project uses something else, use whatever it documents (e.g. activate its
`.venv` and use `pip`, or run `poetry run`). Match the project; don't force `uv`.


## Issue tracking structure

```bash
cellpy/
    .issueflows/
        00-tools/
        01-current-issues/
            issueXX_original.md
            issueXX_status.md
        02-partly-solved-issues/
        03-solved-issues/
        04-designs-and-guides/
        05-epics/
            epicXX_plan.md
    pyproject.toml
    readme.md
    ...
```


## Development information


### Working on issues

After each iteration, update the documents in `.issueflows/01-current-issues` (should contain one file labelled `_original` with the original issue description, a `_plan` file with the confirmed approach, and supplementary status files describing what has been done, current status, and remaining work).
Use an explicit status checkbox in the status file:
- `- [x] Done` when fully resolved
- `- [ ] Done` when not fully resolved

### Chat invocation (no slash)

On keyboard layouts where `/` and `@` are awkward to type (for example Norwegian), invoke lifecycle skills in **chat** without special keys.

**Primary form:** `iflow <step>` (space-separated) — e.g. `iflow plan`, `iflow pick`, `iflow close`. Plain `iflow` runs the smart dispatcher.

**Also recognized** (same obligation as slash-menu invocation): hyphen form (`iflow-plan`), slash form (`/iflow-plan`), slash + space (`/iflow plan`).

When the user message is **exactly** one of these forms, or **starts with** it followed by a space and trailing arguments, **read and follow** the matching skill immediately. Forward trailing text verbatim (e.g. `iflow pick fix` → `iflow-pick` with arg `fix`). Do **not** treat incidental mid-sentence mentions as commands — the message must **start with** the invocation.

| Chat / slash form | Skill |
|-------------------|-------|
| `iflow` / `/iflow` | `iflow` (dispatcher) |

| `iflow archive`, `iflow-archive`, `/iflow-archive`, `/iflow archive` | `iflow-archive` |

| `iflow auto`, `iflow-auto`, `/iflow-auto`, `/iflow auto` | `iflow-auto` |

| `iflow build`, `iflow-build`, `/iflow-build`, `/iflow build` | `iflow-build` |

| `iflow cleanup`, `iflow-cleanup`, `/iflow-cleanup`, `/iflow cleanup` | `iflow-cleanup` |

| `iflow close`, `iflow-close`, `/iflow-close`, `/iflow close` | `iflow-close` |

| `iflow cycle`, `iflow-cycle`, `/iflow-cycle`, `/iflow cycle` | `iflow-cycle` |

| `iflow doctor`, `iflow-doctor`, `/iflow-doctor`, `/iflow doctor` | `iflow-doctor` |

| `iflow epic`, `iflow-epic`, `/iflow-epic`, `/iflow epic` | `iflow-epic` |

| `iflow fix`, `iflow-fix`, `/iflow-fix`, `/iflow fix` | `iflow-fix` |

| `iflow graphify`, `iflow-graphify`, `/iflow-graphify`, `/iflow graphify` | `iflow-graphify` |

| `iflow init`, `iflow-init`, `/iflow-init`, `/iflow init` | `iflow-init` |

| `iflow issue`, `iflow-issue`, `/iflow-issue`, `/iflow issue` | `iflow-issue` |

| `iflow pause`, `iflow-pause`, `/iflow-pause`, `/iflow pause` | `iflow-pause` |

| `iflow pick`, `iflow-pick`, `/iflow-pick`, `/iflow pick` | `iflow-pick` |

| `iflow plan`, `iflow-plan`, `/iflow-plan`, `/iflow plan` | `iflow-plan` |

| `iflow review`, `iflow-review`, `/iflow-review`, `/iflow review` | `iflow-review` |

| `iflow status`, `iflow-status`, `/iflow-status`, `/iflow status` | `iflow-status` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jepegit/cellpy](https://github.com/jepegit/cellpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
