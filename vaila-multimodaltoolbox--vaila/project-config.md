---
trigger: always_on
description: Rules for developing and maintaining the vailá multimodal biomechanical toolbox (Python 3.12, Astral toolchain)
---


# vailá Development Rules

## Project Identity
- **vailá** — open-source Python 3.12 biomechanical analysis toolbox (AGPLv3)
- Sensors: IMU, MoCap, Markerless (MediaPipe/YOLO), EMG, Force Plates, GNSS/GPS
- GUI: **Tkinter only** — never introduce Qt, wx, Dear PyGui, or any other GUI framework
- Build: `hatchling` backend managed via `uv`

---

## Astral Toolchain (mandatory)

The project uses the full Astral Rust-based stack. **Never use** bare `pip`, `black`, `isort`, `flake8`, or `mypy`.

| Tool | Role | Docs |
|------|------|------|
| `uv` | Package manager, venv, Python installer | https://docs.astral.sh/uv/ |
| `ruff` | Linter + formatter | https://docs.astral.sh/ruff/ |
| `ty` | Type checker (beta) | https://docs.astral.sh/ty/ |

---

## uv Commands

```bash
uv run vaila.py                  # run the app
uv sync                          # default: CUDA 12.8 PyTorch (pytorch-cu128 index)
uv sync --extra gpu              # adds tensorrt + nvidia-ml-py on CUDA workstation
uv sync --extra sam              # optional SAM 3; CUDA required at runtime
uv sync --extra fifa             # FIFA Skeletal Tracking Light stack
uv sync --frozen                 # CI mode: strict lock
uv add <pkg>                     # add runtime dep
uv add --dev <pkg>               # add dev dep
uv lock --upgrade                # upgrade all packages
uv export --format requirements-txt > requirements.txt
```

> Always prefix tool runs with `uv run` — never activate the venv manually.

> **Note (v0.3.44):** the checked-in `pyproject.toml` now points `torch`,
> `torchvision`, `torchaudio` at the explicit `pytorch-cu128` index. CPU-only
> machines must switch to the universal template before `uv sync`
> (`bash bin/use_pyproject_universal_cpu.sh`).

---

## ruff Commands

```bash
uv run ruff check vaila/ --fix        # lint + auto-fix
uv run ruff format vaila/             # format (replaces black)
uv run ruff check vaila/ --diff       # preview lint fixes
uv run ruff format vaila/ --check     # CI: check without writing
```

**Config in `pyproject.toml`:**
```toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "W", "F", "I", "N", "NPY", "UP", "B", "C4", "SIM"]
ignore = ["E501", "N806", "N803"]  # scientific uppercase vars OK

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]           # intentional re-exports
```

**Inline suppression:** `# noqa: F841` or `# noqa: F841, E501`

---

## ty Commands

```bash
uv run ty check vaila/                # type-check all files
uv run ty check vaila/ --watch        # watch mode (re-checks on save)
uv run ty check vaila/ --error unresolved-import
uv run ty check vaila/ --warn  possibly-unbound
uv run ty check vaila/ --ignore division-by-zero
```

**Config in `pyproject.toml`:**
```toml
[tool.ty.rules]
unresolved-import     = "warn"
possibly-unbound      = "warn"
division-by-zero      = "error"
unused-ignore-comment = "warn"

[tool.ty.src]
include = ["vaila", "tests"]
```

**Inline suppression:** `# ty: ignore[invalid-assignment]`

> `ty` is in **beta** — use it as a complement to ruff, not a strict gate.

---

## Full QA Pipeline (run before every commit)

```bash
uv run ruff check vaila/ --fix && \
uv run ruff format vaila/         && \
uv run ty check vaila/            && \
uv run pytest tests/ -v
```

---

## Python Version & Style

- Target: **Python 3.12 strictly** (`>=3.12,<3.13`)
- Use `match/case`, `tomllib`, structural pattern matching where appropriate
- Scientific variable names (X, Y, Z, F, R, T, etc.) are **valid** — suppressed in ruff
- Output dirs: always timestamped → `processed_<type>_YYYYMMDD_HHMMSS/`
- No hard-coded absolute paths
- No files ≥20 MiB (git hook enforced via `install-hooks.sh`)

---

## Mandatory Dual-Import Pattern

Every `vaila/` module must support both package import AND standalone execution:

```python
try:
    from .common_utils import get_headers      # package import
    from .dialogsuser import ask_for_directory
    from .filtering import butter_filter
except ImportError:
    from common_utils import get_headers       # standalone fallback
    from dialogsuser import ask_for_directory
    from filtering import butter_filter
```

---

## New Module Template

```python
"""Short description of this module."""
from __future__ import annotations

from pathlib import Path
import numpy as np
import pandas as pd

try:
    from .common_utils import get_headers
    from .dialogsuser import ask_for_directory
except ImportError:
    from common_utils import get_headers
    from dialogsuser import ask_for_directory


def run_my_module() -> None:
    """GUI entry point — called from vaila.py."""
    input_dir = ask_for_directory("Select input directory")
    if not input_dir:
        return
    # ... analysis logic ...
    output_dir = Path(input_dir) / f"processed_mymodule_{timestamp()}"
    output_dir.mkdir(parents=True, exist_ok=True)
    # write CSV + PNG results to output_dir


if __name__ == "__main__":
    # Optional CLI support
    run_my_module()
```

---

## GUI Dispatch in vaila.py

### Pattern 1 — Direct import (same process)
```python
def _on_my_analysis(self) -> None:
    from vaila import my_module
    my_module.run_my_module()
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vaila-multimodaltoolbox/vaila](https://github.com/vaila-multimodaltoolbox/vaila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
