---
trigger: always_on
description: This file contains instructions for GitHub Copilot when working with the Phased Agent Workflow (PAW) project.
---

# GitHub Copilot Instructions

This file contains instructions for GitHub Copilot when working with the Phased Agent Workflow (PAW) project.

## Code Quality

All code changes must pass the linter before being committed:

```bash
npm run lint
```

## Agent Development

When creating or modifying agent files in `agents/`, ALWAYS run the prompting linter script:

```bash
./scripts/lint-prompting.sh agents/<filename>.agent.md
```

When creating or modifying skill files in `skills/`, ALWAYS run the prompting linter script:

```bash
./scripts/lint-prompting.sh skills/<skillname>/SKILL.md
```

To lint all agents and skills at once:

```bash
npm run lint:agent:all
```

To lint only skills:

```bash
npm run lint:skills
```


### Documentation Updates

When implementing features or making changes that affect user-facing behavior, consider updating:

- **User Guide** (`docs/guide/`) - For user-facing features, commands, or workflows
- **Specification** (`docs/specification/`) - For changes to workflow stages or agent behavior
- **Reference** (`docs/reference/`) - For new agents, artifacts, or configuration options

### Validation

Before committing documentation changes:
```bash
source .venv/bin/activate
mkdocs build --strict
```

This validates all internal links and catches configuration errors.

## Pull Request Labels

All pull requests to `main` must be labeled with one of the following category labels:
- `enhancement` - For new features
- `bug` - For bug fixes
- `documentation` - For documentation changes
- `maintenance` - For maintenance, refactoring, or chores

### CLI Label

PRs that **only** affect CLI-specific code (`cli/` directory, `publish-cli.yml`, etc.) should have the `cli` label. These won't appear in the VS Code extension changelog.

### VS Code Label

PRs that **only** affect VS Code extension code should have the `vscode` label. These are excluded from the CLI release changelog.

PRs that touch agents, skills, prompts, or shared code don't need a platform label—they're included in both changelogs by default. Platform labels are applied at release-prep time using `prepare-cli-release.prompt.md`.

## Platform Runtime Separation

- PAW has two runtimes: GitHub Copilot CLI and the VS Code extension.
- Copilot CLI executes installed `agents/`, `skills/`, and prompt content directly. It does **not** run `src/` TypeScript or access `vscode.ExtensionContext` state.
- `src/` is VS Code-only automation (commands, extension state, worktree handoff, prompt/template loading). Do not treat it as hidden runtime support for CLI behavior.
- If a behavior depends on VS Code-only automation or state, document it as VS Code-only or provide a CLI analogue in agent/skill text.
- `tests/integration/` exercises CLI runtime behavior through the Copilot SDK, not the VS Code extension implementation.

IMPORTANT: **PAW Architecture Philosophy** - tools provide procedural operations, agents provide decision-making logic and reasoning. Rely on agents to use reasoning and logic over hardcoding procedural steps into tools.

## Skill Development

This project has two skill locations with different capabilities:

| Location | Loaded Via | Bundled Resources |
|----------|------------|-------------------|
| `.github/skills/` | VS Code skills system | ✅ Supported (`scripts/`, `references/`, `assets/`) |
| `skills/` | `paw_get_skill` tool | ⚠️ SKILL.md via tool; `references/` via filesystem |

**When creating skills in `skills/`**: The `paw_get_skill` tool returns only SKILL.md content as text. Skills may include a `references/` subdirectory for supplementary content that subagents load via filesystem tools (`view`, `glob`, `grep`) — these files are NOT accessible through `paw_get_skill`. Do not create `scripts/` or `assets/` subdirectories.

**When creating skills in `.github/skills/`**: Full Agent Skills spec is supported, including bundled resources that Claude can read or execute.

**For skill creation guidance**: Load the `skill-creator` skill at `.github/skills/skill-creator/SKILL.md` for comprehensive guidance on skill design principles, anatomy, and creation process.

## Shell Commands

When writing shell commands in agent prompts or documentation, follow these guidelines:

- DO NOT pipe output to /dev/null (e.g. `2>/dev/null`), as it forces the command to require approval.
- DO NOT activate virtual environments on every command (e.g. `source .venv/bin/activate && ...`); activate it once per session instead.

## Agent Prompt Edit Token Discipline (for `*.agent.md`, system prompts, instruction prompts)**

* Treat system prompt tokens as **expensive** (paid every run).
* Compute net token addition percentage for new prompt content, ensure the percentage matches the value delivered.
* Prefer **replacing/condensing** existing text over appending.
* Use **bullets over prose**; avoid rationale paragraphs and long examples.
* Always report **before/after token counts** (via lint) and the delta; if over budget or token additions outpace value, do a **compression pass**.

## Prompt Writing Best Practices

When writing agent prompts, skills, or instructions, follow these guidelines to maximize agent effectiveness while minimizing token usage.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lossyrob/phased-agent-workflow](https://github.com/lossyrob/phased-agent-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
