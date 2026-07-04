---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Whetstone is a Rust CLI that installs and configures three token optimization tools for Claude Code:

- **Headroom** — HTTP proxy between Claude Code and the Anthropic API (50-90% context compression)
- **RTK** — Hook that rewrites CLI commands to compress output before entering context (60-90% savings)
- **Memory** — Persistent memory via ICM (embedded SQLite) or AutoMem (graph memory), with bundled skills and session hooks

Single binary distribution. Users run `whetstone setup` from inside a git project. Global tools (Headroom, RTK) install once; skills, rules, and memory provider are configured per-project.

**Bundled assets** in this repo:
- `assets/skills/` — 20 skill directories (copied to project's `.claude/skills/`)
- `assets/hooks/` — 5 hook `.sh` scripts (copied to `~/.claude/hooks/`)
- `assets/rules/` — 8 rule `.md` files (copied to project's `.claude/rules/`)
- `assets/commands/` — 2 command `.md` files (copied to project's `.claude/commands/`)
- `assets/db/schema.sql` — SQLite schema for session database

## Commands

<!-- AUTO-GENERATED: commands -->
| Command | Description |
|---------|-------------|
| `just run <args>` | Run whetstone with arguments |
| `just build` | Build debug binary |
| `just build-release` | Build optimized release binary |
| `just test` | Run all tests |
| `just test-one <name>` | Run a single test by name |
| `just lint` | Format check + clippy |
| `just fix` | Auto-format + clippy fix |
| `just check` | Check compilation without binaries |
| `just release-check` | Release quality gate (fmt + clippy + test) |
| `just release <level>` | Bump version, create release branch + PR |
| `just release-dry-run <level>` | Preview release without changes |
| `just info` | Show project and toolchain versions |
| `just loc` | Show lines of code |
| `just clean` | Remove build artifacts |
| `just deps` | Show dependency tree |
| `just audit` | Audit dependencies for vulnerabilities |
<!-- AUTO-GENERATED: end -->

## CLI Reference

<!-- AUTO-GENERATED: cli -->
```
whetstone                              # Default: headroom wrap claude (auto-selects newest available Sonnet; falls back to claude-opus-4-6)
whetstone setup [--full] [--headroom-extras EXTRAS]
whetstone uninstall
whetstone claude [args...]
whetstone code [args...]               # Alias for claude
whetstone proxy [args...]
whetstone rtk [args...]
whetstone version
whetstone update [--full]
whetstone release patch|minor|major|set X.Y.Z
whetstone release-publish patch|minor|major|set X.Y.Z # Deprecated
whetstone db init|add-session|add-insight|search|get-sessions|...
```
<!-- AUTO-GENERATED: end -->

`--headroom-extras` accepts: `all` (default = `proxy,code,mcp`), `none`, or comma-separated like `proxy,code`.

`--memory` is a global flag (e.g. `whetstone --memory`, `whetstone --memory claude`). It enables Headroom persistent cross-session memory by passing `--memory` to the proxy whetstone spawns. If a proxy is already running *without* memory, whetstone prompts: restart it with memory (replaces the global proxy), start a memory proxy for this session only, or cancel. It can also be set persistently per-project or globally via `whetstone settings` (Headroom Memory).

## Architecture

```
User → Claude Code
         ├── Bash calls → [RTK Hook] → rtk <cmd> → compressed output
         ├── Context    → [Headroom Proxy :8787] → Anthropic API
         └── Memory     → [ICM or AutoMem] → persistent context
```

**Setup flow** (`whetstone setup`, orchestrated by `src/setup.rs`):
1. Preflight: verify Python 3.10+, git, curl, uv; confirm inside git repo
2. Install Headroom via `uv tool install "headroom-ai[EXTRAS]"` (extras configurable)
3. Install RTK from GitHub (detects name collision with Rust Type Kit)
4. Configure RTK hook globally + set `ANTHROPIC_BASE_URL` in shell profile
5. Self-install binary to `~/.local/bin/whetstone`
6. Prompt for memory provider (ICM, AutoMem, or Skip)
7. Copy skills, rules, commands, MEMSTACK.md; create config.local.json
8. Install and configure chosen memory provider
9. Copy hook scripts to `~/.claude/hooks/`; merge into `~/.claude/settings.json` (backed up with timestamp)
10. Generate `STACK-SETUP.md`

**Hook system** — registered in `~/.claude/settings.json`:

| Event | What Fires | Source |
|-------|-----------|--------|
| PreToolUse (Bash) | RTK rewrites command | RTK |
| PreToolUse (Write/Edit/Bash) | TTS notification | whetstone |
| PreToolUse (Bash, git push) | Build check + secrets scan | whetstone |
| PostToolUse (git commit) | Debug artifact scan | whetstone |
| SessionStart | Headroom auto-start + indexing | whetstone |
| Stop | Session reporting | whetstone |

## Source Layout

<!-- AUTO-GENERATED: source-layout -->
```
src/
├── main.rs          # Entry: parse CLI, dispatch subcommands
├── cli.rs           # clap derive structs for all subcommands
├── setup.rs         # whetstone setup orchestrator (8 steps)
├── uninstall.rs     # Interactive component removal
├── wrapper.rs       # claude/proxy/rtk exec wrappers
├── update.rs        # 12h-cached remote version check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z19r/whetstone](https://github.com/z19r/whetstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
