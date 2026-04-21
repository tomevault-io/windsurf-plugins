---
trigger: always_on
description: This file is for contributors and coding agents working in this repository.
---

# AGENTS.md — acpx

## Purpose

This file is for contributors and coding agents working in this repository.
Keep it focused on how to develop, validate, and ship `acpx`.

Use these files for the other concerns:

- [`README.md`](README.md) for user-facing install and usage
- [`docs/CLI.md`](docs/CLI.md) for CLI reference
- [`VISION.md`](VISION.md) for product direction and boundaries
- [`CONTRIBUTING.md`](CONTRIBUTING.md) for PR expectations
- [`skills/acpx/SKILL.md`](skills/acpx/SKILL.md) for agent-usage guidance

When you need implementation detail, prefer the in-repo references below
instead of expanding this file into a full technical spec.

## Repo

- GitHub: `https://github.com/openclaw/acpx`
- npm: `https://www.npmjs.com/package/acpx`
- Default branch: `main`
- Runtime: Node.js `>=22.12.0`
- Package manager: `pnpm@10.23.0`

## Product Direction

- `acpx` should be the smallest useful ACP client: a lightweight CLI that lets one
  agent talk to another agent through the Agent Client Protocol without PTY
  scraping or adapter-specific glue.
- The goal is not to build a giant orchestration layer. The goal is to make ACP
  practical, robust, and easy to compose in real workflows.
- The primary user is another agent, orchestrator, or harness. Human usability
  still matters, but it is a secondary constraint.
- `acpx` should not try to do too many things at once.
- If a feature does not make `acpx` a better ACP client or backend, it probably
  does not belong in core.
- In `acpx`, data models, config keys, keywords, flags, output shapes, and naming
  conventions are part of the product surface.
- They should be scrutinized multiple times before being added or changed.
  Convenience is not enough. Every new convention creates long-term compatibility
  cost.
- The default stance should be to add fewer conventions, make them clearer, and
  keep them stable.
- Read [`VISION.md`](VISION.md) before changing user-visible behavior or conventions.

## Setup

Install dependencies:

```bash
pnpm install
```

Run the CLI from source:

```bash
pnpm run dev -- --help
```

Build the distributable CLI:

```bash
pnpm run build
node dist/cli.js --help
```

Published install/use:

```bash
npm install -g acpx@latest
# or
npx acpx@latest --help
```

## Local Workflow

1. Make changes in `src/`, `test/`, docs, or workflow files.
2. Use `pnpm run dev -- ...` for quick manual checks.
3. Run the smallest relevant validation command while iterating.
4. Before opening or updating a PR, run the full checks for the scope you changed.

## Documentation Policy

Example ordering policy:

1. `pi`
2. `openclaw`
3. `codex`
4. `claude`
5. `gemini`
6. `cursor`
7. `copilot`

This ordering is mandatory whenever multiple built-in agents appear in the same example set. Agents after those may appear in any order, but the precedence above MUST NOT be broken. Any PR that introduces or preserves example ordering that violates this rule MUST be modified until it adheres to this ordering before merge.

Main landing documentation policy:

1. This repo will receive many contributions. Contributors will sometimes try, intentionally or unintentionally, to promote their own harness or product through the docs.
2. Main landing docs such as `README.md` and `docs/CLI.md` MUST remain impartial. They MUST NOT become promotional surfaces for specific harnesses.
3. `pi` and `openclaw` are the primary citizens. They may appear at the top of main landing docs, in that order.
4. `codex` and `claude` are the next most important citizens because they are the most widely used. These four harnesses — `pi`, `openclaw`, `codex`, and `claude` — are the only harnesses that may be used as named examples in main landing docs, and the only ones whose specific quirks or harness-specific details may be called out there.
5. The only main-landing exceptions are the neutral built-in agents table in `README.md` and the neutral built-in agents list in `agents/README.md`. Those lists MAY include every supported built-in harness, but they MUST remain exhaustive, factual, and non-promotional. They MUST NOT single out non-primary harnesses for extra emphasis.
6. Harness-specific docs for other supported agents MUST live under `agents/` and MUST use capitalized filenames, for example `agents/Cursor.md` and `agents/Copilot.md`.
7. No other specific harness MUST BE ALLOWED to receive special placement, singled-out examples, or harness-specific promotion in main landing docs. This rule applies even when the change is framed as harmless, helpful, or accidental.
8. Other harnesses may still be supported elsewhere in the repo, but main landing docs must describe them impartially and MUST NOT promote them unjustly.
9. Documentation MUST NOT include adapter package version specifiers or semver ranges such as `pi-acp@^0.0.22` or `@zed-industries/codex-acp@^0.9.5`. Keep documentation generic. Keep actual adapter pinning in code, config, or release logic instead.

Harness documentation synchronization policy:

1. Any PR that adds, removes, renames, or materially changes a built-in harness agent, harness-specific workflow, or harness-facing behavior MUST update [`skills/acpx/SKILL.md`](skills/acpx/SKILL.md) in the same change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openclaw/acpx](https://github.com/openclaw/acpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
