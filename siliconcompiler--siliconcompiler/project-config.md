---
trigger: always_on
description: Orientation for coding agents working in this repository, and for anyone writing
---

# AGENTS.md

Orientation for coding agents working in this repository, and for anyone writing
SiliconCompiler build scripts. It is deliberately the same file humans read: a
separate AI-only document would rot.

Read this before generating SiliconCompiler code. Most of it is about avoiding
one specific failure -- confidently emitting an API that was removed in 2025 and
still dominates the training data, forum answers and search results.

## What SiliconCompiler is

A modular hardware build system -- "make for silicon". It compiles RTL to GDSII
(ASIC) or a bitstream (FPGA) by driving pluggable flows over open-source and
commercial EDA tools. Everything is configuration in a single, versioned schema;
the Python API is a typed surface over that schema.

## The API, in one working example

This is the current idiom. It is
[`examples/heartbeat/heartbeat.py`](examples/heartbeat/heartbeat.py), which runs.

```python
from siliconcompiler import ASIC, Design
from siliconcompiler.targets import skywater130_demo

design = Design("heartbeat")                        # what to build
design.set_dataroot("heartbeat", __file__)          # where its files are rooted
design.set_topmodule("heartbeat", fileset="rtl")
design.add_file("heartbeat.v", dataroot="heartbeat", fileset="rtl")
design.add_file("heartbeat.sdc", dataroot="heartbeat", fileset="sdc")

project = ASIC(design)                              # how to build it
project.add_fileset(["rtl", "sdc"])                 # which filesets to compile
skywater130_demo(project)                           # PDK, libraries, flow
project.run()
project.summary()
```

The split matters: a **`Design`** describes source code and is reusable across
builds; a **project** describes one compilation of it.

**Use one of the named project classes -- `ASIC`, `FPGA`, `Lint`, `Sim` -- not
the bare `Project`.** Choosing the class is how you say what kind of build this
is, and each named class brings the schema, constraints and metrics for its
domain: `ASIC` carries the `asic,*` parameters and floorplan constraints, `Lint`
carries neither and needs no PDK. `Project` is the base class they extend. It is
the right choice only when writing code that must work across project types, and
the wrong choice for a build script -- a bare `Project` has no domain schema, so
the target and flow you want will not fit it.

Top-level exports, in full: `Design`, `Project`, `ASIC`, `FPGA`, `Lint`, `Sim`,
`PDK`, `StdCellLibrary`, `FPGADevice`, `Flowgraph`, `Checklist`, `Task`,
`TaskSkip`, `OpenTask`, `ShowTask`, `ScreenshotTask`, `NodeStatus`, `sc_open`,
`__version__`.

## Five things generated code gets wrong

**1. `Chip` does not exist.** `Chip('design')`, `chip.set(...)`,
`chip.use(...)`, `chip.input(...)`, `chip.register_source(...)` and
`chip.load_target(...)` were removed in **v0.35.0** (October 2025) and replaced by
`Design` + `Project`. If you are about to write `Chip`, you are writing the old
API. See [Migrating from the Chip API](docs/user_guide/migration.rst) for the
old-to-new table.

**2. There is no `sc` command.** The entry points are `sc-dashboard`, `sc-issue`,
`sc-remote`, `sc-server`, `sc-show`, `sc-install` and `smake` -- that is the whole
list, from `[project.scripts]` in `pyproject.toml`. A bare `sc -target ...`
invocation is from the pre-0.35 CLI and will fail with `command not found`. To
run something from the shell, use a Python script, `smake`, or the demos:

```sh
python3 -m siliconcompiler.demos.asic_demo      # ASIC, remote by default
python3 -m siliconcompiler.demos.fpga_demo      # FPGA
```

**3. Prefer typed accessors over raw keypaths.** `project.option.add_fileset('rtl')`
and `project.get('option', 'fileset')` reach the same stored value, but the
accessor is the supported, self-documenting interface and is what the tutorials
and `examples/` use. Reach for a keypath only when no accessor exists -- most
often reading metrics and records, which are keyed per flowgraph node:

```python
project.get('metric', 'cellarea', step='synthesis', index='0')
```

Do not use a keypath for a parameter that has an accessor.

**4. Files go into filesets, not a flat list.** A `fileset` is a named group of
files with a role -- `rtl`, `sdc`, `xdc`, `testbench`. `Design.add_file` puts a
file in one; `Project.add_fileset` selects which ones this compilation uses. A
design can carry filesets it does not compile every time, which is the point.

**5. Paths are rooted at a `dataroot`, not the current directory.**
`design.set_dataroot("name", __file__)` anchors a design's files to the script
that defines them, so it works regardless of where it is run from. An
environment-variable dataroot (`set_dataroot("foundry", "$FOUNDRY_ROOT/...")`)
is how foundry data is referenced without committing it.

## Where new code goes

Answer this before writing a module -- the wrong destination is the most common
reason a contribution has to be restarted. Full table with links:
[docs/development_guide/contribution.rst](docs/development_guide/contribution.rst).

| What you have | Where it goes |
|---|---|
| Open-source PDK or standard cell library | the separate [`lambdapdk`](https://github.com/siliconcompiler/lambdapdk) package -- **not** this repo |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siliconcompiler/siliconcompiler](https://github.com/siliconcompiler/siliconcompiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
