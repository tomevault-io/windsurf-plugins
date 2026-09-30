---
trigger: always_on
description: KiForge is a KiCad 10 plugin and CLI exporter. It runs in three places with
---

# KiForge — working agreements

KiForge is a KiCad 10 plugin and CLI exporter. It runs in three places with
different constraints: inside KiCad's bundled Python (Studio), under a system
Python (CLI), and inside a headless Docker container (the GitHub Action).
Almost every rule below exists because one of those three broke.

Adapted from the [KiCad developer rules and
guidelines](https://dev-docs.kicad.org/en/rules-guidelines/index.html) where
they transfer. Those are written for KiCad's C++ application code; what carries
over is the reasoning, not the syntax.

---

## 1. The interpreter floor is 3.9 — this is the one that bites

KiCad bundles its own Python and **the version differs per platform**: macOS
ships 3.9.13 inside `KiCad.app`, other platforms ship newer 3.x. The supported
interpreter is a range with a floor, and every shipped module must run across
all of it.

The trap: `str | None` (PEP 604) is valid *syntax* on 3.9 but is evaluated at
`def` time, so it raises `TypeError` on import. Inside KiCad that is invisible —
PCM reports the package installed and no toolbar button ever appears.

- Shipped modules carry `from __future__ import annotations`.
- Do not use syntax or stdlib APIs newer than 3.9 (`match`, `tomllib`,
  `zip(strict=)`, `X | Y` at runtime).
- The floor is declared in four places that must agree:
  `plugins/__init__.py:MIN_PYTHON`, `pyproject.toml` `[tool.kiforge] min-python`,
  `[tool.ruff] target-version`, and the CI matrix.
  `tests/test_python_compat.py` fails if they drift.

## 2. Run these before saying it works

```bash
python tests/kicad_runtime_stub.py                       # load gate, no deps
python -m unittest discover -s tests -p 'test_*.py'      # headless suite
ruff check .                                             # FA102 compat rules
```

The strongest local check is the load gate under **KiCad's own Python**, which
is what the plugin actually runs on:

```bash
/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3 tests/kicad_runtime_stub.py
```

GUI tests need `KIFORGE_RUN_GUI_TESTS=1` and real wx, so run them with KiCad's
Python too.

**Gotcha eliminated:** `package_plugin.py` stages `kiforge.py` into `dist/staging/`
before zipping it as `plugins/kiforge.py`. The packager never writes to or leaves
`plugins/kiforge.py` in the workspace, ensuring `plugins/kiforge_studio.py`'s
`from . import kiforge` always imports the active repo-root module. Any stray
`plugins/kiforge.py` is automatically unlinked by the packager. `tests/test_studio.py`
retains a defensive guard to prevent shadowing if created manually.

## 3. Dependencies: resolve them lazily, at the point of need

**PCM has no install hook.** Verified against the [KiCad addon
specification](https://dev-docs.kicad.org/en/addons/index.html): a package is a
plain zip extracted to fixed locations, with no install script, no post-install
hook, and no way to declare Python dependencies. The `runtime` field selects
`ipc` vs `swig`, nothing more. The earliest code KiCad runs is
`plugins/__init__.py` at plugin-scan time, and pip-installing from there would
block KiCad's startup on a network download.

So a missing package is resolved **where it is first needed**, inside the export
task, through the cancellable `_run_subprocess` runner so progress reports and
Cancel keep working. `IbomExportTask` and
`HomebrewPdfExportTask._ensure_renderer_installed` share one ladder,
`pip_install_attempts()`: `--user`, then `--user --force-reinstall`, then
`--target get_package_dir()`. Best-effort, and the failure path names the
remedy.

Two rules the ladder exists to enforce:

- **Success is the import, never pip's exit code.** pip answers "Requirement
  already satisfied" from metadata on disk, so an orphaned `.dist-info` makes
  every install a silent no-op while the import keeps failing — and the next
  export tries again, forever. Re-probe the target interpreter after each
  attempt.
- **Never `--break-system-packages`.** It lifts PEP 668's guard for the whole
  environment; a plugin wanting two optional packages should not be making
  that promise. `--target get_package_dir()` writes into a directory KiForge
  owns and nothing else reads, so it cannot shadow a distribution package and
  uninstalling is deleting the folder. Every interpreter KiForge drives is
  told about it — `package_dir_path_snippet()` for a `-c` snippet,
  `add_package_dir_to_path()` in-process.

Install into **KiCad's interpreter**, not the running one. Inside the KiCad GUI
`sys.executable` is the application binary, not Python — use
`PathResolver.get_kicad_python_path()`, and probe that same interpreter when
deciding what is missing.

Anything that is a full desktop application is not a dependency. Inkscape was
dropped for this reason.

## 4. Assets ship with the plugin

Studio's icons live in `icons/` and are packaged to `plugins/icons/`. Do not
fetch UI assets at render time: KiCad's macOS Python is a python.org framework
build with **no CA store**, so every HTTPS request fails with
`CERTIFICATE_VERIFY_FAILED` and the UI silently renders blank. A plugin must not
need the network to draw itself.

When a network call is genuinely required, get the context from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alphaseneca/kiforge](https://github.com/alphaseneca/kiforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
