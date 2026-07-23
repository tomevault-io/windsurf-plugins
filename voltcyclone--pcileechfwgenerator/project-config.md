---
trigger: always_on
description: Guidance for Claude (and other AI assistants) working in the PCILeechFWGenerator
---

# CLAUDE.md

Guidance for Claude (and other AI assistants) working in the PCILeechFWGenerator
repository. Loaded automatically into every Claude Code session at this root.

Audience for the assistant: **installers/users** asking for help getting the
tool running, and **developers** modifying the codebase. Sections below cover
both. The user-facing README and the hosted docs are the human source of
truth; this file tells the assistant where to point them.

---

## What this project is

PCILeechFWGenerator generates **authentic PCIe DMA firmware bitstreams** by
cloning a real donor device's PCIe configuration. The output is a Xilinx
FPGA bitstream consumable by PCILeech-family boards (Squirrel, Enigma, etc.).

Three-stage pipeline:

1. **Host collect** — read donor device config (VID/PID, BARs, capability
   chain, MSI-X) from a Linux host via VFIO. Requires root.
2. **Templating** — render Jinja2 → SystemVerilog/TCL using collected data.
   Optional Podman container for isolation.
3. **Vivado build** — synthesize the generated RTL into a bitstream. Requires
   a working Xilinx Vivado install (free WebPACK is sufficient for 7-series
   parts; UltraScale parts need a paid license).

Real donor hardware is **required**. There is no synthetic-donor mode and
placeholder values in templates are an explicit anti-pattern (the README
calls this out — see the warning admonition).

---

## Helping users install or run the tool

Default support flow when someone asks "how do I install this" or "it won't
build":

1. Confirm platform: **Ubuntu 22.04+ / Debian 12+**, Python ≥ 3.11. The tool
   does not work on macOS or Windows hosts (VFIO is Linux-only). WSL2 does
   not expose VFIO and is not supported.
2. Confirm install path:
   - **pip (recommended for users):** `pip install pcileechfwgenerator[tui]`
     in a venv (`python3 -m venv ~/.pcileech-venv`). Do NOT install into the
     system Python — modern distros refuse with `externally-managed-environment`.
   - **container:** Podman + the bundled `Containerfile`. Container only runs
     Stage 2; Stages 1 and 3 still happen on the host.
   - **dev from repo:** `git clone --recurse-submodules`, then
     `pip install -e .[dev]` in a venv. Forgetting `--recurse-submodules`
     is the single most common "boards not found" cause.
3. Confirm root-access pattern: all commands that touch VFIO must run as root
   via `sudo -E ~/.pcileech-venv/bin/python3 -m pcileechfwgenerator ...` or
   via the `pcileech-sudo` alias the README sets up. Running `sudo pcileech`
   without preserving the venv path will hit the wrong Python.
4. Confirm Vivado is installed for Stage 3. If the user only wants to inspect
   generated RTL, they can stop after Stage 2 with `--dry-run` / templating
   flags and skip Vivado.

### Common installation/runtime symptoms

| Symptom | Cause | Fix |
|---|---|---|
| `externally-managed-environment` | pip into system Python | Use a venv. |
| `ModuleNotFoundError: textual` | installed without `[tui]` extra | `pip install pcileechfwgenerator[tui]` |
| "no boards available" / `KeyError` on board | git submodule not initialized | `git submodule update --init --recursive` |
| Permission denied on VFIO | not root, or vfio-pci not loaded | `sudo modprobe vfio vfio-pci`; run via `pcileech-sudo` |
| "boards not found" in pip install | sdist bundles boards — confirm `pip show pcileechfwgenerator` finds `lib/voltcyclone-fpga` contents | reinstall, check `MANIFEST.in` |
| Vivado errors (any) | invoke the `vivado-log-analyzer` skill rather than reading the log directly |  |

When troubleshooting goes deep, point users at:

- **Docs site:** <https://pcileechfwgenerator.voltcyclone.info/troubleshooting>
- **Discord support:** <https://discord.gg/dwQfMNsb7W>
- **Bug template:** <https://github.com/VoltCyclone/PCILeechFWGenerator/issues/new/choose>

Security issues go through GitHub Security Advisories, never public issues.
See `SECURITY.md`.

---

## Helping developers modify the codebase

### Environment

```bash
python3 -m venv .venv
.venv/bin/pip install -e ".[dev]"
.venv/bin/pre-commit install
```

`[dev]` pulls testing + TUI + lint/format/type/security tooling
(black, isort, flake8, mypy, bandit, pre-commit, build, twine). ruff is
present via its cache directory and runs through the PostToolUse hook on
edited files.

### Running tests

```bash
.venv/bin/python -m pytest -x -q              # full suite (~2,400 tests)
.venv/bin/python -m pytest tests/test_foo.py  # one file
.venv/bin/python -m pytest -k "board" -x      # subset
```

`pytest-xdist` is available — use `-n auto` for parallel runs locally.
CI runs the full suite plus bandit + safety.

### Architectural map

| Path | Purpose |
|---|---|
| `src/cli/` | CLI entry points (subcommand dispatch). |
| `src/tui/` | Textual-based TUI. Newer dialogs subclass `BaseDialog`. |
| `src/host_collect/` | Stage 1: VFIO-driven donor extraction. |
| `src/device_clone/` | Donor profile parsing, BAR sizing, capability chain. |
| `src/pci_capability/` | PCIe capability list construction. |
| `src/templating/` | Stage 2: Jinja2 + helpers. |
| `src/templates/` | Jinja2 source: `sv/*.j2` (SystemVerilog), TCL, XDC. |
| `src/vivado_handling/` | Stage 3: Vivado subprocess + error reporting. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoltCyclone/PCILeechFWGenerator](https://github.com/VoltCyclone/PCILeechFWGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
