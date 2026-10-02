---
trigger: always_on
description: SPDX-FileCopyrightText: Amutable GmbH <https://amutable.com/>
---

<!--
SPDX-FileCopyrightText: Amutable GmbH <https://amutable.com/>
SPDX-License-Identifier: MPL-2.0
-->

# AGENTS.md

## General

- **Never touch `buck-out/` directly**: no `rm`, `mv`, or edits. To inspect an output, get its path from
  `buck build --show-output`/`--out` and read it read-only.
- **Never `buck build | tail`**: redirect the full output to a log file, print the path so the user can
  follow along, and check the exit status separately. `tine buck log what-ran --failed --show-std-err`
  prints the full stderr of the actions that failed in the last build, which the build output truncates.
- Keep `buck run tine//tools:check` green.
- Start every new file with the SPDX header, in the file's comment syntax, after a shebang if there is
  one: `SPDX-FileCopyrightText: Amutable GmbH <https://amutable.com/>` and `SPDX-License-Identifier:
  MPL-2.0`. A file that cannot carry one, such as data, is listed in `REUSE.toml` instead; `check` runs
  `reuse` and fails on a file with neither.
- Line break documents and plans at 109 columns.
- Comments explain *why*, not *what*; say each thing once.
- Do not write user-facing documentation. At most, add a TODO comment where documentation should be
  written by a human.

## Layout

Cell layout and architecture: [the design plan](docs/design/architecture.md). User guides are under
`docs/user/`, design documents under `docs/design/`.

- A unit test is `<driver>_test.py` beside its driver, run by a `box_python_test` in the same package.
- An assertion about what a build produced is a `box_sh_test` beside the target that produced it, taking
  artifacts as `$(location)` arguments so it runs against pinned tools rather than the host's.
- Python sources reach a suite in another package through `deps` on a `tine_python_library`, never
  `export_file`.
- Python drivers use `tine_python_library`/`tine_python_binary`. Never declare the prelude's bootstrap
  target or a `-ty` check by hand; the macros do both.
- A driver importing third-party modules lists the `boxes` its check resolves against, one check per box.
  `typecheck = False` is for vendored sources only.

## Commands

**Every `buck` below means `tine buck`**: nothing else on this machine is the Buck2 this project pins.
`mise.toml` puts `./bin` and `./tools` on `PATH`; every pinned tool is declared in `tools/tools.json`.
`tine buck` runs Buck with the mounts declared by `tine mount` in `.buck/tine-mounts.toml`.

- **Build everything:** `buck build tine//...`, which runs no tests.
- **Lint (format, lint, type-check, whole-graph analysis):** `buck run tine//tools:lint`
- **Check (lint plus the fast test suites):** `buck run tine//tools:check`
- **Auto-format + auto-fix:** `buck run tine//tools:fmt`
- **Analyze part of the graph without building it:** `buck bxl tine//tools/graph.bxl:analyze -- --pattern
  <pattern>`; `lint` already runs it over everything.
- **Tests:** `buck test tine//...`, or one suite: `buck test tine//image:test`. Anything needing an example
  image carries the `image` label, the boot smokes additionally `vm`; `--exclude image` is the
  seconds-long half that `check` runs, `--include image` the rest, which CI runs as its own step.
- **Catalog lock:** `buck run tine//tools:refresh-catalog`; `tools:verify-catalog` asserts the committed
  lock matches.
- **Example image digests:** `buck run tine//examples/image:refresh-expected`, then commit
  `examples/image/expected.json` with the change that moved it. Anything that can alter what the example
  images build needs this: a package they install, an image or UKI rule, a driver that writes into a root.
  `examples/image:reproducibility-test` asserts the committed digests, and it carries the `image` label, so
  `check` does not run it and only CI's image step catches a stale file.
- **Full CI pipeline:** `tools/ci.sh`, fail-fast. Run individual groups by stating their names, e.g.
  `tools/ci.sh check build`
- **rpm importer:** `buck run tine//tools:importer -- <verb>`; see [importer.md](docs/user/importer.md).
- **BuildRequires cycle analysis:** `buck run tine//tools:scc -- <branch label>`; background in
  [self-host-approaches.md](docs/design/self-host-approaches.md).

## Starlark

`starlark_fmt` owns the layout of every `.bzl` and `BUCK` file and wraps at 160 columns.

- Leave a trailing comma after the last argument to keep a signature or call broken across lines.
- It sorts dict keys unconditionally; mark a dict whose order is a contract (`disk.bzl:partition` feeds
  partition UUIDs) `# @unsorted-dict-items`.
- It drops load symbols it cannot see used; annotate type-only imports `# @unused`.

## Macros and rules

A macro forwards arguments it does not read as `**kwargs`; one it *reads* is a typed parameter, never a
`kwargs` lookup, and its helpers take those values as parameters rather than the whole dict. Such a
parameter defaults to `None`, which Buck reads as the attribute's own default, and is annotated
`X | Select` if a caller may reach it with a `select()`.

`ctx.attrs` belongs to the rule implementation that owns those attributes. `_impl` reads whatever it likes
from it; every other function takes what it needs as typed parameters.

## Workspace

- Use cell-relative `tine//...` labels for same-cell targets, never `root//...`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amutable-systems/tine](https://github.com/amutable-systems/tine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
