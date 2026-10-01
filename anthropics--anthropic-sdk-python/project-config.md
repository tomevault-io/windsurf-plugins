---
trigger: always_on
description: Context for contributors (human or AI) on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Working in this repository

Context for contributors (human or AI) on how this SDK is put together, plus the things reviews most often come back to. Setup instructions live in [CONTRIBUTING.md](CONTRIBUTING.md).

## Overview

The official Python SDK for the Claude API (`anthropic` on PyPI). This is the v1 line: it is built on `httpx2` and needs Python 3.10 or later (`requires-python` in `pyproject.toml`).

Most of it is generated from the OpenAPI spec. Hand-written helpers, tool runners, credentials, middleware, and platform clients sit on top.

### Who owns which files

- **`src/anthropic/lib/`, `examples/`, and `tests/lib/` are hand-written.** The generator never touches them.
- **Everything else comes from the generator, with custom code mixed in.** Generated files carry no marker comment. Hand-written methods, imports, and whole blocks sit inside many of them, including core modules such as `_client.py` and `_base_client.py`. So a file's location doesn't tell you who owns a given line. The next section says how to tell.
- **`.stats.yml` and `.github/workflows` are generator-owned too.** Avoid editing them, because the next codegen push rewrites them and a change tends to conflict.
- **`src/anthropic/_vendor/` is vendored.** Re-copy it from upstream rather than editing it.
- **Release-please writes `src/anthropic/_version.py`, `.release-please-manifest.json`, and `CHANGELOG.md`.** It rewrites them in each release PR, so a manual edit is overwritten or conflicts with it.

### Editing generated files

- **Check whether a line is generated before you change it.** Every commit the generator produced carries a `Stainless-Generated-From: <sha>` trailer, and a hand-written commit has none. The SHA is the generator's own output for that commit, with no custom code in it.
  - Find the latest one with `git log -1 --grep='Stainless-Generated-From' --format='%(trailers:key=Stainless-Generated-From,valueonly)'`.
  - Fetch it with `git fetch origin <sha>`, then diff a file against it: `git diff <sha> -- src/anthropic/resources/messages/messages.py`.
  - Lines that appear only on your side are custom code. If the line you want to change is in the generated commit, the fix probably belongs in the generator (see below).
- **Where the lines go matters, not how many.** Edits to generated files survive regeneration, because new generator output is merged with them, but the merge can conflict.
  - Changing lines the generator writes, such as a method's parameters or the way it builds the request, is likely to conflict the next time the spec changes.
  - Adding lines of your own, such as a new method or a block inside a method body, is usually fine, even a large one. `AsyncWork.poller` and `AsyncWork.worker` in `resources/beta/environments/work.py` are whole methods added after the generated ones, with their `lib/` imports inside the method body.
- **No provenance comments.** Don't add a comment that marks code as hand-written or mentions Stainless, and remove one from a block you are already editing.
- **Fix generator-owned behaviour upstream.**
  - Broken generated plumbing (retries, SSE parsing, serialization) is a generator bug.
  - A real response the generated types can't parse is a spec bug.
  - The model name in `tests/api_resources/` comes from the spec's `example`.

### Branches

- **PRs target the repository's default branch.** In the public repository that is `main`. Elsewhere don't assume its name. `gh repo view --json defaultBranchRef` shows it.
- **Don't merge into `next`.** `next` is a release branch in the public repository, and only automation writes to it: release-please runs there and merges `next` into `main` at a release. Don't open a PR against `next`, and don't merge or push to it by hand.
- **An unreleased API feature has its own branch.** Hand-written work for that feature targets the feature's branch instead of the default branch. Those branches get force-pushed, so rebase with `git rebase --onto <new-base> <old-base-sha>`.

## Build, test, lint

The entry points are `./scripts/{bootstrap,format,lint,test}`. Always use them to format, lint, and run the tests, and pass pytest arguments through `./scripts/test`. They run through `uv`, but call `uv` yourself only to add a dependency or to run an ad hoc script, never to format, lint, or test.

| Script | What it does |
| --- | --- |
| `./scripts/lint` | Runs ruff, a dependency-cap check, pyright in strict mode, mypy, and an `import anthropic` smoke test. |
| `./scripts/format` | Runs `ruff format` and `ruff check --fix`, and formats the code blocks in `README.md` and `api.md`. |
| `./scripts/test` | Runs the suite. Starts a Steady mock server on port 4010 unless `TEST_API_BASE_URL` is set. |

### Test matrix

- **Python and Pydantic versions.** `./scripts/test` runs the suite on the oldest and newest supported Python under Pydantic v2, and again under Pydantic v1 on the oldest only, because Pydantic v1 doesn't support the newest. On the newest it also runs the MCP tests against `mcp>=2`. Set `UV_PYTHON` to run one version.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthropics/anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
