---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## What this is

Sphinx-Gallery is a Sphinx extension that turns directories of Python example scripts into
rendered galleries: it executes each example, captures figures and output, and generates
reStructuredText into the user's **source** directory, which Sphinx then reads like any other
document. See @doc/contribute.rst for the human contributor guide.

## Setup

```bash
pip install -e ".[dev]"     # do not skip: see below
pre-commit install --install-hooks
```

The `dev` extra matters more than usual. `sphinx_gallery/tests/test_full.py` — the tinybuild
integration suite, and most of the interesting coverage — skips at **module level** unless
`jupyterlite_sphinx`, `joblib`, and `matplotlib.animation` with a working ffmpeg writer are all
importable. Without them the whole file silently disappears and a green run means almost
nothing, so check the skip count rather than just the pass count. `sphinxcontrib-video` is a
separate trap: tinybuild requests mp4 animations, so its absence surfaces as a `ConfigError`
during fixture setup instead of a skip.

## Commands

```bash
pytest sphinx_gallery                                  # full suite, ~2.5 min
pytest sphinx_gallery/tests/test_full.py -k rebuild    # single test / keyword
pre-commit run -a                                      # the actual lint gate
make -C doc html                                       # build SG's own docs
```

`pre-commit` is the source of truth for linting. It runs `ruff-format` and `ruff` with
**`--select=I` only** (import sorting), plus codespell, yamllint, sphinx-lint, and `ty`.
Running bare `ruff check sphinx_gallery/` reports a dozen pre-existing violations of rules the
project does not enforce — do not "fix" them. `ty check sphinx_gallery/` has one known
pre-existing diagnostic (`scrapers.py`, `anim._fig`).

## Generated files and incremental builds

Most subtle bugs in this codebase live here. Sphinx decides a document is stale by comparing
**mtimes**, so anything written into the source directory is part of the build's caching
contract.

- **Write through `_replace_md5`, never directly.** It writes `<name>.new`, compares, and keeps
  the original file (and its mtime) when the content is unchanged. Writing unconditionally
  makes Sphinx re-read and re-render that page on every build, forever.
- **Generated content must be deterministic.** Sets and dicts-built-from-sets iterate in a
  different order in every process, so a file generated from one can differ between two builds
  that changed nothing. Sort anything set-derived before rendering, with a *total* order —
  `sorted(x, key=len)` leaves ties in arbitrary order.
- **Do not delete generated files at `build-finished`.** Directives such as `graphviz` register
  their input via `env.note_dependency()`, and a *missing* dependency marks the page changed on
  the next build. Leave the file and let `_replace_md5` keep it stable; add it to
  `.gitignore` instead.
- **Cross-document state belongs on `env`, not on `gallery_conf`.** Config mutations made
  during the read phase do not survive: under `-j`, `read_doc` runs in forked workers and only
  data merged back via `env-merge-info` reaches the main process. State on `env` is pickled
  between builds, so it also needs `env-purge-doc` to drop entries for a re-read document.
  `write_api_entries` / `purge_api_entries` / `merge_api_entries` in `gen_gallery.py` are the
  worked example.
- **The example-skipping path must restore whatever a run produced.** `generate_file_rst`
  returns early when an example's md5 is unchanged. Anything derived from actually running it
  — backreferences, execution cost — has to be read back from its cache on that path
  (`_read_cached_backrefs`, `_read_cached_cost`), or every rebuild silently degrades the
  generated output rather than failing.
- A page whose content depends on autodoc cannot be filled in at `source-read` without
  ordering and parallelism hazards; `env-updated` is the first point where every
  `autodoc-process-docstring` has fired and worker environments have been merged.

## tinybuild

`sphinx_gallery/tests/tinybuild/` is a miniature project built by the `sphinx_app` fixture and
asserted against throughout `test_full.py`. When a bug shows up in a downstream project
(MNE-Python, scikit-learn, …), the convention is to extend tinybuild so the suite reproduces it.

- The `sphinx_app` fixture is **module-scoped and mutated** by `test_rebuild`, which rebuilds
  and edits examples in the shared source directory. Later tests read that mutated tree, so one
  `test_rebuild` failure can cascade into unrelated-looking failures (e.g. `test_error_messages`).
  Check whether an earlier test failed before investigating a later one.
- `examples_dirs` points **outside** `doc/`, so a test that copies the tree must copy the
  tinybuild root, not `sphinx_app.srcdir`.
- Several assertions hard-code counts: `N_RST` / `N_OTHER` at the top of `test_full.py`, and
  `n_documented` in `test_backreferences_examples_html`. Adding a module to the autosummary in
  `tinybuild/doc/index.rst`, or a class to an autodoc'd module, will break them. Prefer adding a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sphinx-gallery/sphinx-gallery](https://github.com/sphinx-gallery/sphinx-gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
