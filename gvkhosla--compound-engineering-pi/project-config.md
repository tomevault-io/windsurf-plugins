---
trigger: always_on
description: These instructions apply when working under `plugins/compound-engineering/`.
---

# Plugin Instructions

These instructions apply when working under `plugins/compound-engineering/`.
They supplement the repo-root `AGENTS.md`.

# Compounding Engineering Plugin Development

## Versioning Requirements

**IMPORTANT**: Routine PRs should not cut releases for this plugin.

The repo uses an automated release process to prepare plugin releases, including version selection and changelog generation. Because multiple PRs may merge before the next release, contributors cannot know the final released version from within an individual PR.

### Contributor Rules

- Do **not** manually bump `.claude-plugin/plugin.json` version in a normal feature PR.
- Do **not** manually bump `.claude-plugin/marketplace.json` plugin version in a normal feature PR.
- Do **not** cut a release section in the canonical root `CHANGELOG.md` for a normal feature PR.
- Do update substantive docs that are part of the actual change, such as `README.md`, component tables, usage instructions, or counts when they would otherwise become inaccurate.

### Pre-Commit Checklist

Before committing ANY changes:

- [ ] No manual release-version bump in `.claude-plugin/plugin.json`
- [ ] No manual release-version bump in `.claude-plugin/marketplace.json`
- [ ] No manual release entry added to the root `CHANGELOG.md`
- [ ] README.md component counts verified
- [ ] README.md tables accurate (agents, commands, skills)
- [ ] plugin.json description matches current counts

### Directory Structure

```
agents/
├── review/           # Code review agents
├── document-review/  # Plan and requirements document review agents
├── research/         # Research and analysis agents
├── design/           # Design and UI agents
└── docs/             # Documentation agents

skills/
├── ce-*/          # Core workflow skills (ce:plan, ce:review, etc.)
└── */             # All other skills
```

> **Note:** Commands were migrated to skills in v2.39.0. All former
> `/command-name` slash commands now live under `skills/command-name/SKILL.md`
> and work identically in Claude Code. Other targets may convert or map these references differently.

## Command Naming Convention

**Workflow commands** use `ce:` prefix to unambiguously identify them as compound-engineering commands:
- `/ce:brainstorm` - Explore requirements and approaches before planning
- `/ce:plan` - Create implementation plans
- `/ce:review` - Run comprehensive code reviews
- `/ce:work` - Execute work items systematically
- `/ce:compound` - Document solved problems

**Why `ce:`?** Claude Code has built-in `/plan` and `/review` commands. The `ce:` namespace (short for compound-engineering) makes it immediately clear these commands belong to this plugin.

## Skill Compliance Checklist

When adding or modifying skills, verify compliance with the skill spec:

### YAML Frontmatter (Required)

- [ ] `name:` present and matches directory name (lowercase-with-hyphens)
- [ ] `description:` present and describes **what it does and when to use it** (per official spec: "Explains code with diagrams. Use when exploring how code works.")

### Reference Links (Required if references/ exists)

- [ ] All files in `references/` are linked as `[filename.md](./references/filename.md)`
- [ ] All files in `assets/` are linked as `[filename](./assets/filename)`
- [ ] All files in `scripts/` are linked as `[filename](./scripts/filename)`
- [ ] No bare backtick references like `` `references/file.md` `` - use proper markdown links

### Writing Style

- [ ] Use imperative/infinitive form (verb-first instructions)
- [ ] Avoid second person ("you should") - use objective language ("To accomplish X, do Y")

### Cross-Platform User Interaction

- [ ] When a skill needs to ask the user a question, instruct use of the platform's blocking question tool and name the known equivalents (`AskUserQuestion` in Claude Code, `request_user_input` in Codex, `ask_user` in Gemini)
- [ ] Include a fallback for environments without a question tool (e.g., present numbered options and wait for the user's reply before proceeding)

### Cross-Platform Task Tracking

- [ ] When a skill needs to create or track tasks, describe the intent (e.g., "create a task list") and name the known equivalents (`TaskCreate`/`TaskUpdate`/`TaskList` in Claude Code, `update_plan` in Codex)
- [ ] Do not reference `TodoWrite` or `TodoRead` — these are legacy Claude Code tools replaced by `TaskCreate`/`TaskUpdate`/`TaskList`
- [ ] When a skill dispatches sub-agents, prefer parallel execution but include a sequential fallback for platforms that do not support parallel dispatch

### Script Path References in Skills

- [ ] In bash code blocks, reference co-located scripts using relative paths (e.g., `bash scripts/my-script ARG`) — not `${CLAUDE_PLUGIN_ROOT}` or other platform-specific variables
- [ ] All platforms resolve script paths relative to the skill's directory; no env var prefix is needed
- [ ] Always also include a markdown link to the script (e.g., `[scripts/my-script](scripts/my-script)`) so the agent can locate and read it

### Cross-Platform Reference Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gvkhosla/compound-engineering-pi](https://github.com/gvkhosla/compound-engineering-pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
