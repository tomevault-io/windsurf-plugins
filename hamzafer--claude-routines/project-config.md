---
trigger: always_on
description: This repo manages Claude Code Routines as code. Each routine is a `.md` file with YAML frontmatter (config) and a markdown body (the prompt). When the user asks you to operate on a routine, follow the instructions below.
---

# claude-routines — operational instructions for Claude Code

This repo manages Claude Code Routines as code. Each routine is a `.md` file with YAML frontmatter (config) and a markdown body (the prompt). When the user asks you to operate on a routine, follow the instructions below.

You execute these operations in-process via the `RemoteTrigger` skill that ships with Claude Code. Auth is handled automatically — do not look for tokens or env vars.

---

## When the user says…

- **"deploy `<file>`"** or "push `<file>`" → if the file's frontmatter has a `trigger_id`, do **update**; else do **create**.
- **"create `<file>`"** → strict create. Frontmatter must NOT have `trigger_id`. Add the returned `trigger_id` to the file's frontmatter after success.
- **"update `<file>`"** → strict update. Frontmatter must have `trigger_id`. Use the read-modify-write protocol.
- **"list"** or "show my routines" → list action.
- **"pull"** → list, then get each, write all to `routines/`.
- **"get `<trigger_id>`"** → fetch one, write to `routines/<slug>.md`.
- **"run `<trigger_id>`"** → fire the routine now.
- **"delete `<trigger_id>`"** → tell the user this isn't supported via API; they need to delete via web UI at https://claude.ai/code/routines.
- **"validate `<file>`"** or "lint `<file>`" → check the file against the rules in [Validate](#validate) below. Don't call the API. Report errors with line numbers when possible.
- **"validate"** with no file → validate every `.md` file under `routines/` and `personal/` (excluding `personal/snippets/` and READMEs). Print a one-line summary per file.
- **"deploy all"** / "deploy everything in `<dir>`" / similar bulk → see [Bulk operations](#bulk-operations) below.
- **"diff `<file>`"** or "what's changed in `<file>`" → see [Diff](#diff) below.
- **"orphans"** or "find orphans" or "list orphans" → list local routine files whose `trigger_id` no longer exists in the cloud (probably deleted via web UI). Pure-local check after a `RemoteTrigger.list`. Print one line per orphan: `<file path> → <trigger_id> (not in cloud)`. End with a count. Don't delete the files — that's the user's call.
- **"dry-run deploy `<file>`"** / "preview deploy `<file>`" / "what would `deploy <file>` send" → build the API body exactly as `deploy` would (including read-modify-write merge for an update), but **do not call the API**. Show the user the body that would be sent (formatted JSON, pretty-printed) plus a one-line summary of which operation (`create` vs `update`) and which fields would change vs. live state. Useful pairs: dry-run + diff for full pre-deploy confidence.

---

## Frontmatter spec (full reference: `docs/reference.md`)

```yaml
---
trigger_id: trig_01ABC...     # presence = update; absence = create
name: "Display name"          # required

# Trigger — exactly one of:
cron: "0 8 * * *"             # UTC, minimum 1-hour interval
# run_once_at: "2027-01-01T00:00:00Z"

enabled: true                 # default true; false = scheduled triggers paused, manual run still works
env_id: env_01...             # required; cloud environment ID
model: claude-sonnet-4-6      # optional; one of claude-opus-4-7, claude-opus-4-7[1m], claude-sonnet-4-6, claude-haiku-4-5
allowed_tools: [Bash, Read, Write, Edit, Glob, Grep, WebFetch, WebSearch]
sources:                      # optional, repos cloned at session start
  - url: https://github.com/owner/repo
    allow_unrestricted_git_push: false
mcp_connections:              # optional
  - connector_uuid: <v4 uuid>
    name: <name>
    url: <url>
    permitted_tools: []
---

(prompt body — sent verbatim as the routine's user message)
```

---

## How to call `RemoteTrigger`

`RemoteTrigger` accepts `action` plus an optional `trigger_id` and `body`. Available actions: `list`, `get`, `create`, `update`, `run`. There is no `delete`.

### create

```
RemoteTrigger({
  action: "create",
  body: {
    name: <frontmatter.name>,
    cron_expression: <frontmatter.cron>,         // OR run_once_at: <frontmatter.run_once_at>
    enabled: <frontmatter.enabled ?? true>,
    job_config: {
      ccr: {
        environment_id: <frontmatter.env_id>,
        events: [{
          data: {
            uuid: <generated lowercase v4 uuid>,
            session_id: "",
            type: "user",
            parent_tool_use_id: null,
            message: { content: <prompt body>, role: "user" }
          }
        }],
        session_context: {
          allowed_tools: <frontmatter.allowed_tools>,
          model: <frontmatter.model>,            // omit if not in frontmatter
          sources: <frontmatter.sources>         // omit if not in frontmatter
        }
      }
    },
    mcp_connections: <frontmatter.mcp_connections>  // omit if not in frontmatter
  }
})
```

After success, write `trigger_id` from the response back into the file's frontmatter.

### update — read-modify-write (MANDATORY)

The API resets missing nested fields inside `job_config` to maximally-permissive defaults (see `docs/superpowers/specs/2026-04-26-claude-routines/03-update-safety.md`). To update safely:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hamzafer/claude-routines](https://github.com/hamzafer/claude-routines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
