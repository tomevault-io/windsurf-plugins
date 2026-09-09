---
trigger: always_on
description: VGV AI Flutter Plugin provides best-practices skills for Flutter and Dart development. It is a **documentation-only repository** — there is no Dart/Flutter source code, no `pubspec.yaml`, and no tests. All value lives in the markdown skill files.
---

# VGV AI Flutter Plugin

## Project Overview

VGV AI Flutter Plugin provides best-practices skills for Flutter and Dart development. It is a **documentation-only repository** — there is no Dart/Flutter source code, no `pubspec.yaml`, and no tests. All value lives in the markdown skill files.

## Repository Structure

```text
.mcp.json                # MCP server configuration (Dart and Very Good CLI)
.claude-plugin/
  plugin.json          # Plugin manifest (name, version, keywords)
agents/
  flutter-reviewer.md  # Read-only Flutter code reviewer subagent
docs/
  plan/                # Planning and design documents
evals/
  README.md            # Case format, assertion reference, how to add a case
  promptfooconfig.yaml # Claude Agent SDK provider + the two ablation columns
  tests/               # Eval cases, one YAML file per skill — all 15 covered, 100 cases
    accessibility.yaml
    animations.yaml
    bloc.yaml
    create-project.yaml
    dart-flutter-sdk-upgrade.yaml
    green-gate.yaml
    internationalization.yaml
    layered-architecture.yaml
    license-compliance.yaml
    material-theming.yaml
    navigation.yaml
    static-security.yaml
    testing.yaml
    ui-package.yaml
    very-good-analysis-upgrade.yaml
  assertions/
    dart-parses.js     # The one custom promptfoo assertion we own
  ci-summary.js        # Renders a promptfoo export into a GitHub step summary
  fixture/
    pubspec.yaml       # Neutral Flutter skeleton used as working-directory context
hooks/
  hooks.json           # Hook definitions (PreToolUse and PostToolUse)
  scripts/
    allow-readonly-git.sh  # Restricts flutter-reviewer Bash to git diff/status
    analyze.sh         # Runs dart analyze on modified .dart files
    block-cli-workarounds.sh  # Prevents direct CLI bypass via Bash
    check-vgv-cli.sh   # Validates VGV CLI installed and >= 1.3.0
    format.sh          # Runs dart format on modified .dart files
    vgv-cli-common.sh  # Shared utilities for VGV CLI hook scripts
    warn-missing-mcp.sh  # Warns at session start if VGV CLI is missing/outdated
skills/                  # every <skill>/ ships SKILL.md + agents/openai.yaml (Codex sidecar)
  accessibility/SKILL.md
  accessibility/references/
  animations/SKILL.md
  animations/references/
    explicit-animations.md
    looping-animations.md
    page-transitions.md
    staggered-animations.md
  bloc/SKILL.md
  bloc/references/
  create-project/SKILL.md
  dart-flutter-sdk-upgrade/SKILL.md
  dart-flutter-sdk-upgrade/references/
    version-conflicts.md
  green-gate/SKILL.md
  green-gate/references/
    coverage.md
  internationalization/SKILL.md
  layered-architecture/SKILL.md
  layered-architecture/references/
  license-compliance/SKILL.md
  material-theming/SKILL.md
  navigation/SKILL.md
  static-security/SKILL.md
  static-security/references/
  testing/SKILL.md
  testing/references/
  ui-package/SKILL.md
  ui-package/reference.md
  very-good-analysis-upgrade/SKILL.md
  very-good-analysis-upgrade/references/
    lint-fixes.md
```

## Skill File Format

Every `SKILL.md` follows this structure:

1. **YAML frontmatter** with the following fields:
   - `name` _(required)_ — must match the skill's folder name exactly; lowercase letters, numbers, and hyphens only (e.g., `bloc`)
   - `description` _(required)_ — when the skill should be triggered
   - `allowed-tools` _(optional)_ — space-separated list of tools the skill may use (e.g., `Read Glob Grep`)
   - `argument-hint` _(optional)_ — placeholder hint shown to the user (e.g., `"[file-or-directory]"`)
2. **H1 title** — human-readable skill name
3. **Core Standards** — enforced constraints, always first
4. **Content sections** — architecture, code examples, workflows, anti-patterns

Every skill also ships a Codex sidecar at `agents/openai.yaml` beside its `SKILL.md`, holding the skill-picker metadata `interface.display_name` and `interface.short_description`. Every skill in this plugin is **model-invoked** (the model may auto-activate it), so no skill sets `disable-model-invocation` or a Codex `policy` block; a user-invoked-only skill would set both, kept in sync. See `CONTRIBUTING.md` → Cross-harness portability.

## Writing Conventions

- Frame standards as clear directives — no soft language ("consider", "prefer")
- Use fenced code blocks with language identifiers for all examples
- Provide complete, copy-pasteable snippets, not fragments
- Reference packages by full name (e.g., `package:mocktail`)
- Include anti-patterns alongside correct patterns when helpful
- Align pipe characters vertically in all markdown tables (enforced by markdownlint MD060)

## Adding a New Skill

1. Create `skills/<skill_name>/SKILL.md` following the format above, plus the Codex sidecar
   `skills/<skill_name>/agents/openai.yaml` (`interface.display_name` + `interface.short_description`)
2. Create `evals/tests/<skill_name>.yaml` — eval cases with one prompt per major
   workflow the skill covers, a `skill-used` assertion on each, and one
   `not-skill-used` negative control. Routing assertions carry `weight: 3` so a routing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VeryGoodOpenSource/vgv-ai-flutter-plugin](https://github.com/VeryGoodOpenSource/vgv-ai-flutter-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
