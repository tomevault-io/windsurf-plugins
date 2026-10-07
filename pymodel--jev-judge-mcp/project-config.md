---
trigger: always_on
description: `jev-judge-mcp install` registers the MCP server under the name `jev` in `~/.claude.json` (ADR-0033). Tool names in Claude Code are `mcp__jev__*`. The installer does not write permission allow-rules, and it does not merge or enable the command hook.
---

# Claude Code

`jev-judge-mcp install` registers the MCP server under the name `jev` in `~/.claude.json` (ADR-0033). Tool names in Claude Code are `mcp__jev__*`. The installer does not write permission allow-rules, and it does not merge or enable the command hook.

The command hook is not the `jev_gate` MCP tool, which stays the completion gate (ADR-0035).

## Allow-rules

Headless runs need an allow-rule. `acceptEdits` covers file edits. Add the published tools, or the one server rule `mcp__jev`, in `~/.claude/settings.json` or a project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__jev__jev_verify",
      "mcp__jev__jev_screen",
      "mcp__jev__jev_find",
      "mcp__jev__jev_classify",
      "mcp__jev__jev_decide",
      "mcp__jev__jev_rerank",
      "mcp__jev__jev_compare",
      "mcp__jev__jev_extract",
      "mcp__jev__jev_review",
      "mcp__jev__jev_gate",
      "mcp__jev__jev_score",
      "mcp__jev__jev_file_judge",
      "mcp__jev__jev_ask",
      "mcp__jev__jev_files_judge"
    ]
  }
}
```

`mcp__jev` allows the whole server in one rule. The installer does not write either form.

`jev_ask` is safe to allow because its `command` argument is off unless the operator sets
`JEV_ASK_COMMANDS=1` in the server environment: enabling it lets the server run what Jev judges
read-only without a harness prompt, so decide that in the server's launch configuration, not in a
permission file. Enabling it is not containment: the denylist blocks named clients such as `curl`
and `ssh`, but an interpreter one-liner like `python -c` or `node -e` passes the denylist and can
still reach the network. Run the server under your own sandbox when command isolation matters.

## Routing skill

Which tool fits a step is `docs/skills/jev-mcp/SKILL.md`. Building an app on the Jev API, rather than calling these tools, is `src/jev_judge_mcp/skills/jev/SKILL.md`. A connected client reads them at `jev-skill://jev-mcp/SKILL.md` and `jev-skill://jev/SKILL.md`. Do not copy the `jev` skill's cookbook thresholds onto these tools; this server's defaults are `docs/reference/limits.md`. The on-demand rule is in `docs/agent-rules.md`. This repository ships no plugin manifest. A plugin entry is appropriate only when it points at one of those skills and embeds no machine path and no key.

## Reading deduplicated ids

`jev_find`, `jev_verify`, and `jev_gate` sanitize and de-duplicate caller ids. When the same id is sent on several items, the first occurrence in caller order keeps the sent id and later occurrences get `_1`, `_2`, …; an unsuffixed id in a row therefore names the first physical item sent under that id. The payload's `renamed_ids` maps the sent id to the returned id of its last renamed occurrence (ADR-0062).

## Command hook

The hook is opt-in. It can deny or ask, and it can never allow. Empty stdout means the hook abstained and Claude Code keeps its own permission flow. Any other stdout is one JSON object whose `permissionDecision` is `deny` or `ask`.

The judgment is routed by the event's tool name (ADR-0076). A `Bash` action is judged by two questions in one provider call: what the command would do to material in the working directory (`read_only`, `reversible`, or `irreversible`), and whether destroying work is its purpose. A `Write` or `Edit` gets two checks computed in code first: a target outside the working directory is denied before the provider is even constructed, and so is content carrying a credential literal — a well-known token format, or a quoted high-entropy value assigned to a secret-named key. Ordinary code is never a hit and is judged as written. What survives both is judged by one credential-in-content question. Any other tool name keeps the generic allow/deny question. Every deny reason names the failure and its confidence or probability, and ends with a final-block notice telling the agent not to retry, split, or reroute the action.

`jev-judge-mcp install` does not merge [`gate.hooks.json`](gate.hooks.json) and does not enable it. To turn the hook on, merge that fragment's `hooks` object into `~/.claude/settings.json` or a project's `.claude/settings.json`. The sample command is `/absolute/path/to/jev-judge-mcp hook gate`. Real settings need the absolute path of the `jev-judge-mcp` executable, because the hook's working directory is the project. Do not use `uv run --directory`: that flag changes the working directory, so the hook judges the checkout instead of the caller's repo. `uv run --project <repo>` keeps the caller's cwd. The matcher is `Bash|Write|Edit` and the timeout is 30 seconds, the same budget `src/jev_judge_mcp/hook.py` passes to the provider call.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyModel/jev-judge-mcp](https://github.com/PyModel/jev-judge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
