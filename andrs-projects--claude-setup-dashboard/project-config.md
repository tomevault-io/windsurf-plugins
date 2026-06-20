---
trigger: always_on
description: Create or refresh the Claude Capabilities Dashboard. Scans agents, skills, hooks, and MCP servers from the live system and writes both the .md source and the styled HTML file. Works on first run (no existing files) or as an update.
---


# Dashboard Refresh

Create or update the Claude Code Capabilities Dashboard to reflect the current state of your setup.

## When to Use

Run `/dashboard-refresh` when:
- You want to create the dashboard for the first time (no files needed)
- You installed a new plugin, agent, or skill
- You added or removed MCP servers or hooks
- You added files to `~/.claude/knowledge/`

## Target Files

| File | Role |
|---|---|
| `~/Desktop/Claude Capabilities Dashboard/claude-code-capabilities.md` | Source of truth — written first |
| `~/Desktop/Claude Capabilities Dashboard/claude-code-capabilities-styled_1.html` | Styled HTML view |

---

## Step 1 — Detect mode: Create vs Refresh

Check whether the target files exist:

```
~/.../Claude Capabilities Dashboard/claude-code-capabilities.md
~/.../Claude Capabilities Dashboard/claude-code-capabilities-styled_1.html
```

- **Both exist** → Refresh mode: read the current `.md`, diff against live system, update only what changed
- **Missing or first run** → Create mode: scan everything from scratch, write both files fresh

If the directory doesn't exist, note this — you'll need to create it with the Write tool (which creates parent dirs automatically).

---

## Step 2 — Gather live system state

Scan these sources:

### Agents
- Custom agents: read all `~/.claude/agents/*.md` — extract `name` and `description` from frontmatter
- Plugin agents: read from the system-reminder "Available agent types" list. Group by plugin prefix (`everything-claude-code:`, etc.). Strip the prefix for the display name.

### Skills
- Custom skills: read all `~/.claude/skills/*/SKILL.md` — extract `name` and `description` from frontmatter
- Built-in skills: list skills in the system-reminder that have no plugin prefix (e.g. `update-config`, `loop`, `schedule`, `simplify`, `keybindings-help`, `claude-api`)
- Plugin skills: skills in the system-reminder with a plugin prefix. Group by prefix.

### Hooks
- Read `~/.claude/settings.json` — look for the `hooks` key
- Also check `~/.claude/settings.local.json` if it exists
- For each hook, note: event type (PreToolUse, PostToolUse, PreCompact, SessionStart, Stop, SessionEnd, UserPromptSubmit), matcher/trigger, and description

### MCP Servers
- Local servers: read `~/.claude/settings.json` → `mcpServers` key. These go under "Core · Local (Developer Settings)"
- claude.ai connectors: infer from `mcp__claude_ai_<Name>__*` tool names in the system-reminder. Each unique `<Name>` is a connector. These go under "Core · claude.ai Connected Connectors"
- Plugin MCPs: look for `mcp__plugin_<plugin>_<server>__*` tool prefixes in the system-reminder

### Knowledge Library (optional)
- Check if `~/.claude/knowledge/` exists
- If it does: list all `.md` files excluding `README.md`
- For each file, extract: the filename (slug) and the first `# ` heading as the title
- If the folder does not exist, skip this section entirely — do not create an empty section

---

## Step 3 — Write the .md file

Write `claude-code-capabilities.md` using today's date. Structure:

```markdown
# Claude Code Capabilities
> Generated: YYYY-MM-DD

---

## Agents

### Core (Custom)
| Agent | Description |
|---|---|
| `name` | description |

### Plugin: everything-claude-code (v1.9.0)
| Agent | Description |
|---|---|
...

> Invoke agents with `/agent:<name>` or via the Agent tool in conversation.

---

## Skills

### Core (Built-in)
| Skill | Description |
|---|---|
...

### Core (Custom)
| Skill | Description |
|---|---|
...

### Plugin: claude-mem (v10.6.3)
| Skill | Description |
|---|---|
...

### Plugin: everything-claude-code (v1.9.0)
#### Core Workflow
| Skill | Description |
...

> Invoke with `/skill-name` or `/plugin:skill-name`

---

## Hooks

### Plugin: everything-claude-code (v1.9.0)

#### PreToolUse
| Hook | Trigger | Description |
|---|---|---|
...

#### PostToolUse
...

#### Stop
...

#### SessionStart / SessionEnd / PreCompact
...

---

### Plugin: claude-mem (v10.6.3)
...

---

## MCP Servers

### Core (Local — Developer Settings)
| Name | Description |
|---|---|
...

### Core (claude.ai Connected Connectors)
| Name | Description |
|---|---|
...

### Plugin: everything-claude-code (v1.9.0)
| MCP Server | Key Tools |
|---|---|
...

## Knowledge Library
> Only include this section if `~/.claude/knowledge/` exists and contains .md files.

| File | Title |
|---|---|
| `filename` | First # heading from the file |

> Location: `~/.claude/knowledge/`
```

---

## Step 4 — Write the HTML file

Write the full HTML file. In **Refresh mode**, preserve the existing CSS/JS and only replace the `<div class="wrapper">...</div>` block. In **Create mode**, write the complete file from the template below.

### HTML Template

The complete file structure to use (or preserve) for the HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Claude Code Capabilities</title>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANDRS-Projects/claude-setup-dashboard](https://github.com/ANDRS-Projects/claude-setup-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
