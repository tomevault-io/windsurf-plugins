---
trigger: always_on
description: This is the **meta-project**: the multi-agent orchestration framework itself. This CLAUDE.md governs work _on_ megavibe, not work _using_ megavibe (that's in `~/.claude/CLAUDE.md`).
---

# CLAUDE.md — Megavibe Framework

This is the **meta-project**: the multi-agent orchestration framework itself. This CLAUDE.md governs work _on_ megavibe, not work _using_ megavibe (that's in `~/.claude/CLAUDE.md`).

## Project identity

Megavibe is a bootstrapper + protocol for AI-assisted development. It is NOT a software application — it's config, shell scripts, and markdown that get deployed _onto_ other projects. The core thesis: externalize all context to files, regenerate working context from the full log via AI subcontractors (Gemini or Codex, with automatic fallback), never rely on lossy compaction.

**There is no build step, no test suite, no package.json.** Verification is manual: run the scripts, inspect the output, confirm idempotency.

## File map

| File | Purpose | Change risk |
|------|---------|-------------|
| `megavibe` | CLI wrapper — setup + init + launch Claude | High — primary entry point |
| `install.sh` | One-command cross-platform install (`curl \| bash`) | Medium — bootstrap only |
| `setup.sh` | One-time machine bootstrapper (tools, MCP, CLI, protocol) | High — affects all users |
| `init.sh` | Per-project bootstrapper (.agent/, hooks, skills, settings) | High — affects all projects |
| `telegram-bot.py` | Megavibe Remote v4: personal assistant + project session launcher via TG | Medium — remote access |
| `template/CLAUDE.md` | Core protocol (90 lines) — installed to `~/.claude/CLAUDE.md` | Critical — review required |
| `template/.claude/rules/spinouts.md` | Subtask spinout protocol | Medium — review recommended |
| `template/.claude/rules/delegation.md` | Tool routing + delegation protocols + selective compaction | Medium — review recommended |
| `template/.claude/skills/*/SKILL.md` | Slash command skills (rehydrate, catchup, prune-context) | Low — workflow shortcuts |
| `template/statusline.sh` | Context usage progress bar | Low |
| `template/.claude/settings.json` | Hook registrations template | Medium — when hooks change |
| `template/.claude/hooks/*.sh` | Hook scripts template (13 hooks; canonical list in init.sh) | Medium |
| `template/.claude/agents/summarizer.md` | Last-resort fallback agent (sonnet) | Low — rarely changes |
| `.claude/hooks/*.sh` | Live hooks for THIS repo (copied from template) | Should mirror template |
| `.claude/rules/*.md` | Live rules for THIS repo (copied from template) | Should mirror template |
| `.claude/agents/*.md` | Live agents for THIS repo (copied from template) | Should mirror template |
| `.agent/` | Live context for developing megavibe itself | Continuous |
| `README.md` | Full documentation | When features change |

## Critical invariants

1. **Idempotency is sacred.** Both `setup.sh` and `init.sh` must be safe to re-run. Every create/copy/install checks before writing. Test by running twice — second run must show all "skip" messages.

2. **Marker-based dedup and update.** `<!-- megavibe-v3 -->` and `<!-- /megavibe-v3 -->` bracket the protocol block in `template/CLAUDE.md`. The start marker prevents duplicate installs; the end marker enables `setup.sh` to surgically replace the block on re-run. Never remove or rename either marker. If bumping to v4, update both markers AND the grep patterns in `setup.sh`.

3. **Hook safety guards.** Every hook script must:
   - `[ -d ".agent" ] || exit 0` — no-op outside megavibe projects
   - `command -v jq &>/dev/null || exit 0` — graceful without jq
   - Never block Claude over infra issues (exit 0, not exit 2)
   - Exception: `block-dangerous-bash.sh` exits 2 intentionally

4. **Template/live parity.** `template/.claude/` and the repo's own `.claude/` should stay in sync. After editing a template hook, run `bash init.sh .` to sync the live copy.

5. **No project CLAUDE.md in template.** `init.sh` never touches a project's `CLAUDE.md`. The protocol lives user-level. Project CLAUDE.md is the user's domain.

## Verification protocol

Before committing changes to scripts or hooks:

```bash
# 1. Idempotency: run init.sh twice on a temp project
mkdir -p /tmp/mv-test && bash init.sh /tmp/mv-test
bash init.sh /tmp/mv-test  # hooks "synced", .agent/ files "skip"

# 2. Hook executability
ls -la /tmp/mv-test/.claude/hooks/*.sh  # all -rwxr-xr-x

# 3. Settings merge: pre-existing settings.json preserved
mkdir -p /tmp/mv-test2/.claude
echo '{"permissions":{"allow":["Read"]}}' > /tmp/mv-test2/.claude/settings.json
bash init.sh /tmp/mv-test2
# verify both original permissions AND megavibe hooks present

# 4. Marker detection
grep "megavibe-v3" ~/.claude/CLAUDE.md

# 5. Hook isolation: no-ops without .agent/
cd /tmp && echo '{}' | /path/to/hooks/log-tool-event.sh  # exits silently
```

For hook logic changes, test the specific scenario (counter nudge at 15 calls, rehydration flag set/clear, on-compact JSON output with special characters).

```bash
# 6. Update propagation: just re-run the scripts
bash setup.sh                 # always updates protocol + statusline
bash init.sh /path/to/project # always syncs hooks from template
```

## Shell scripting conventions

- `set -euo pipefail` at top of every script
- `$()` not backticks; `"$VAR"` not `$VAR`
- `[[ ]]` for bash conditionals

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poma-ai/megavibe](https://github.com/poma-ai/megavibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
