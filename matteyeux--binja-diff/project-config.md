---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# CLAUDE.md

Guidance for AI agents working in this repository.

## What this repository is

`binja_diff/` is a Binary Ninja UI plugin that diffs two binaries side by side
using QBinDiff for function matching. Everything else in the repository is a
vendored read-only reference checkout:

| Directory | Status |
| --- | --- |
| `binja_diff/` | The plugin. This is the only thing to edit. |
| `qbindiff/` | Upstream QBinDiff source, for reference. Do not modify. |
| `quokka/` | Upstream Quokka source, for reference. Not used at runtime. |
| `binaryninja-api/` | Binary Ninja API source and headers, for reference. Do not modify. |
| `.venv-qbindiff/` | Virtualenv used by the test suite. Not committed. |

Read the vendored trees freely; they are the authoritative source for API
signatures, which is important because much of the Binary Ninja Python API is
generated at build time and is not introspectable from a checkout.

## Environment

QBinDiff lives in a virtualenv rather than Binary Ninja's bundled interpreter,
because it pins numpy >= 2 and pulls in scipy and scikit-learn. Use that
interpreter for anything touching qbindiff:

```bash
.venv-qbindiff-312/bin/python ...
```

Two failure modes are worth recognizing on sight, because both look like
"QBinDiff is not installed" and neither is:

- **Python version mismatch.** A site-packages built for a different minor
  version is silently unimportable. It must match whatever Binary Ninja runs
  (`import sys; sys.version` in its console).
- **Namespace shadowing.** A `qbindiff` directory sits at the repository root.
  If the repo root lands on `sys.path` it shadows the installed package as a
  namespace package: `import qbindiff` succeeds and every submodule fails.

`dependency_error()` in `binja_diff/__init__.py` distinguishes these and names
the cause; extend it rather than reintroducing a generic message.

QBinDiff also imports `python-magic`, which dlopens the system `libmagic`. On a
minimal Linux image that library is often absent and surfaces as
`ImportError: failed to find libmagic` at plugin startup.

## Running the tests

```bash
.venv-qbindiff-312/bin/python binja_diff/tests/run_all.py
```

There are two tiers.

`test_live.py` needs a **headless** licence, which the Personal edition does
not grant: it prints SKIP on a machine that only has the GUI. A headless host
is worth arranging before touching classification — every bug in `align.py`
that survived more than one round did so because it could only be reproduced
against real Binary Ninja rendering, and the stub encoded what the author
assumed the tokens looked like rather than what they are.

`test_backend.py`, `test_align.py`, `test_engine.py`, `test_persist.py`,
`test_symbols.py`, `test_scope.py` and `test_cli.py` install a stub
`binaryninja` module into `sys.modules` and run the **real** QBinDiff against
it, including a full end-to-end diff with belief propagation. They cover the
backend's object model, instruction and operand extraction, block and line
alignment, `DiffTask`'s completion, cancellation and failure paths, the
saved-diff format, what porting symbols refuses to do, container scoping, and
the CLI's argument contract and report. These need no Binary Ninja
installation.

`test_live.py` drives the **real** Binary Ninja API over real binaries. It
prints SKIP and exits 0 wherever Binary Ninja is not importable. It is the only
test that validates against genuine analysis output, so run it after touching
`core/`. Pass two paths to pick the binaries (`test_live.py /bin/ls /bin/cat`);
it defaults to `/bin/true` against `/bin/false`. `run_all.py --no-live` skips
it.

Building a pair of binaries that differ only in constants is the sharpest test
of the diff classifier, and cheap:

```bash
printf 'int check(int x){return x>42?x*3:x+7;}\nint main(){return check(10);}\n' > /tmp/a.c
sed 's/42/99/' /tmp/a.c > /tmp/b.c
gcc -O0 -o /tmp/a /tmp/a.c && gcc -O0 -o /tmp/b /tmp/b.c
.venv-qbindiff-312/bin/python binja_diff/tests/test_live.py /tmp/a /tmp/b
```

Two things to know about the stubbed harness:

- `binja_diff/__init__.py` imports `binaryninja` at module scope, so the stub
  must be registered before any `binja_diff` import. That is what
  `tests/bootstrap.py` is for; load it by path, not through the package.
- The stub only models the API surface the plugin actually uses. When you touch
  a new Binary Ninja API, extend `tests/stub_binaryninja.py` to match, and
  check the real signature against `binaryninja-api/` first.

Also lint and type-check before finishing:

```bash
ruff check . && ruff format --check . && ty check
```

Those are the same three commands `.pre-commit-config.yaml` runs, reading
`ruff.toml` and `ty.toml` from the repo root, so a green run here is a green
commit. Both configs exclude the vendored trees.

Two settings are load-bearing rather than taste, and re-enabling either breaks
something that no test will catch:

- **isort (`I`) is off in ruff.** Sorting the imports under `ui/` puts `PySide6`
  before `binaryninjaui`, which takes the process down (see below).
- **`unresolved-import` is off in ty.** Nothing the plugin imports resolves from
  a bare checkout — the Binary Ninja modules live in the app bundle, qbindiff is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matteyeux/binja-diff](https://github.com/matteyeux/binja-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
