---
trigger: always_on
description: Workflow guidelines for AI agents making changes inside `ai_coding_project_base/`.
---

# AGENTS.md (Toolkit Repo)

Workflow guidelines for AI agents making changes inside `ai_coding_project_base/`.

This repository is a **toolkit** (prompts, slash commands, skills). It is not a
“target app” that you execute phases against.

## Repo Context

- **Primary artifacts:** Markdown prompt templates and Claude Code skills.
- **Main directories:**
  - `.claude/skills/` — Skills (`SKILL.md`) — these create `/slash-commands`
  - `.claude/commands/` — Legacy command format (still works, but prefer skills)
  - `deprecated/` — Legacy/reference-only prompt files (avoid editing unless required)
  - `docs/` — Static documentation site assets

**Note:** Skills and commands are now merged in Claude Code. A file at `.claude/skills/review/SKILL.md`
creates `/review`. The `.claude/commands/` format still works but skills are preferred for new work.

## Operating Principles

- **Be conservative:** Prefer small, targeted edits over rewrites.
- **Keep behavior compatible:** Existing command names and core workflows should not
  break without a strong reason and explicit documentation.
- **Docs are code:** Treat prompt/command changes like API changes—update related
  docs when behavior or outputs change.
- **Avoid speculation:** If requirements are unclear, ask the user rather than
  encoding assumptions into prompts.

## Plan Review Protocol

After writing a plan in plan mode, use AskUserQuestion **before** calling
ExitPlanMode:

- "Ready to implement (Recommended)" → Call ExitPlanMode
- "Review with /codex-consult first" → Call ExitPlanMode (Skill/Bash tools
  are unavailable in plan mode, so you must exit first). After the user
  approves, **before doing any implementation work**, save the plan to a
  file if not already on disk, then run `/codex-consult <plan-file>`.
  Present the findings and use AskUserQuestion to confirm whether to
  proceed with implementation or revise the plan.
- "I want to modify the plan" → Stay in plan mode, ask what to change

Do NOT call ExitPlanMode without offering these options first.

## Editing Rules (Markdown / Prompts)

- Preserve existing structure, headings, and code fences unless intentionally changing
  behavior.
- When adding examples, keep them minimal and copy-pastable.
- Avoid introducing new long, duplicated sections—prefer referencing an existing
  command/skill or consolidating.

## Skills (`.claude/skills/*/SKILL.md` and `.claude/commands/*.md`)

Skills and commands are merged in Claude Code. Both create `/slash-commands`:
- `.claude/skills/foo/SKILL.md` → `/foo`
- `.claude/commands/foo.md` → `/foo`

**Prefer skills format** for new work (supports supporting files, better discovery).

Guidelines:
- Keep the YAML frontmatter valid (`name`, `description`, `argument-hint`, `allowed-tools`).
- Use `toolkit-only: true` in frontmatter for skills that should NOT sync to target
  projects (e.g., `setup`, `update-target-projects`). All sync surfaces (setup,
  update-target-projects, sync, install-codex-skill-pack.sh) check this flag.
- Keep "Directory Guard" sections accurate; skills should fail fast when run in the
  wrong directory.
- Prefer general, reusable workflows (avoid product-specific rules).
- If a skill references additional assets, link them explicitly.
- If you add/rename a skill:
  - Update `docs/commands.md` (command reference) accordingly.
  - Ensure the skill aligns with constraints in `.claude/settings.json`.

## Global Skill Resolution

The toolkit now uses a global-only policy for skills. Projects should resolve
skills from `~/.claude/skills/` and should not keep long-lived project-local
copies in `.claude/skills/`.

Claude Code discovers skills from three tiers with project-local shadowing global:

1. **managed** (`/Library/Application Support/ClaudeCode/.claude/skills`)
2. **user** (`~/.claude/skills`) ← toolkit symlinks go here
3. **project** (`.claude/skills/`) ← shadows user/managed if present

**Resolution modes:**

| Mode | Description | When Used |
|------|-------------|-----------|
| `global` | All skills resolve via `~/.claude/skills/` | Required default for toolkit-managed projects |
| `local` | All skills copied to project `.claude/skills/` | Legacy state to migrate away from |
| `mixed` | Some global, some local | Legacy state to migrate away from |

**Behavior by project type:**

- **New projects:** `/setup` verifies global symlinks, then records global
  resolution without copying skills into the project.
- **Existing projects:** `/update-target-projects` should migrate legacy local
  or mixed projects to global resolution by default.
- **Shared repos:** Also use global resolution. Portability is handled by each
  collaborator bootstrapping `~/.claude/skills` from their own toolkit clone.

**Configuration (`toolkit-version.json`):**

```json
{
  "force_local_skills": false,
  "skill_resolution": "global"
}
```

- `force_local_skills`: legacy field; keep `false` for global-only behavior
- `skill_resolution`: expected value is `global` for toolkit-managed projects

**Migration:**

- **Adopt global:** `/update-target-projects` removes local copies, backs up
  modified skills, and switches to global resolution.
- **Legacy local projects:** treat as migration debt and convert during normal
  sync runs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benjaminshoemaker/ai_coding_project_base](https://github.com/benjaminshoemaker/ai_coding_project_base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
