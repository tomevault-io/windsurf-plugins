---
trigger: always_on
description: This repository is the open-source Reactor Runtime, published under Apache 2.0.
---

# Agent instructions for reactor-runtime

This repository is the open-source Reactor Runtime, published under Apache 2.0.
Everything in it — code, comments, docs, commit messages, PR descriptions — is
public. These rules are non-negotiable.

## This repo is public: no private-system references

Treat any private, internal infrastructure or non-public services as if they
do not exist. Do not name them, link to them, depend on them, or write code
that assumes them. Rely only on publicly available tooling and artifacts. The
audience is an outside developer with no insider context.

In scope: this package's own APIs, its protocol, and the public-facing product
surface. If a piece of functionality depends on something private, it does not
belong in this repo.

Bad: "The Redis runner connects to the Coordinator over gRPC."
Good: "Production runners are distributed separately as packages built on top
of this runtime."

## README

`README.md` is the project's front page for an outside developer. It keeps
this structure, in this order: banner, one-line pitch with doc links,
introduction, Highlights, How it works, Install, Learn more, Development,
License. Edits refine sections in place; do not reorder, drop, or duplicate
them.

- Install is through the `reactor` CLI and Docker only. Never add a
  `pip install reactor-runtime` path or any other direct-package install;
  the package is consumed inside the image the CLI builds.
- Documentation links point at the public docs site, https://docs.reactor.inc —
  the model-authoring (runtime) pages under `https://docs.reactor.inc/deploy`.
- The README must not fall behind the code. A PR that changes the authoring
  surface, the CLI workflow, or what the project does updates `README.md` in
  the same PR, and review flags a PR that changes behaviour the README
  describes without touching it.

## Examples

`examples/` holds three workspaces, and each shows one shape on purpose.
`starter` is the smallest complete model: one class that writes `generate()`
alone, with the runtime's default `process_input()` and `process_output()`.
It stays that way; a change that adds either hook, or splits it into two
files, removes the example's reason to exist. `echo` and `waypoint` carry the
application/model split, and they are where that pattern is taught. Its rules
live in `skills/application-model-isolation`, which says the split is
optional and when to make it.

## Toolchain

`mise` is the task runner and pins the toolchain (uv, ruff, ty, lefthook, buf,
and nox) from `mise.lock`. Every command below is a mise task; a thin `make`
shim forwards the same names, so `make lint` runs `mise run lint`. `mise run
install` sets up a clone: deps, wire bindings, and git hooks.

```sh
mise run install      # deps + wire bindings + git hooks (first-time setup)
mise run lint         # ruff check, ruff format --check, and mise.lock drift
mise run format       # apply ruff formatting
mise run typecheck    # ty (strict)
mise run test         # unit + contract tests on the floor Python
mise run test-matrix  # unit + contract tests on every supported Python
mise run test-integration         # integration tests on the floor Python
mise run test-integration-matrix  # integration tests on every supported Python
```

- Python is owned by uv, not mise: `requires-python` in `pyproject.toml`
  (`>=3.12`) is the sole floor. Write modern syntax: `X | None`, builtin
  generics, `type` statements where they help. Never `Optional[...]`/`Dict[...]`.
- nox owns the supported-Python matrix (`noxfile.py`): each version is its own
  session, `mise run test-matrix` runs them all, and CI declares the same list
  in its workflow matrix.
- ty type-checks strictly. Do not weaken it with blanket ignores; a targeted
  `# type: ignore[ty:code]` needs a reason the reader can verify.
- Releases are version-driven. Merging a `uv version X.Y.Z` bump to main tags
  `vX.Y.Z`, publishes the GitHub release, and uploads to PyPI. Every other merge
  publishes an `X.Y.Z.dev<run>` pre-release on GitHub alone. CI writes the tag,
  so never tag by hand, and never edit the version field in an unrelated change.

## Tests

Every PR that adds or changes behaviour ships tests in the same PR. Unit tests
live in `tests/unit/`, integration tests in `tests/integration/`, mirroring
the `src/reactor_runtime/` module layout. A failing test must surface its full
traceback — never wrap test execution in machinery that swallows exceptions
from worker threads or event loops.

## Lints

`mise run lint` (ruff check, ruff format --check, and the mise.lock drift check)
and `mise run typecheck` (ty) must pass before a PR is opened. Do not disable
rules file-wide to silence a finding; fix the code or narrowly suppress with a
justification.

## Docstrings

Google style, enforced by ruff's pydocstyle rules. Docstrings document the
contract — what the caller can rely on — not the implementation.

- Every public module, class, and function has a docstring. Private helpers
  need one only when the name cannot carry the intent.
- One-line summary in the imperative ("Return the...", not "Returns the...").
  Add `Args:` / `Returns:` / `Raises:` sections when they say something the
  signature does not.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactor-team/reactor-runtime](https://github.com/reactor-team/reactor-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
