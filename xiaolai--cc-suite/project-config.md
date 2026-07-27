---
trigger: always_on
description: Project instructions for cc-suite — the Claude Code plugin that bridges Claude Code, Codex CLI, and Antigravity CLI (agy) with single-source AGENTS.md, shared skills, mirrored hooks, and bidirectional MCP delegation; adds a Claude→Grok delegation lane (ACP) and opt-in MCP bridging to more coding agents (Grok Build, opencode, Qwen Code, Kimi CLI).
---


# Project Instructions

> cc-suite

## Guidelines

- Bump the version in `.claude-plugin/plugin.json` for every release (patch/minor/major per semver). `package.json` must carry the same version — a test enforces it.
- **`main` always equals the newest tag.** Every commit pushed to `main` is a release: bump, commit, `git tag -a vX.Y.Z`, push both. Do not land a change and leave it untagged "until the next real release" — that includes docs-only and `dev-docs/` changes, which take a patch bump. If `git describe --exact-match HEAD` fails, the release is unfinished.
- New commands go in `commands/`; new skills go in `skills/cc-suite/<name>/SKILL.md`.
- All scripts in `scripts/` must be idempotent — running twice must produce the same result.
- Write new project-level instructions into `AGENTS.md` only; never edit `CLAUDE.md` or legacy `GEMINI.md` directly.

## Prerequisites

- **Claude Code** (≥ 2.0) — primary host for all commands and skills
- **Codex CLI** (optional) — required for the `codex-cli` MCP delegation lane and `bridge_hooks.py`
- **Python 3** — required by `scripts/bridge_hooks.py`, MCP projections, and migration helpers
- **Antigravity CLI (`agy`)** (optional) — required for the Google backend, `/cc-suite:agy-preflight`, and headless agy delegation
- **Grok Build (`grok`, xAI)** (optional) — required for the Claude→Grok ACP delegation lane (`/cc-suite:grok`) and `/cc-suite:grok-preflight`
- **Bash** — required by all `scripts/*.sh` files
- **`claude-octopus`** (npm, pinned) — the MCP server cc-suite registers in `.codex/config.toml` so Codex can delegate to Claude. cc-suite does not install it explicitly; `mcp_claude.sh` writes a `npx -y claude-octopus@<pin>` invocation and npm fetches it on first Codex start. The pin lives in `scripts/lib/claude-octopus-pin.txt` — single source of truth, read by `mcp_claude.sh`, the integration suite, and the boot-handshake test.

### Coordinating the claude-octopus pin

When claude-octopus ships a new version that cc-suite should adopt:

1. Edit `scripts/lib/claude-octopus-pin.txt` (one line, just the version).
2. Run `bash tests/integration.sh` — T39 actually boots the new pin and exchanges one MCP `initialize` to verify it works.
3. Bump cc-suite's own version per the normal release workflow.

Users get the new pin when they run `claude plugin update cc-suite@xiaolai` followed by `/cc-suite:update` — the second command re-renders the `.codex/config.toml` block in place (the pre-existing block won't be silently preserved because refresh-aware `mcp_claude.sh` detects the pin mismatch and rewrites).

## Smoke Test

After any setup change, run `/cc-suite:status` and confirm every bridge artifact shows `✓`.

## Build / Run

cc-suite is a Claude Code plugin — there is no compilation step. During development, exercise it as follows:

```bash
# Install for the current project (one-time)
claude plugin install cc-suite@xiaolai --scope project

# Refresh the bridge layer in a target project after editing any script
bash "${CLAUDE_PLUGIN_ROOT}/scripts/init.sh"

# Run cc-suite commands from the project root
# e.g. /cc-suite:status, /cc-suite:diagnose, /cc-suite:repair, /cc-suite:audit-fix
```

## Tests

Run the integration suite (72 test sections, 386 assertions):

```bash
bash tests/integration.sh
```

Tests cover every `scripts/*.sh`, the Codex and Antigravity MCP projection paths, the multi-tool bridge (`bridge_tools.py` — grok/opencode/qwen/kimi emitters, selection, unbridge), the grok delegation runner and preflight (`grok-runner.mjs`, `grok-preflight.sh`), legacy Gemini cleanup, `status.sh` output, refresh semantics for `mcp_claude.sh`, a real boot-and-handshake against the pinned `claude-octopus` (network-dependent — set `CC_SUITE_SKIP_BOOT_TEST=1` to skip), and the advisor-agent subsystem. Add a new `T<N>` section for any new behavior; the suite uses `make_tmp` / `cleanup` / `assert_*` helpers and tallies pass/fail counts in its summary. Command-file frontmatter is validated separately by `tests/commands.test.mjs` (`node --test`).

## Shared Memory

**Always write new instructions, rules, and memory to `AGENTS.md` only.**

Never modify `CLAUDE.md` or legacy `GEMINI.md` directly — they only import `AGENTS.md`.
This keeps Claude Code, Codex CLI, and Antigravity CLI (`agy`) on the same context.

## Project Structure

- `.claude/` — Claude Code skills, agents, rules, hooks, commands
- `.agents/skills/` — symlink to `.claude/skills/` (Codex skill scan path)
- `.agents/mcp_config.json` — generated Antigravity workspace MCP projection (ignored by default)
- `.codex/prompts/` — Codex slash-command prompts
- `.codex/hooks.json` / `.codex/config.toml` — Codex hooks/config (optional)
- `.mcp.json` — MCP server registrations shared by Claude Code and Codex
- `.cc-suite.md` — per-project config, incl. the `## Enabled Tools` list that selects which agents the multi-tool bridge targets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaolai/cc-suite](https://github.com/xiaolai/cc-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
