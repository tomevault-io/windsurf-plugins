---
trigger: always_on
description: Portable Claude Code workflow definitions (agents, skills, hooks, settings) maintained in a single git repo and installed to `~/.claude/` via symlinks.
---

# CLAUDE.md

Portable Claude Code workflow definitions (agents, skills, hooks, settings) maintained in a single git repo and installed to `~/.claude/` via symlinks.

## Operating Mode: Orchestrator

**Posture depends on project type:**
- **Content-only projects** (no `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, or `Makefile` at root): prefer direct implementation over subagent dispatch for simple edits.
- **Code projects**: orchestrator-only. Dispatch all implementation to subagents and coordinate work.

This repo itself (tackline) is content-only -- direct edits to `.md` and `.json` files are preferred over spawning agents for trivial changes.

### Orchestrator Responsibilities

1. **Task Tracking**: Triage, prioritize, and track work using your preferred task tracking approach
2. **Task Dispatch**: Delegate implementation work to appropriate subagents via the Task tool
3. **Coordination**: Manage dependencies between tasks, unblock work, review agent outputs

### Dispatching Strategy

**Default: parallelize with worktree isolation.** Dispatch independent tasks concurrently using `isolation: "worktree"` and `run_in_background: true`. Each agent gets its own repo copy — no merge conflicts, no context bloat. Cherry-pick or merge results after agents complete.

**Fall back to serial** only when tasks have true sequential dependencies (each task needs the previous one's output to proceed). Serial dispatch is the exception, not the default.

**Use agent teams** when agents must communicate mid-execution (not merely because tasks depend on each other).

---

## Quick Reference

```bash
# Add the marketplace (one-time), then install
claude plugin marketplace add tyevans/tackline
claude plugin install tackline@tackline

# Install global rules (not supported by plugin system)
# No local checkout needed — fetch directly from GitHub:
curl -fsSL https://raw.githubusercontent.com/tyevans/tackline/main/dev/install-rules.sh | bash
# Or, from a local checkout:
# bash /path/to/tackline/dev/install-rules.sh

# Uninstall
claude plugin uninstall tackline@tackline
for f in /path/to/tackline/rules/*.md; do rm -f ~/.claude/rules/"$(basename "$f")"; done
```

## Skills

48 skills across three layers: **core** (14 composable primitives), **workflows** (25 orchestrated multi-step workflows), **teams** (9 team orchestration + learning lifecycle). Full catalog with decision tree and chain patterns: [docs/INDEX.md](docs/INDEX.md). Composable primitives follow [pipe format](rules/pipe-format.md).

## Project Structure

```
tackline/
├── agents/
│   ├── agent-generator.md      # Generates project-specific agents
│   ├── project-bootstrapper.md # Bootstraps projects with full Claude Code setup
│   └── code-reviewer.md        # Read-only code review agent
├── skills/                      # Skill definitions (flattened to ~/.claude/skills/ on install)
│   ├── core/                    # Composable primitives + entrypoints (16 skills)
│   │   ├── gather/, distill/, rank/, filter/, assess/, verify/
│   │   ├── expand/, transform/, decompose/, critique/, plan/
│   │   ├── merge/, diff-ideas/, sketch/
│   │   └── do/, discover/       # Skill routing entrypoints
│   ├── workflows/               # Exploration, review, lifecycle, session mgmt (25 skills)
│   │   ├── blossom/, fractal/, consolidate/, review/, bootstrap/
│   │   ├── meeting/, consensus/, premortem/, spec/, bug/
│   │   ├── advise/, handoff/, status/, session-health/, domain/
│   │   ├── evolution/, drift/, test-strategy/, tracer/, deploy/
│   │   └── challenge-gen/, challenge-run/, diagnose-agent/
│   └── teams/                   # Team orchestration + learning lifecycle (9 skills)
│       ├── assemble/, standup/, sprint/, team-meeting/, active-learn/
│       └── curate/, promote/, tend/, retro/
├── docs/                        # Documentation (cookbook, recipes, team guide, INDEX)
│   ├── INDEX.md                 # Skill & agent navigator (decision tree, categories)
│   ├── design-notes.md          # Architecture and design decision notes
│   ├── domains.md               # Domain-to-artifact mapping
│   └── pipelines.md             # Canonical end-to-end lifecycle pipelines
├── demos/                       # Demo projects for primitive walkthroughs
├── dev/                         # Developer scripts
│   ├── lint.sh                  # Linting script
│   ├── new-agent.sh             # Scaffold a new agent definition
│   └── new-skill.sh             # Scaffold a new skill definition
├── rules/                       # Global rules (symlinked to ~/.claude/rules/)
│   ├── batch-safety.md          # Batch processing safety (chunk at 12 items)
│   ├── context-trust.md         # Trust user-provided context
│   ├── delegation.md            # Dispatch to subagents, concurrency by default, skills over intuition
│   ├── memory-layout.md         # Path registry + checkpoint protocol for persistent state
│   ├── pipe-format.md           # Composable primitive output contract (paths: skills/**/SKILL.md)
│   ├── team-protocol.md         # Team manifest, spawn protocol, reflection schema
│   └── test-conventions.md      # Testing conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyevans/tackline](https://github.com/tyevans/tackline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
