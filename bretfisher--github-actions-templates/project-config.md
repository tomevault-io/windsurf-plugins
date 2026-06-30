---
trigger: always_on
description: This repository is a GitHub Actions template and reusable-workflow repo.
---

# AGENTS.md

This repository is a GitHub Actions template and reusable-workflow repo.
Most edits are YAML, Markdown, and a small sample Dockerfile.
There is no application runtime, package manager, or unit-test framework checked into this repository.

## Scope And Intent

- Treat this repo as documentation plus workflow templates.
- Prefer minimal, targeted edits that preserve template readability.
- Keep reusable workflows in `.github/workflows/reusable-*.yaml` generic.
- Keep example caller workflows in `templates/` and local test callers in `.github/workflows/call-local-*.yaml` aligned.
- Do not invent repo infrastructure that is not already present.

## Repository Layout

- `.github/workflows/reusable-*.yaml`: reusable workflows meant to be called from other repos.
- `.github/workflows/call-local-*.yaml`: local test callers for workflows in this repo.
- `.github/workflows/call-super-linter.yaml`: lint workflow for this repo.
- `templates/`: copyable workflow templates for downstream repositories.
- `.github/linters/`: shared linter configuration used by Super-Linter.
- `Dockerfile`: sample image used for Docker-related workflow examples.
- `README.md`: repository overview and usage notes.

## Source Of Truth

- Follow the existing YAML style in current workflow files before applying generic preferences.
- Treat `.github/linters/.yaml-lint.yml` as the main YAML formatting authority.
- Treat `.github/linters/.markdown-lint.yml` as the Markdown formatting authority.
- Treat `.github/linters/.hadolint.yaml` as the Dockerfile lint authority.
- There are no Cursor rules in `.cursor/rules/`.
- There is no `.cursorrules` file.
- There is no Copilot instruction file at `.github/copilot-instructions.md`.

## Build, Lint, And Test

### Reality Check

- There is no single `make`, `npm`, `pnpm`, `pytest`, or similar project entrypoint.
- There is no checked-in unit-test harness.
- The practical validation strategy is: lint changed files, optionally build the sample Dockerfile, and validate workflows through GitHub Actions.

### Useful Commands

Run these from the repository root.

Build the sample container:

```bash
docker build -t gha-templates-test .
```

Lint all YAML:

```bash
yamllint -c .github/linters/.yaml-lint.yml .
```

Lint all Markdown relevant to this repo:

```bash
markdownlint -c .github/linters/.markdown-lint.yml README.md AGENTS.md
```

Lint the Dockerfile:

```bash
hadolint --config .github/linters/.hadolint.yaml Dockerfile
```

Lint GitHub Actions workflow syntax:

```bash
actionlint
```

Lint GitHub Actions workflow security posture:

```bash
zizmor .
```

Lint a single YAML file:

```bash
yamllint -c .github/linters/.yaml-lint.yml path/to/file.yaml
```

Lint a single Markdown file:

```bash
markdownlint -c .github/linters/.markdown-lint.yml path/to/file.md
```

### Tests / Validation

- There is no dedicated local `test` command in this repo.
- The smallest meaningful repo-native validation unit is usually a single file lint.
- For workflow behavior, use the matching local caller workflow in `.github/workflows/call-local-*.yaml` when one exists.
- For reusable workflow changes, update the reusable file and its local caller or example caller together when needed.
- Validate behavior in GitHub Actions rather than claiming local execution coverage that the repo does not provide.
- For workflow security and hygiene, prefer running both `actionlint` and `zizmor .` after workflow edits.

### Single Test Guidance

- There is no unit-test-per-file command such as `pytest path::test_name` or `vitest file.test.ts` here.
- If the change is YAML-only, the closest equivalent of a single test is linting the one changed YAML file.
- If the change affects a reusable workflow, the closest equivalent of a single integration test is exercising the corresponding `call-local-*.yaml` workflow in GitHub Actions.
- If the change is GitHub Actions-specific, also run `actionlint` and review `zizmor` findings for the affected workflow.
- Do not fabricate `act`, `actionlint`, or other tooling commands unless the user explicitly wants to add and maintain them.

## Workflow Editing Rules

- Preserve the distinction between reusable workflows, template callers, and local test callers.
- Keep comments that explain why a setting exists; this repo uses instructional comments intentionally.
- Keep `permissions` explicit in workflows. Existing examples emphasize this.
- Keep concurrency blocks when they already exist.
- Do not convert reusable workflows into direct-use workflows unless explicitly requested.
- Prefer updating both the template and the local test workflow when they mirror each other.

## YAML Style

- Start workflow and template YAML files with `---` when the existing file does.
- Use two-space indentation.
- Keep sequence indentation consistent; `.yaml-lint.yml` requires `indent-sequences: true`.
- Do not worry about line length; YAML line-length linting is disabled.
- Use Unix newlines.
- Avoid trailing spaces.
- Duplicate keys are never acceptable.
- Comments and comment indentation are intentionally not lint-enforced, but keep them readable.

## Markdown Style

- Long lines are acceptable; Markdown line-length is set very high.
- Keep headings and lists simple and readable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BretFisher/github-actions-templates](https://github.com/BretFisher/github-actions-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
