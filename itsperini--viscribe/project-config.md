---
trigger: always_on
description: This is the canonical agent guide for the Viscribe repository. Keep the root
---

# Agent Guidelines for Viscribe

This is the canonical agent guide for the Viscribe repository. Keep the root
`AGENTS.md` as a lightweight discovery pointer or symlink to this file so
tooling can find the shared instructions while `.agents/` remains the source of
truth.

Viscribe is a Python and TypeScript SDK for structured image understanding with
OpenAI-compatible vision models. The public surface is intentionally small:
image methods live under `viscribe.images` and `client.images` in Python, and
under `images` and `client.images` in TypeScript.

## Maintenance Contract

- Keep this file concise and repo-wide.
- Put narrow, reusable workflows in `.agents/skills/**`.
- Update this file when repository structure, required verification, licensing
  guidance, or durable agent conventions materially change.
- Do not codify one-off preferences or task-local decisions here.
- Preserve user changes in the working tree. Do not revert unrelated edits.

## Start Here By Task

- Product and SDK architecture principles:
  `.agents/ARCHITECTURE_PRINCIPLES.md`
- Shared agent setup and config:
  `.agents/README.md`
- Stress-testing plans, public API changes, or larger refactors:
  `.agents/skills/grill-me/SKILL.md`
- Creating or refining shared skills:
  `.agents/skills/skill-creator/SKILL.md`
- Python SDK implementation:
  `python/src/viscribe/`
- Python tests and examples:
  `python/tests/`, `python/examples/`
- TypeScript SDK implementation:
  `typescript/src/`
- TypeScript tests and examples:
  `typescript/tests/`, `typescript/examples/`
- CI, issue templates, PR templates, and dependency automation:
  `.github/`
- Public docs and package READMEs:
  `docs/`, `README.md`, `CONTRIBUTING.md`, `SECURITY.md`,
  `python/README.md`, `typescript/README.md`
- Shared assets:
  `assets/`

Read the smallest set of files needed for the task. More-specific guidance in a
subdirectory takes precedence over this root guide for that scoped area.

## Default Planning Behavior

Use `.agents/skills/grill-me/SKILL.md` before finalizing non-trivial public API
changes, SDK architecture decisions, refactor strategies, compatibility breaks,
or release plans. Skip it for small mechanical tasks, direct implementation
requests with an already-approved plan, or narrow verification work.

## Project Structure

```text
viscribe/
├─ python/                  # Python package, tests, and runnable examples
├─ typescript/              # TypeScript package, tests, and runnable examples
├─ docs/                    # Mintlify documentation site
├─ assets/                  # Shared README/package assets
├─ .github/                 # GitHub Actions, Dependabot, and templates
├─ .agents/                 # Shared agent instructions and skills
├─ README.md                # Root package documentation
├─ CONTRIBUTING.md          # Contribution and branch workflow
├─ SECURITY.md              # Vulnerability reporting policy
├─ ROADMAP.md               # Product direction
└─ LICENSE                  # MIT license
```

## Core Commands

Python:

- Install/sync dependencies from `python/`: `uv sync`
- Test from `python/`: `uv run python -m pytest`
- Lint from `python/`: `uv run ruff check .`
- Run an example from `python/`: `uv run python examples/<name>.py`

TypeScript:

- Install dependencies from `typescript/`: `npm install`
- Test from `typescript/`: `npm test`
- Typecheck from `typescript/`: `npm run typecheck`
- Build from `typescript/`: `npm run build`
- Run examples from `typescript/`: `npm run example`,
  `npm run example:describe`, `npm run example:classify`,
  `npm run example:ask`, `npm run example:compare`, or
  `npm run example:client`

Docs:

- Install the Mintlify CLI if needed: `npm install -g mint`
- Run Mint with an LTS Node version supported by the CLI. Mint currently
  rejects Node 25+, so switch to Node 22 or 24 before running `mint` if the
  active shell is newer. Temporary fallback:
  `cd docs && npx -y -p node@22 node "$(command -v mint)" dev --no-open`
- Preview docs locally from `docs/`: `mint dev --no-open`
- Validate the docs build from `docs/`: `mint validate`
- Check docs links from `docs/`: `mint broken-links`
- Validate docs config from the repo root:
  `python3 -m json.tool docs/docs.json >/dev/null`

Root and agent files:

- Install root tooling: `npm install`
- Check formatting: `npm run format:check`
- Format supported files: `npm run format`
- Check a commit or PR title: `printf '%s\n' "feat: example" | npm run commitlint -- --verbose`
- Run semantic-release locally only in dry-run mode unless explicitly publishing:
  `npm run release -- --dry-run`
- Validate JSON: `python3 -m json.tool .agents/config.json`
- Validate YAML: `uv run --with pyyaml python -c "import pathlib, yaml; [yaml.safe_load(p.read_text()) for p in pathlib.Path('.github').rglob('*.yml')]"`
- Lint GitHub Actions workflows: `actionlint .github/workflows/*.yml`
  or, if `actionlint` is not installed, `go run github.com/rhysd/actionlint/cmd/actionlint@latest .github/workflows/*.yml`
- Check whitespace: `git diff --check`

## Verification Defaults


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itsperini/viscribe](https://github.com/itsperini/viscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
