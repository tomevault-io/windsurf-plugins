---
trigger: always_on
description: **Transform product ideas into approved, testable plans — systematically.**
---

# stelow

**Transform product ideas into approved, testable plans — systematically.**

## Project Overview

**Type:** Workflow CLI for product planning (skills + stages).
**Stack:** Node 20+, TypeScript 6.0 strict, npm.

## Architecture

See [architecture.md](architecture.md) for module layout, data flow, and how to extend. Skills live in `skills/*/SKILL.md`; stages defined in `stages.yaml` (single source of truth). Visual review gates: `gate`, `int-gate`, `plan-gate`, `diff-gate` (Plannotator) — conditional by review mode.

### Critical Muxy extension knowledge

**Permission `files:read` is required but NOT in the pinned schema.**
The Muxy files API (`muxy.files.read/write/list`) requires
`files:read` and `files:write` in `manifest.permissions` *per the
official Muxy docs*. However, the pinned manifest schema at
`integrations/muxy/stelow/manifest.schema.json` is
OUTDATED — it was fetched before Muxy added these permissions to
the schema enum. **Do NOT trust the pinned schema as the source
of truth for valid permissions.** The source of truth is:
- https://muxy.app/docs/extensions/files (docs)
- https://muxy.app/docs/extensions/permissions (permissions list)
- Muxy.app runtime validator (what actually accepts/rejects)

The pinned schema is only useful for structural validation
(panel keys, command shapes) — NOT for permission validity.
If a muxy extension suddenly shows "No workflow data" despite
correct workspace, check that `files:read` and `files:write`
are present in package.json muxy.permissions.

### Top-level layout

| Directory | Purpose |
|---|---|
| `skills/` | Stelow skills consumed by pi coding agents (LLM-facing) |
| `extensions/stelow/` | Pi runtime extension (in-process TS) — single extension, all `/sw-*` commands and hooks live here. Workflow root detection (`workflow-root.ts`) is the source of truth across all integrations; mirror functions exist in `integrations/muxy/` and `integrations/herdr/`. |
| `integrations/<host>/<plugin>/` | Plugins for **external hosts** (Muxy, Herdr, etc.) — each host has its own incompatible extension model. Shipped via `package.json#files[]` and synced through `scripts/version-sync.mjs`. |
| `docs/design/` | Design docs, plans, ADR (PT-BR discussion, EN artifacts) |
| `stelow.schema.json` / `stelow.json` | Workflow tracking schema + runtime state |


## Commands

| Command | Description |
|---------|-------------|
| `/sw-start` | Begin planning |
| `/sw-status` | Show workflow status |

> **Command aliases:** `/stelow-*` names are registered alongside `/sw-*` for readability. Both prefixes work.

> **Source of Truth:** Stage/skill counts derive from `stages.yaml` and `ls skills/*/SKILL.md | wc -l`. Never update counts here without verifying.

```bash
npm run build            # Compile TypeScript
npm test                 # Run all tests
npm run test:unit        # Unit tests only
npm run test:integration # Integration tests only
npm run test:skills      # Skill structure tests
npm run typecheck        # Type check
```

## Conventions

- **Commits:** conventional (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`). Squash merge to main.
- **Files:** `lowercase-kebab-case` (e.g. `spec-product.md`, not `SpecProduct.md`).
- **Stage headings:** must use `slug:major.minor` format — see [docs/agents-md-refs/stage-numbering.md](docs/agents-md-refs/stage-numbering.md) for the gap-based numbering rules.
- **Tool calls in stage files:** never call `ask_user_question`, `subagent`, or `start_supervision` directly. Use the CLI-agnostic reference in `references/cli-tools/{tool}.md` — see [docs/agents-md-refs/tool-reference-pattern.md](docs/agents-md-refs/tool-reference-pattern.md).
- **Product name:** `stelow` (canonical). Legacy `cali-product-workflow` / `Cali Product Workflow` must NOT be used in new files, manifests, display names, or commands. The runtime state directory `.cali-product-workflow/` is the one exception (filesystem path kept for backward compat — do not change).

## Versioning

- **Single source:** `package.json` → `npm run version:sync` syncs plugin files.
- **Tag and Release are linked — never create one without the other.** A git tag alone does not create a GitHub Release; the landing page shows only Releases, not tags.
- **Full release workflow (do NOT skip steps):**
  1. `npm version <major.minor.patch> --no-git-tag-version` — bump `package.json`
  2. `npm run version:sync` — sync plugin files
  3. Update `CHANGELOG.md` — add entry with changes
  4. `git add -A && git commit -m "chore: bump to v<version>"`
  5. `git tag -a v$(node -p "require('./package.json').version") -m "v<version>: <summary>"`
  6. `git push origin main --tags`
  7. **`gh release create v$(node -p "require('./package.json').version") --title "v<version>" --notes "<changelog>"`** — required for GitHub landing page visibility
- **Never guess the version** — always read `package.json` first.

## Don'ts

- **Do NOT put ops-only config inside `skills/*/`.** Files consumed by extension/ops
  code (never by the LLM in runtime) go at project root. If a file is read by
  `extensions/`, `scripts/`, or `install.sh` — not by `SKILL.md` — it belongs at
  root, not inside a skill directory. Example: `retired-skills.yaml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calionauta/stelow](https://github.com/calionauta/stelow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
