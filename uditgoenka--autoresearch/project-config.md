---
trigger: always_on
description: > Drop this file into your project root. Any AI agent (Claude Code, Codex, OpenCode, Gemini CLI, etc.) can then use Autoresearch immediately.
---

# AGENTS.md — Autoresearch

> Drop this file into your project root. Any AI agent (Claude Code, Codex, OpenCode, Gemini CLI, etc.) can then use Autoresearch immediately.

## What is Autoresearch?

Autonomous goal-directed iteration based on [Karpathy's autoresearch](https://github.com/karpathy/autoresearch). One metric, constrained scope, fast verification, automatic rollback, git as memory. Works on ANY domain — code, content, marketing, sales, DevOps — anything with a measurable metric.

**Core loop:** Modify → Verify → Keep/Discard → Repeat.

---

## Installation

### Claude Code (plugin)

```
/plugin marketplace add uditgoenka/autoresearch
/plugin install autoresearch@autoresearch
```

Restart session after install. All 10 commands become available as `/autoresearch` and `/autoresearch:<subcommand>`.

### Codex (plugin)

```bash
git clone https://github.com/uditgoenka/autoresearch.git
cd autoresearch
python3 plugins/autoresearch/scripts/install_local_plugin.py
```

Use the wrapper CLI: `bin/autoresearch <subcommand> [flags]`

### Manual (any agent)

Copy the skill files into your agent's skill directory:

```bash
git clone https://github.com/uditgoenka/autoresearch.git

# Claude Code
cp -r autoresearch/claude-plugin/skills/autoresearch .claude/skills/autoresearch
cp -r autoresearch/claude-plugin/commands/autoresearch .claude/commands/autoresearch
cp autoresearch/claude-plugin/commands/autoresearch.md .claude/commands/autoresearch.md

# Codex
cp -r autoresearch/plugins/autoresearch ~/.agents/plugins/autoresearch
```

---

## Commands

| Command | Purpose |
|---------|---------|
| `autoresearch` | Autonomous iteration loop (unlimited or bounded with `Iterations: N`) |
| `autoresearch:plan` | Interactive wizard: Goal → Scope, Metric, Direction, Verify config |
| `autoresearch:debug` | Autonomous bug-hunting — scientific method + iterative investigation |
| `autoresearch:fix` | Autonomous error repair — one fix per iteration until zero errors |
| `autoresearch:security` | STRIDE + OWASP + red-team security audit (read-only unless `--fix`) |
| `autoresearch:ship` | Universal shipping workflow — 8 phases, 9 shipment types |
| `autoresearch:scenario` | Scenario exploration — 12 dimensions, edge cases, derivative scenarios |
| `autoresearch:predict` | Multi-persona swarm — 5 expert perspectives before acting |
| `autoresearch:learn` | Autonomous documentation engine — scout, generate, validate, fix |
| `autoresearch:reason` | Adversarial refinement — blind judge panel for subjective domains |

---

## Quick Start

### Basic autonomous loop

```
autoresearch
Goal: Increase test coverage from 72% to 90%
Scope: src/**/*.test.ts, src/**/*.ts
Metric: coverage % (higher is better)
Verify: npm test -- --coverage | grep "All files"
Iterations: 50
```

### Don't know what metric to use?

```
autoresearch:plan
Goal: Make the API respond faster
```

The wizard walks you through scope, metric, direction, and verify — with dry-run validation.

### Hunt all bugs

```
autoresearch:debug
Scope: src/api/**/*.ts
Symptom: API returns 500 on POST /users
Iterations: 20
```

### Fix all errors

```
autoresearch:fix
```

Auto-detects broken tests/types/lint/build, fixes one at a time, stops at zero errors.

### Security audit

```
autoresearch:security
Scope: src/**/*.ts
Iterations: 10
```

### Ship a PR

```
autoresearch:ship --auto
```

### Explore edge cases

```
autoresearch:scenario
Scenario: User attempts checkout with expired card
Iterations: 25
```

### Get expert opinions before acting

```
autoresearch:predict --chain debug
Scope: src/auth/**/*.ts
```

### Refine a subjective decision

```
autoresearch:reason
Task: Should we use event sourcing for order management?
Domain: software
Iterations: 8
```

---

## Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `Goal` | Yes | What you want to achieve (plain language) |
| `Scope` | Yes | Glob patterns for files the agent can modify |
| `Metric` | Yes | What number to optimize (higher/lower + unit) |
| `Verify` | Yes | Shell command that outputs the metric value |
| `Guard` | No | Safety command that must always pass (prevents regressions) |
| `Iterations` | No | Bounded run — stop after N iterations (default: unlimited) |
| `Direction` | No | `higher` or `lower` — which direction is better |

---

## Flags

### Core loop (`autoresearch`)

| Flag | Purpose |
|------|---------|
| `--scope <glob>` | Override scope |
| `--iterations <N>` | Bounded iteration count |

### Security (`autoresearch:security`)

| Flag | Purpose |
|------|---------|
| `--diff` | Only audit changed files |
| `--fix` | Auto-fix Critical/High findings |
| `--fail-on <severity>` | Non-zero exit for CI/CD gating |

### Ship (`autoresearch:ship`)

| Flag | Purpose |
|------|---------|
| `--auto` | Auto-approve if checklist passes |
| `--dry-run` | Validate without shipping |
| `--checklist-only` | Just check readiness |
| `--rollback` | Undo last ship |
| `--monitor <N>` | Post-ship monitoring (minutes) |

### Debug (`autoresearch:debug`)

| Flag | Purpose |
|------|---------|
| `--fix` | After hunting, auto-switch to fix mode |
| `--scope <glob>` | Limit investigation scope |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uditgoenka/autoresearch](https://github.com/uditgoenka/autoresearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
