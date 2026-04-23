---
trigger: always_on
description: <!-- project-template: 48 -->
---

<!-- project-template: 48 -->
# Claude Code Setup

## About

A modular, minimal setup for Claude Code with clear workflow and persistent memory via Markdown files. Open source release.

## Tech Stack

- Bash (install.sh)
- Markdown (templates, commands)
- Node.js (MCP servers via npx, Nextra docs site)
- GitHub Actions (CI/CD)

---

## Current Status

| Story | Status | Notes |
|-------|--------|-------|
| JetBrains Integration Guide | In Progress | PR #51 — docs-only guide for Debugger MCP + Built-in MCP |

**Legend:** Open | In Progress | Done

**Next Step:** Merge PR #51.

### Future

| Todo | Priority | Problem | Solution |
|------|----------|---------|----------|
| Docker Matrix Tests | Low | deps.json install commands not tested on real distros | GitHub Actions with Docker matrix ([Record 022](docs/records/022-docker-matrix-tests.md)) |
| Slidev skill type review | Low | `create-slidev-presentation` is command but could benefit from auto-loading | Consider changing to context with `applies_to: [slidev]` |

---

## Recent Decisions

| Date | Decision | Why |
|------|----------|-----|
| 2026-02-27 | 2 agents default + `--security`/`--full` flag for 3rd | Balance between review quality and token cost; security-auditor not needed for every review |
| 2026-02-27 | Tracking update after plugin install, not before | Prevents state mismatch if `claude plugin install` fails (review feedback) |
| 2026-02-11 | Replaced team-setup.mdx with customizing.mdx | All custom module topics (skills, commands, scripts, MCP) belong in one page; team-setup was redundant; Solo vs Team already in init-project |

---

## Project Instructions

<!-- PROJECT INSTRUCTIONS START -->

### Pull Request Format

When creating PRs, use this structure (see PRs #45, #46 for examples):

```markdown
## Summary
- Bullet points of what changed

## Problem
Why this change is needed

## Solution
How the problem is solved

## Test plan
- [ ] Test description 1
- [ ] Test description 2
- [ ] All tests pass

## Files Changed (optional)
- List of changed files
```

**Important:**
- Use "Test plan" section with checkboxes, NOT "Actions taken"
- Include `/do-review` results in test plan if applicable
- Keep summary concise (3-5 bullets max)

<!-- PROJECT INSTRUCTIONS END -->

---

## Architecture

**Install-time** and **runtime** are separate concerns:

### Install-time: `install.sh` + `lib/`

The installer is a modular Bash script. `install.sh` is the entry point, sourcing libraries from `lib/`:

| Library | Responsibility |
|---------|---------------|
| `platform.sh` | OS detection (macOS/Ubuntu/Arch/Fedora), package manager |
| `helpers.sh` | Colors, printing, JSON utilities, TTY-aware input, custom command/script install |
| `modules.sh` | Interactive toggle selection UI, module discovery |
| `mcp.sh` | MCP server installation (JSON config → `~/.claude.json`) |
| `skills.sh` | Skill installation + `build_claude_md()` |
| `update.sh` | `--update` mode, content version comparison |
| `uninstall.sh` | `--remove` mode |
| `external-plugins.sh` | Third-party plugin installation (document-skills, comprehensive-review) |
| `statusline.sh` | ccstatusline configuration |
| `hooks.sh` | Claude Code hooks setup |
| `agent-teams.sh` | Agent Teams env var toggle in settings.json |
| `setup-status.sh` | Discovery script for `/claude-code-setup` — outputs JSON status (standalone, NOT sourced by install.sh) |

**Install flow:** detect OS → select modules (interactive toggle) → copy commands to `~/.claude/commands/` → install MCP configs to `~/.claude.json` → copy skills to `~/.claude/skills/` → build global CLAUDE.md from template + dynamic tables → install external plugins → configure statusline/hooks/agent-teams.

### Runtime: Markdown files consumed by Claude Code

Nothing runs at runtime. The installer produces static Markdown files that Claude Code reads:

```
~/.claude/
├── CLAUDE.md              ← Global instructions (built from templates/base/global-CLAUDE.md + dynamic tables)
├── commands/*.md          ← Slash commands (Claude reads on /command invocation)
├── skills/*/SKILL.md      ← Context skills (Claude reads based on tech stack / file type)
└── templates/             ← Project CLAUDE.md template (used by /init-project)

project/
├── CLAUDE.md              ← Project-specific instructions (created by /init-project)
└── docs/records/*.md      ← Design decisions, feature specs (created by /design)
```

### Module types

| Type | Source format | Installed to | Consumed by |
|------|-------------|-------------|-------------|
| Commands | `commands/*.md` | `~/.claude/commands/` | Claude on `/command` |
| Skills | `skills/*/SKILL.md` | `~/.claude/skills/*/` | Claude via auto-loading (tech stack match or file extension) |
| MCP servers | `mcp/*.json` | `~/.claude.json` (merged) | Claude Code MCP client |
| Templates | `templates/*.md` | `~/.claude/templates/` | `/init-project`, `/catchup` (migration) |

### Content versioning

`templates/VERSION` tracks all managed content. `<!-- project-template: N -->` in `templates/project-CLAUDE.md` tracks template structure separately. The installer compares installed vs available versions for `--update`.

---

## Files

```
claude-code-setup/
├── .github/
│   ├── workflows/test.yml
│   ├── ISSUE_TEMPLATE/{bug_report,feature_request}.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b33eep/claude-code-setup](https://github.com/b33eep/claude-code-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
