---
trigger: always_on
description: > Curated plugin marketplace for AI coding agents.
---

# n-skills

> Curated plugin marketplace for AI coding agents.
> https://github.com/numman-ali/n-skills

This repository contains high-quality, curated skills for AI coding agents.
Skills follow the [agentskills.io](https://agentskills.io) SKILL.md format.

## Installation

```bash
npm i -g openskills
openskills install numman-ali/n-skills
openskills sync
```

## Usage

When a user asks you to perform a task, check if any available skill can help.
Invoke skills using: `openskills read <skill-name>`

<available_skills>

<skill>
<name>zai-cli</name>
<description>Z.AI vision, search, reader, and GitHub exploration via MCP</description>
<location>skills/tools/zai-cli</location>
<invoke>openskills read zai-cli</invoke>
</skill>

<skill>
<name>dev-browser</name>
<description>Browser automation with persistent page state. Use when users ask to navigate websites, fill forms, take screenshots, extract web data, test web apps, or automate browser workflows.</description>
<location>skills/automation/dev-browser</location>
<invoke>openskills read dev-browser</invoke>
</skill>

<skill>
<name>gastown</name>
<description>Multi-agent orchestrator for Claude Code. Use when user mentions gastown, gas town, gt commands, convoys, polecats, rigs, slinging work, multi-agent coordination, beads, hooks, the witness, the mayor, the refinery, or wants to run multiple AI agents on projects simultaneously.</description>
<location>skills/tools/gastown</location>
<invoke>openskills read gastown</invoke>
</skill>

<skill>
<name>orchestration</name>
<description>Multi-agent orchestration for complex tasks using cc-mirror tasks and TodoWrite. Use when tasks require parallel work, multiple agents, sophisticated coordination, or decomposition into parallel subtasks. Triggers include features, reviews, refactoring, testing, documentation, or any work benefiting from parallel execution.</description>
<location>skills/workflow/orchestration</location>
<invoke>openskills read orchestration</invoke>
</skill>

<skill>
<name>open-source-maintainer</name>
<description>End-to-end GitHub repository maintenance for open-source projects. Use when asked to triage issues, review PRs, analyze contributor activity, generate maintenance reports, or maintain a repository. Triggers include "triage", "maintain", "review PRs", "analyze issues", "repo maintenance", "what needs attention", "open source maintenance", or any request to understand and act on GitHub issues/PRs.</description>
<location>skills/workflow/open-source-maintainer</location>
<invoke>openskills read open-source-maintainer</invoke>
</skill>

</available_skills>

## Categories

### tools
- **zai-cli**: Z.AI vision, search, reader, and GitHub exploration via MCP
- **gastown**: Multi-agent orchestrator for Claude Code

### automation
- **dev-browser**: Browser automation with persistent page state

### workflow
- **orchestration**: Multi-agent orchestration for complex tasks using cc-mirror tasks and TodoWrite
- **open-source-maintainer**: End-to-end GitHub repository maintenance for open-source projects

## Attribution

Skills may be sourced from external repositories. Check each skill's `.source.json` for attribution.

---

## Maintainer Notes: Plugin Structure

Each skill must be a **separate entry** in the `plugins` array in `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "skill-name",
      "description": "...",
      "source": "./skills/category/skill-name",
      "category": "workflow",
      "author": {
        "name": "Author Name",
        "github": "github-username"
      }
    }
  ]
}
```

**Plugin folder structure (matches official Claude Code format):**
```
skills/category/skill-name/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
└── skills/
    └── skill-name/           # Skill subfolder (required!)
        ├── SKILL.md          # Skill content
        └── references/       # Optional supporting docs
```

**plugin.json template:**
```json
{
  "name": "skill-name",
  "description": "...",
  "version": "1.0.0",
  "author": {
    "name": "Author Name",
    "url": "https://github.com/username"
  },
  "repository": "https://github.com/repo",
  "license": "Apache-2.0",
  "keywords": ["keyword1", "keyword2"]
}
```

**Critical rules:**
- Each skill is a separate plugin entry (NOT a nested `skills` array)
- `source` points to the skill folder path
- SKILL.md must be inside `skills/<skill-name>/` subfolder (Claude Code auto-discovery)
- **NO `$schema` key** — triggers Claude Code impersonation validation error

---

## Maintainer Notes: Releases

When making significant updates, create a release with tags and notes.

### Versioning Scheme

```
v1.MAJOR.MINOR (we're past v0.x now)

MAJOR: New skills, breaking changes, significant skill updates
MINOR: Bug fixes, documentation updates, small improvements
```

The `marketplace.json` version should match the release version.

### Release Process

```bash
# 1. Update marketplace.json version to match release
# 2. Commit changes
git add .
git commit -m "Description of changes"
git push origin main

# 3. Create annotated tag
git tag -a v1.X.Y -m "Brief description"
git push origin v1.X.Y

# 4. Create GitHub release with notes
gh release create v1.X.Y --title "v1.X.Y - Title" --notes "$(cat <<'EOF'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numman-ali/n-skills](https://github.com/numman-ali/n-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
