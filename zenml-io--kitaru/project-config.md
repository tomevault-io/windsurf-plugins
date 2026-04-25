---
trigger: always_on
description: Kitaru is a **mixed Python + web repo** that produces three things: a Python SDK package, a documentation site, and a marketing landing page — all deployed together.
---

# Repository Guidelines

## Project Structure

Kitaru is a **mixed Python + web repo** that produces three things: a Python SDK package, a documentation site, and a marketing landing page — all deployed together.

```
src/kitaru/           # Python SDK package (src layout)
  cli.py              # CLI facade / console entrypoint (cyclopts)
  _cli/               # Internal command modules + shared CLI helpers
  adapters/           # Framework adapters (includes PydanticAI)
  mcp/                # MCP server tools (optional `kitaru[mcp]` extra)
tests/                # pytest tests
tests/mcp/            # MCP-specific tests (runs in `[mcp]` CI path)
examples/             # Runnable SDK examples
docs/                 # FumaDocs Next.js app — documentation at kitaru.ai/docs
  content/docs/       # Documentation content (MDX files)
  scripts/            # Node-side doc generation (convert-sdk-docs.mjs)
  app/                # Next.js app routes, layout, metadata
site/                 # Astro landing page + runtime shell at kitaru.ai/
  src/pages/api/      # Server-side API routes (e.g. /api/waitlist with KV)
scripts/              # Doc generation, site merge, and smoke test scripts
docker/               # Dockerfiles (Dockerfile = production server, Dockerfile.dev = dev/testing stack)
spec/                 # SDK design specifications (planning material, not shipped code)
design/               # Design docs, meeting notes (gitignored, never commit)
wrangler.toml         # Unified Cloudflare Worker deployment config
```

### Unified deployment model

The docs and landing page deploy as **one Cloudflare Worker**:

1. Python scripts generate docs content (`scripts/generate_cli_docs.py`, `scripts/generate_changelog_docs.py`, `scripts/generate_sdk_docs.py`)
2. `docs/` builds a static export into `docs/out/` (Next.js with `basePath: '/docs'`)
3. `site/` builds the Astro app into `site/dist/`
4. `scripts/merge_site.sh` copies `docs/out/*` into `site/dist/docs/`
5. Root `wrangler.toml` deploys the merged output as one Worker

Astro is the runtime shell because it owns the `/api/waitlist` endpoint (backed by Cloudflare KV). The docs are pure static files mounted under `/docs`.

## Build, Test, and Development Commands

Use `uv` for Python dependency management and `just` as the command stack.

### Python workflows

- `uv sync`: install and sync dependencies
- `uv sync --extra local`: install with local ZenML runtime components
- `uv run kitaru init`: **required in a fresh `git worktree`.** Creates the `.kitaru/` project marker that ZenML's dynamic pipeline resolver needs to re-import example modules by dotted path. Without it, ~14 `test_phase*_example::*_runs_end_to_end` tests fail with `RuntimeError: Unable to resolve dynamic pipeline source`. Worktrees do not inherit `.kitaru/` from the main checkout.

**Core dev loop — these three commands handle the vast majority of development needs:**

| Command | What it does | When to run |
|---|---|---|
| **`just check`** | All checks: format, lint, typecheck, typos, yaml, links | After every chunk of work, before commit/push |
| **`just fix`** | Auto-fixes formatting, lint issues, yaml | When `just check` reports fixable issues — resolves most linting problems automatically |
| **`just test`** | Full pytest suite | After code changes, before commit/push |

**Typical loop:** write code → `just fix` → `just check` → `just test` → commit.

- `just test tests/test_file.py::test_name`: run one test
- `just lint`: lint only
- `just typecheck`: type check only
- `just typos`: typo check only
- `just format-check`: check formatting without modifying
- `just yaml-check`: check YAML formatting
- `just links`: check markdown links offline (requires `lychee`: `brew install lychee`)
- `just links-external`: check links including external URLs (slow)
- `just build`: build wheel and sdist locally

### Docs/site workflows

These require Node 22+ and pnpm.

- `just generate-docs`: generate CLI reference + changelog + SDK reference docs
- `just docs`: preview docs locally (dev server at localhost:3000)
- `just docs-build`: build docs static export
- `just site`: preview landing page locally (dev server at localhost:4321)
- `just site-build-only`: build landing page only (no docs merge)
- `just site-build`: full unified build (generate docs, build docs, build site, merge)

## Coding Style & Naming Conventions

- Follow US English spelling in code and docs (`initialize`, `serialize`, `color`).
- Use type hints on all public functions and return values.
- Prefer modern annotations (`list[str]`, `str | None`) over legacy `typing` aliases.
- **Do not use `from __future__ import annotations` in files that define Kitaru `@flow`/`@checkpoint` functions or ZenML `@pipeline`/`@step` functions.** ZenML inspects step output annotations at runtime and currently rejects postponed/string annotations such as `"dict[str, Any]"`. Use real runtime annotations instead; Python 3.11+ supports `list[str]` / `str | None` without the future import.
- Follow Google Python style for docstrings.
- Keep comments focused on *why* (intent/trade-offs), not line-by-line narration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zenml-io/kitaru](https://github.com/zenml-io/kitaru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
