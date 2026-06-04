---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is hapai

hapai is a deterministic guardrails system for AI coding assistants (Claude Code, Cursor, Copilot). It enforces security rules via shell-based hooks that intercept tool calls and block violations **before execution** — not probabilistic prompts that can be ignored. Pure Bash, only external dependency is `jq`.

The system combines:
- **Hook enforcement** — Shell scripts that run before/after tool execution
- **Svelte 5 Analytics Dashboard** — Real-time guardrail event visualization
- **Cloud integration** — BigQuery + Cloud Storage + GitHub Pages deployment
- **Multi-tool exporters** — Export guardrails to Cursor, Copilot, Windsurf, etc.

## Prerequisites

Before working on this codebase, ensure:
- **jq 1.6+** — Required for all hook scripts and validation. Install: `brew install jq`
- **Node.js 20+ + npm** — Required only for dashboard development (`infra/gcp/dashboard/`)
- **Bash 4+** — All scripts use `set -euo pipefail` and POSIX-compatible tools
- **Git** — Version control and release tagging
- **Tested platforms** — macOS and Linux. CI runs on both via `ci.yml`. WSL supported via installer detection.

## Development Environment Setup

**Complete setup for contributing:**

```bash
# 1. Clone the repository
git clone https://github.com/renatobardi/hapai.git
cd hapai

# 2. Verify prerequisites
jq --version      # Should be 1.6+
bash --version    # Should be 4+
node --version    # Should be 20+ (only if developing dashboard)

# 3. Install for development with hooks enabled
HAPAI_DEV=1 bash install.sh

# 4. Validate installation
hapai validate

# 5. Run full test suite to verify everything works
bash tests/run-tests.sh

# 6. For dashboard development, install Node dependencies
cd infra/gcp/dashboard && npm ci && cd ../..

# You're ready to develop!
```

**Development setup notes:**
- `HAPAI_DEV=1` installs hooks to `~/.hapai/hooks/` and registers them in `~/.claude/settings.json`
- Project-local `hapai.yaml` overrides user-global `~/.hapai/hapai.yaml` — useful for testing
- All hook changes take effect immediately; no reinstall needed
- Dashboard requires `.env` file with Firebase credentials (see Dashboard section)

## Quick Commands

Common tasks when developing hapai:

```bash
# Run all tests (bash assertions, no framework)
bash tests/run-tests.sh

# Test a specific guardrail
bash tests/run-tests.sh 2>&1 | grep -A 30 "guard-branch"

# Test individual hook in isolation
echo '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"git commit -m test"}}' | \
  bash hooks/pre-tool-use/guard-branch.sh

# Validate installation
hapai validate

# Check active hooks and audit counts
hapai status

# View recent audit log entries
hapai audit

# Dashboard development (local)
cd infra/gcp/dashboard && npm ci && npm run dev

# Dashboard production build
cd infra/gcp/dashboard && npm run build

# Test CLI installation locally
HAPAI_DEV=1 bash install.sh
```

### Testing Individual Hooks

Hooks read Claude Code's hook JSON format from stdin. The correct schema:

```bash
# PreToolUse — Bash command
echo '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"git commit -m test"}}' | \
  bash hooks/pre-tool-use/guard-branch.sh
echo "Exit: $?"  # 0=allow, 2=deny

# PreToolUse — file write
echo '{"hook_event_name":"PreToolUse","tool_name":"Write","tool_input":{"file_path":".env","content":"SECRET=1"}}' | \
  bash hooks/pre-tool-use/guard-files.sh
```

To test with a clean isolated state:

```bash
export HAPAI_HOME="$(mktemp -d)"
mkdir -p "$HAPAI_HOME/state"
cp hapai.defaults.yaml "$HAPAI_HOME/hapai.yaml"
echo '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"rm -rf /"}}' | \
  bash hooks/pre-tool-use/guard-destructive.sh
```

## Architecture

### Directory Structure

```
hapai/
├── bin/hapai                    # CLI entry point (command dispatcher)
├── hooks/                       # All guardrail scripts (pure Bash)
│   ├── _lib.sh                 # Shared library (YAML parsing, JSON I/O, audit, state)
│   ├── _pr-review-agent.sh     # Background PR reviewer (claude CLI, Haiku model)
│   ├── _pr-fix-agent.sh        # Optional auto-fixer (Sonnet model)
│   ├── pre-tool-use/           # Block before Claude Code execution
│   │   ├── guard-*.sh          # Individual guardrails (branch, commit, files, etc.)
│   │   └── flow-dispatcher.sh  # Sequential hook chains from config
│   ├── post-tool-use/          # Run after Claude Code execution
│   │   ├── auto-*.sh           # Automations (format, lint, checkpoint)
│   │   └── audit-trail.sh      # Audit logging and PR review
│   ├── stop/                   # Run at session end (cleanup, cost tracking)
│   ├── session-start/          # Load context, scan TODOs/issues on session init
│   ├── user-prompt-submit/     # Warn on production keywords before any tool runs
│   ├── pre-compact/            # Backup transcript before context compaction
│   ├── notification/           # Sound alerts on guardrail events
│   ├── permission-request/     # Auto-allow read-only operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renatobardi/hapai](https://github.com/renatobardi/hapai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
