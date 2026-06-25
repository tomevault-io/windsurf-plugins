---
trigger: always_on
description: This file follows the [agents.md](https://agents.md/) convention: a single agent-facing readme that any coding agent (Claude Code, OpenCode, Codex, Antigravity, Cursor, Aider, …) can read on entry to this repository. Sections below follow the spec's recommended four-section structure (Project Structure / Coding Conventions / Testing / PR Guidelines) plus two autopilot-added sections (Build / Contribution).
---

# autopilot — Agent Instructions

This file follows the [agents.md](https://agents.md/) convention: a single agent-facing readme that any coding agent (Claude Code, OpenCode, Codex, Antigravity, Cursor, Aider, …) can read on entry to this repository. Sections below follow the spec's recommended four-section structure (Project Structure / Coding Conventions / Testing / PR Guidelines) plus two autopilot-added sections (Build / Contribution).

For Claude Code-specific conventions, see [`CLAUDE.md`](CLAUDE.md). For cross-agent portability detail (what each platform actually supports vs. what's unverified), see [`references/multi-agent-portability.md`](references/multi-agent-portability.md).

---

## Project Structure (spec)

```
skills/              23 lifecycle/methodology skills (SKILL.md format)
agents/              3 methodology agents (reviewer, debugger, planner) — markdown body + YAML frontmatter
hooks/               Claude Code hooks (bash + JS) and hooks.json manifest
.opencode/           OpenCode wrapper (opencode.json, in-process TS plugin)
.claude-plugin/      Claude Code plugin manifest (canonical for version + description)
plugin.json          Root mirror of .claude-plugin/plugin.json (for non-Claude tools)
references/          Cross-skill reference docs (blind-dispatch, model-routing, portability)
scripts/             Deterministic tooling — prefer these over LLM judgment for mechanical work
docs/                projects/ (active + archive), plans/, BACKLOG.md, CHANGELOG.md
.githooks/           Repo-tracked pre-commit hooks (activated via scripts/install-hooks.sh)
```

Skill body is in `skills/<name>/SKILL.md`. Methodology agent prompt body is in `agents/<role>.md`. Hooks live in `hooks/` and are registered via `hooks/hooks.json`.

## Coding Conventions (spec)

- **Severity vocabulary** (unified across skills + agents): `🔴 Critical / 🟠 Major / 🟡 Minor / 🔵 Suggestion`. The dialectic skills (`think-tank*`) use a separate lowercase `critical / important / minor` tag for risk-not-severity — intentionally distinct.
- **No hardcoded dispatch metadata** in skill files. Use `scripts/resolve-dispatch.sh` to map role → `{model, mode, agent}` JSON.
- **Prefer scripts over LLM judgment** for mechanical work (regex scans, JSON parsing, diff filtering). When you write a new script, wire it into both `CLAUDE.md` scripts inventory and the relevant skill's "Available Scripts" table.

## Testing (spec)

- **Skill structure**: `scripts/validate.sh` validates every `SKILL.md` has the required YAML frontmatter (`name`, `description`) and structure.
- **Version manifest sync**: `node scripts/sync-version.js --check` (read-only) verifies the version mirrors (root `plugin.json`, `marketplace.json`, `README.md` version badge) + the description's skill/hook fragments match the canonical `.claude-plugin/plugin.json`. Run before any commit that touches version metadata.
- **Hook inventory drift**: `node scripts/check-hook-inventory.js --check` (read-only) is the single source of truth for the hook tally — it derives default-on/opt-in/disabled from real wiring (`hooks.json` + `settings.example.json`) and asserts every doc agrees on counts AND tier membership. Run it (no flag) to print the canonical lists when editing the hook docs.
- **Hooks runtime smoke test**: `CLAUDE_PLUGIN_ROOT=$(pwd) node hooks/intent-capture.js < /dev/null` should write to `~/.autopilot/intent/` without throwing.
- **Pre-commit gate**: After running `scripts/install-hooks.sh` once per clone, `git commit` runs `sync-version.js --check` automatically. Drift blocks the commit.

## Build (autopilot-added)

- **No build step for production**. Skills, agents, hooks ship as source. Claude Code loads them at plugin install time.
- **OpenCode plugin** (`.opencode/plugins/autopilot.ts`) is loaded by Bun in-process — no compilation, but `@opencode-ai/plugin` types are needed at edit time (see `.opencode/package.json`).
- **Version bump**: `node scripts/sync-version.js --version X.Y.Z --hook-count N --skill-count M --opt-in-count K --disabled-count X` propagates the new version + description fragments to all mirrors atomically (two-pass; fail-loud on regex drift). `--disabled-count` defaults to 0; default-on = hook-count − opt-in − disabled. Hook-count correctness itself is gated separately by `check-hook-inventory.js --check`.
- **Dev mode for Claude Code**: `scripts/dev-setup.sh` replaces the installed plugin cache with a symlink to your local clone. Edits take effect immediately.

## PR Guidelines (spec)

- **Branch naming**: `feat/<scope>`, `fix/<scope>`, `chore/<scope>`, `docs/<scope>`. For multi-phase work, use `<type>/v<version>-<short-name>` (e.g. `fix/v2.7.3-multi-agent-portability-correction`).
- **Commit messages**: Conventional Commits style (`type(scope): summary`). Co-authored-by lines are welcome for AI-assisted commits.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cookys/autopilot](https://github.com/cookys/autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
