---
trigger: always_on
description: Hooks that make Claude Code check its own work, rulesets that teach every agent what this project flags, and an MCP server they can call on demand.
---

# Coding agents

Hooks that make Claude Code check its own work, rulesets that teach every agent what this project flags, and an MCP server they can call on demand.

[← Back to the README](../README.md) · [All documentation](README.md)

## Let the agent check itself

A ruleset tells the agent what to avoid, and the MCP server lets it check when
it remembers to. Neither makes it check. Hooks do: Claude Code runs Sloppy
itself, after every edit and before the agent is allowed to call the task
done, whether or not the model thought to ask.

```bash
vendor/bin/sloppy agents install          # or: php artisan sloppy:agents
vendor/bin/sloppy agents install --local  # .claude/settings.local.json, out of git
vendor/bin/sloppy agents install --dry-run
```

That adds two hooks to `.claude/settings.json` and writes the ruleset into
`CLAUDE.md` as [`sloppy:rules`](#rules-for-coding-agents) would:

| When | What runs | What the agent sees |
| --- | --- | --- |
| After every `Edit`, `Write` or `MultiEdit` of a PHP file | `sloppy hook post-edit` | Findings *that edit* introduced in *that file*, compared with `HEAD`, while the code is still in front of it |
| When the agent tries to finish | `sloppy hook stop` | New findings at or above `fail_on` across the whole change. The agent is sent back to fix them |

Only what the change introduced is ever reported. An agent editing one method
in a legacy file is not told about the twelve problems the file already had,
because an agent told about them goes and "fixes" code nobody asked it to touch.

Tests are covered too, although the analyser never scans them. When the agent
edits a test file, [`SL503`](rules.md#suppression) compares it with `HEAD`, so
a test skipped, stripped of assertions or given `assertTrue(true)` to make it
pass is fed back the same way. A method left as `// ... existing code ...` or
`throw new Exception('Not implemented')` is `SL112`.

The Stop hook blocks **once**. If the agent tries to finish again, Claude Code
marks the retry and Sloppy lets it through, listing whatever is left in the
transcript. A false positive costs one round trip, not the session. And every
way the hook could fail to run — no git, no commits yet, a broken config —
lets the agent carry on, with a note in the transcript. A broken analyser never
traps an agent.

What the model reads is plain text, capped at ten findings:

```text
Sloppy: this edit to app/Services/Billing.php introduced 1 finding.

- app/Services/Billing.php:42 SL107 Swallowed Exception (high): The catch block for Throwable discards the exception...
  Fix: Handle the failure or let it travel...
```

Installing is safe to repeat. Your permissions, environment and other hooks
are left exactly as they were. Sloppy's own entries are found by their command
and replaced in place, and a settings file that is not valid JSON is refused
rather than rewritten. In a project with Sloppy in `vendor/`, the hooks call
`php "${CLAUDE_PROJECT_DIR}/vendor/bin/sloppy"`, so the committed file works on
every checkout. With a global or phar install they call the binary that
installed them.

"New" means new compared with `HEAD`, so uncommitted work from before the
session counts as part of the change. That is the same rule `sloppy diff` uses.

## Rules for coding agents

The cheapest finding is the one that never gets written. `sloppy:rules` writes
this project's rules where the agents already look, before they write anything:

```bash
php artisan sloppy:rules                              # CLAUDE.md, or Boost's guidelines
php artisan sloppy:rules --format=cursor --format=agents
php artisan sloppy:rules --format=copilot --format=windsurf
php artisan sloppy:rules --stdout                     # print instead
```

Each format knows the file its tool reads, so `--output` is only for the
unusual case:

| `--format=` | Writes |
| --- | --- |
| `claude` (default) | `CLAUDE.md` |
| `boost` (default with Laravel Boost) | `.ai/guidelines/sloppy.blade.php` |
| `cursor` | `.cursorrules` |
| `agents` | `AGENTS.md` |
| `copilot` | `.github/copilot-instructions.md` |
| `windsurf` | `.windsurfrules` |
| `markdown` | `sloppy-rules.md` |
| `json` | `sloppy-rules.json` |

The Markdown formats merge into whatever is already there. `json` has no
marked block to merge into, so it refuses to overwrite an existing file unless
you pass `--force`.

The file describes *your* configuration -- your paths, your threshold, the
rules you turned off, the ones skipped because you do not use their framework
-- and gives each rule a line an agent can act on:

```markdown
### SL107 Swallowed Exception

- Category: Error handling, severity: high
- Flags: A catch block that discards the exception ...
- Why it costs: A swallowed exception turns a bug into a silent wrong answer ...
- Write it this way instead: Handle the failure or let it travel. If you catch,
  log the exception as the previous one and rethrow something meaningful --
  never return null in place of an answer.
```

Existing files are respected: the generated rules go inside a marked block,
appended the first time and replaced in place afterwards, with everything

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Heyosseus/sloppy](https://github.com/Heyosseus/sloppy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
