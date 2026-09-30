---
trigger: always_on
description: generates an Apache configuration for one application and runs
---

# Agent guidance for mod_wsgi

## Project

mod_wsgi is an Apache HTTP Server module, written in C, that hosts
Python WSGI applications, either embedded in the Apache child
processes or in separate daemon processes. It sits across two C APIs
at once, those of Apache (with APR) and of CPython, which makes the
code a specialised place to work. See README.rst for what users are
told, and docs/ for the documentation published at
https://www.modwsgi.org/.

Layout:

- src/server/ holds the C source of the module. mod_wsgi.c is the
  entry point and the rest is split by area into wsgi_*.c files with
  matching headers.

- src/express/ holds the Python source of `mod_wsgi-express`, which
  generates an Apache configuration for one application and runs
  Apache with it.

- telemetry/ is a separate Python package, `mod_wsgi-telemetry`, with
  its own pyproject.toml, uv.lock, source, tests and release cycle. It
  shares the repository and nothing else.

- docs/ is the Sphinx source of the documentation, with one page per
  directive in docs/configuration-directives/, guides in
  docs/user-guides/, and one file per version in docs/release-notes/.

- tests/ and scripts/ hold the tests and the scripts that run them.
  See TESTING.md for where tests are, how to run them, and the
  conventions for adding new ones. Read it before doing any test
  related work.

There are two ways of building. The classic `./configure` and
Makefile.in install the module straight into an Apache installation.
setup.py builds the PyPI packages. Three packages come out of the
repository: `mod_wsgi`, `mod_wsgi-standalone` (the same source, built
by package.sh with a dependency on the separate `mod_wsgi-httpd`
package), and `mod_wsgi-telemetry`.

In this repository "docs" means the docs/ directory specifically. The
README files in the tree, including telemetry/README.md, are
developer facing source files and are kept up to date as the code
changes, even when told to hold off on changing the docs.

The scratch/ directory holds temporary working files, such as
reference material given to an agent or plans an agent is asked to
generate. It is ignored by git. Its contents come and go, so never
reference scratch/ files by name from code or documentation that
will be committed.

## Tooling

- Python environments are managed with [uv](https://docs.astral.sh/uv/).
  The development environment is `.venv` in the root of the
  repository, and the test scripts depend on it being there.

- The Justfile has targets for the common tasks. Prefer them over
  synthesizing the underlying commands, and run `just --list` to see
  them all. The main ones are `just build`, `just test`,
  `just test-bounds`, `just test-telemetry`, `just docs` and
  `just aplogno`.

- After editing anything under src/server/, rebuild with `just build`
  before testing. It runs `uv pip install -e . --no-cache`. Without
  `--no-cache` a previously built wheel can be reused and the edit is
  silently not picked up. An editable install alone does not
  recompile the extension.

- `just clean` and `just test-versions` delete `.venv`, and
  `just test-python`, `just test-bounds` and `just venv` with a
  version replace it. Anything else that had been installed into it
  is lost, so look before running them.

- telemetry/ is its own uv project. Work on it from inside that
  directory with `uv sync` and `uv run`.

## Versions and releases

- The mod_wsgi version is defined in src/server/wsgi_version.h, as
  three numeric defines and a version string. Keep all four
  consistent. setup.py, docs/conf.py and the release workflow all
  read the version string from that file.

- The mod_wsgi-telemetry version is `__version__` in
  `telemetry/src/mod_wsgi/telemetry/__init__.py`.

- Versions under development carry a suffix on the string in the
  forms `6.1.0.dev1` and `6.1.0rc1`. The numeric defines hold the
  version being worked towards.

- The exact version of `mod_wsgi-httpd` required by
  `mod_wsgi-standalone` is set in one place, the `install_requires`
  line of setup.py. package.sh reads it from there when generating
  the pyproject.toml for the standalone package, so keep the form of
  that line intact.

- Every mod_wsgi version has a file in docs/release-notes/, listed in
  docs/release-notes.rst with the newest first. A change that users
  can observe needs an entry in the file for the version under
  development, under one of the headings already in use: New
  Features, Features Changed, Features Removed, Bugs Fixed.

## C source

- Match the formatting of the surrounding code, and do not reformat
  code that is not otherwise being changed.

- Every source and header file includes `wsgi_python.h` and then
  `wsgi_apache.h` before any other header. `Python.h` has to be the
  first header every compilation unit sees, or structure layouts can
  differ between files on some platforms.

- For any multi-line comment, put `/*` on a line of its own and `*/`
  on a line of its own. Do not start the text on the `/*` line or end
  it with `*/` on the last line of text. Leave a blank line after
  such a comment when it comes before a function, or before the group
  of struct members it describes. Single line comments are not
  affected.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GrahamDumpleton/mod_wsgi](https://github.com/GrahamDumpleton/mod_wsgi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
