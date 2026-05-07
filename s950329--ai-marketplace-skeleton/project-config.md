---
trigger: always_on
description: > For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md — ai-plugin-skeleton

> For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## 1. Project Overview

An AI skill plugin skeleton published in Plugin Marketplace format, compatible with both **Claude Code** and **GitHub Copilot CLI**.

Skills are grouped by user role into plugins:

| Plugin | Description | Target Users |
|--------|-------------|--------------|
| `common-tools` | General-purpose tools: skill creation and evaluation | All roles |

> When using this skeleton, add plugins and skills as needed.

---

## Boundaries

- **Always**: Bump `plugin.json` version before pushing; verify symlinks point to the correct primary file; sync README.md and AGENTS.md whenever the skill list changes.
- **Ask first**: Creating a brand-new plugin, deleting files, renaming an existing skill, modifying CI configuration.
- **Never**: Push directly to `main`; merge a PR without review.

---

## 2. Directory Structure

```
ai-plugin-skeleton/
├── marketplace.json              # Marketplace definition (primary file)
├── .claude-plugin/
│   └── marketplace.json          # → ../marketplace.json (symlink, read by Claude Code)
├── .github/
│   └── plugin/
│       └── marketplace.json      # → ../../marketplace.json (symlink, read by Copilot CLI)
├── plugins/
│   └── common-tools/             # Example plugin
│       ├── plugin.json           # Plugin manifest (primary file)
│       ├── .claude-plugin/
│       │   └── plugin.json       # → ../plugin.json (symlink)
│       ├── .github/
│       │   └── plugin/
│       │       └── plugin.json   # → ../../plugin.json (symlink)
│       └── skills/
│           └── skill-creator/
│               └── SKILL.md
├── scripts/
│   └── install.sh                # One-click install script
├── AGENTS.md                     # This file
├── CONTRIBUTING.md               # Contributor guide for humans
└── README.md
```

---

## 3. Complete Flow for Adding a Skill

### 3a. Choose a Plugin

Pick an existing plugin that fits the target user role, or create a new one.

If none of the existing plugins are a good fit, follow [3d. Register in the marketplace](#3d-register-in-the-marketplace) to create a new plugin.

### 3b. Create SKILL.md

File location: `plugins/{plugin-name}/skills/{skill-name}/SKILL.md`

**Frontmatter template:**

```yaml
---
name: my-skill-name
description: >
  One sentence describing what the skill does. Then list specific trigger conditions:
  trigger when the user mentions "keyword1", "keyword2", or similar phrases.
  Also describe when this skill should NOT be used.
---
```

**Required fields:**
- `name`: kebab-case — **do not add a namespace prefix** (e.g. `dev-tools:`) — this causes Copilot CLI to fail loading the skill.
- `description`: First sentence states the core function; list trigger keywords; describe inapplicable situations.

**Recommended SKILL.md body structure:** Role definition → Workflow → Output format → Constraints

### 3c. Attach Sub-resources

| Directory | Purpose | When to use |
|-----------|---------|-------------|
| `references/` | Reference docs, code snippets | Skill needs to reference existing code or external specs |
| `scripts/` | Executable scripts | Skill needs to run validation, generate reports, or other automation |
| `agents/` | Sub-agent definitions | Skill needs to be split into multiple cooperating agents |

Path references: In SKILL.md, use **paths relative to SKILL.md itself**, e.g. `./references/schema.md`.

### 3d. Register in the Marketplace

- **Adding a skill to an existing plugin**: No changes to `marketplace.json` are needed (`source` points to the whole plugin directory, so new skills are picked up automatically).
- **Creating a brand-new plugin**:
  1. Create the `plugins/{new-plugin}/` directory structure (including `plugin.json`, `.claude-plugin/plugin.json` symlink, `.github/plugin/plugin.json` symlink, and `skills/`).
  2. Add an entry to the `plugins` array in the root `marketplace.json` (symlinks under `.claude-plugin/` and `.github/plugin/` sync automatically):
     ```json
     {
       "name": "new-plugin",
       "description": "Plugin description",
       "version": "1.0.0",
       "source": "./plugins/new-plugin",
       "category": "category",
       "tags": ["tag1", "tag2"]
     }
     ```

### 3e. Version Management

> ⚠️ After modifying any skill content, you **must** bump the `version` in the corresponding plugin's `plugin.json`; otherwise, users who already have it installed will not receive updates due to caching.

Files to keep in sync:
1. `version` in `plugins/{name}/plugin.json` (primary file — symlinks under `.claude-plugin/` and `.github/plugin/` sync automatically).
2. The `version` of the corresponding plugin entry in the root `marketplace.json` (primary file — symlinks sync automatically).

> 📌 The plugin entry `version` in `marketplace.json` is for display purposes, but it **must stay in sync with `plugin.json`** to avoid version discrepancies.

**Semver rules:**
- Modifying skill content → **patch** (1.0.0 → 1.0.1)
- Adding a new skill → **minor** (1.0.0 → 1.1.0)
- Breaking change (removing or renaming a skill) → **major** (1.0.0 → 2.0.0)

### 3f. Validation

```bash
# Structure validation
cd plugins/{plugin-name} && claude plugin validate .


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s950329/ai-marketplace-skeleton](https://github.com/s950329/ai-marketplace-skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
