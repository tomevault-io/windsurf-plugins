---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code Abyss is an npm package that installs a "邪修红尘仙" persona configuration into Claude Code, Codex CLI, and Gemini CLI. It delivers: persona rules, 4 switchable output styles, 56 skill documents, and 5 executable verification/generation tools.

## Commands

```bash
npm test                          # Run Jest test suite
npm run verify:skills             # Validate all SKILL.md frontmatter contracts (fail-fast gate)
node bin/install.js --help        # Installer CLI help
node bin/install.js --target claude -y   # Zero-config install to ~/.claude/
node bin/install.js --target codex -y    # Zero-config install to ~/.codex/
node bin/install.js --target gemini -y   # Zero-config install to ~/.gemini/
node bin/install.js --list-styles        # List available output styles
```

Running individual verify tools directly:
```bash
node skills/tools/verify-security/scripts/security_scanner.js <path>
node skills/tools/verify-module/scripts/module_scanner.js <path>
node skills/tools/verify-change/scripts/change_analyzer.js --mode staged|working
node skills/tools/verify-quality/scripts/quality_checker.js <path>
node skills/tools/gen-docs/scripts/doc_generator.js <path>
```

Running a single test file:
```bash
npx jest test/install-registry.test.js --runInBand
```

CI runs on Node 18/20/22: `npm ci && npm test && npm run verify:skills` plus all 4 verify tools + smoke install/uninstall on 3 platforms.

## Architecture

### Three-Layer System

| Layer | Source | Purpose |
|-------|--------|---------|
| Identity & Rules | `config/CLAUDE.md` | Persona, rules, scene routing, execution chains |
| Output Style | `output-styles/*.md` + `index.json` | Style registry + per-style templates |
| Knowledge | `skills/**/*.md` | Domain skill documents + executable tools |

`config/AGENTS.md` remains a repository snapshot, but Codex runtime installation no longer writes a generated `~/.codex/AGENTS.md`. Codex now runs in a `skills-only` shape and installs Code Abyss plus gstack under `~/.agents/skills/`.

### Skill Registry (Single Source of Truth)

`bin/lib/skill-registry.js` is the authoritative skill discovery engine for installed skills, `run_skill.js`, Claude command generation, and CI validation.

- Each skill's metadata lives in `skills/**/SKILL.md` YAML frontmatter
- Required fields: `name` (kebab-case slug), `description`, `user-invocable`
- Optional fields: `allowed-tools` (default: `Read`), `argument-hint`, `aliases`
- `category` is auto-inferred from directory prefix (`tools/` → tool, `domains/` → domain, `orchestration/` → orchestration)
- `runtimeType` is auto-inferred: `scripts/` has exactly one `.js` → `scripted`, else `knowledge`
- Registry fail-fast validates: missing fields, bad slugs, illegal tool names, duplicate names, multiple script entries

### Pack Registry

`packs/*/manifest.json` defines installable packs. `abyss` is the core pack; `gstack` is a pinned upstream pack consumed by the Claude/Codex auto-install flows. `bin/lib/pack-registry.js` is the source of truth for host file mappings and upstream metadata.

Project-level automatic pack sync is driven by `.code-abyss/packs.lock.json`. The installer reads the nearest lock file from the current working directory upward and installs host-specific packs according to `required`, `optional`, `optional_policy`, and `sources`. `node bin/packs.js bootstrap` initializes the lock plus README/CONTRIBUTING snippets, `--apply-docs` writes them back into repo docs, `vendor-pull` / `vendor-sync` manage local sources, `vendor-sync --check` acts as a gate, `report summary` reads `.code-abyss/reports/`, and `uninstall <pack>` removes pack-specific runtime artifacts with a report.

### Style Registry

`bin/lib/style-registry.js` manages `output-styles/index.json`. Exactly one style must be `default`. Each style has `slug`, `label`, `description`, `file`, `targets`, `default`.

### Dual-Target Generation

The installer generates different artifacts per target CLI:

- **Claude**: `~/.claude/commands/*.md` (slash commands) — `runtimeType=scripted` calls `run_skill.js`, `knowledge` reads SKILL.md directly
- **Codex**: `~/.agents/skills/**/SKILL.md` — Codex discovers user skills from `~/.agents/skills`; Code Abyss auto-installs an embedded gstack runtime under `~/.agents/skills/gstack`
- **Gemini**: `~/.gemini/GEMINI.md` + `~/.gemini/commands/*.toml` + `~/.gemini/skills/**/SKILL.md` — Gemini reads persistent context from `GEMINI.md` and custom commands from TOML files

Claude command generation and Codex skill installation share the same skill source tree; only Claude filters on `user-invocable` to emit slash commands.

### Adapter Pattern

`bin/install.js` is the orchestration layer. Target-specific logic lives in adapters:
- `bin/adapters/claude.js` — Claude auth detection, settings merge, core files mapping
- `bin/adapters/codex.js` — Codex auth detection, config.toml merge, core files mapping
- `bin/lib/ccstatusline.js` — Claude status bar (ccstatusline) integration
- `bin/lib/style-registry.js` — Style catalog + repository AGENTS snapshot assembly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telagod/code-abyss](https://github.com/telagod/code-abyss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
