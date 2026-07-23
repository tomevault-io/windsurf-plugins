---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## New Here? Start With These

1. **[README.md](README.md)** - Project overview, architecture, quick start
2. **[REPO_WALKTHROUGH.md](.claude/docs/REPO_WALKTHROUGH.md)** - Directory structure with annotations
3. **[PRINCIPLES.md](.claude/docs/PRINCIPLES.md)** - Design principles and trade-offs
4. **[INVARIANTS.md](.claude/docs/INVARIANTS.md)** - Rules that must never be violated
5. **[envs/echo_env/](envs/echo_env/)** - Reference implementation to study

## Agentic-First Workflow

OpenEnv uses Claude Code as the primary development tool. We follow a two-phase model:

1. **Design/Alignment** (human-owned): RFCs, principles, trade-off decisions
2. **Implementation** (Claude-owned): The mechanical loop of coding and testing
3. **Review** (collaborative): Claude catches bugs, flags alignment questions for humans

### Getting Started

Skills and agents are auto-discovered when you run Claude Code in this repo:

```bash
git clone https://github.com/huggingface/OpenEnv
cd OpenEnv
# Install git hooks for the team
bash .claude/hooks/install.sh
# Run Claude Code - skills and agents are automatically available
```

Verify with `/agents` or ask "what skills are available?"

## Two Development Modes

OpenEnv supports two development modes:

### Explore Mode (Main Repo)

When working in the main repository clone, direct edits are allowed:
- Quick exploration and prototyping
- Small fixes that don't need TDD workflow
- Documentation updates

### TDD Mode (Opt-In)

TDD is activated by `/work-on-issue`, which writes a `.tdd-session.json` marker.
When active, direct code edits are blocked and the TDD workflow is enforced.
Manually created worktrees do NOT activate TDD — only `/work-on-issue` does.

- Say "skip TDD" to bypass blocking
- Run `bash .claude/hooks/tdd-deactivate.sh` to turn off TDD enforcement

### Creating a Worktree

```bash
# Worktree without TDD enforcement (free editing)
.claude/scripts/worktree-create.sh add-feature
cd .worktrees/add-feature

# Worktree WITH TDD enforcement (via /work-on-issue)
/work-on-issue #42
```

### TDD Workflow

```
/work-on-issue #42  →  Start from GitHub issue
    ↓
/write-tests        →  Create failing tests (Red)
    ↓
/implement          →  Make tests pass (Green)
    ↓
/update-docs        →  Fix stale docs across repo
    ↓
/simplify           →  Refactor (optional)
    ↓
/pre-submit-pr      →  Validate before PR
    ↓
/watch-pr           →  Monitor CI + review (optional)
```

### Skills vs Agents

- **Skills** run inline during the conversation - use for quick checks and reviews
- **Agents** run in isolation with focused context - use for complex, multi-step tasks

### Available Skills

Skills are defined in `.claude/skills/` and run inline:

**Review & Validation Skills:**

| Skill | Trigger | Definition |
|-------|---------|------------|
| [`alignment-review`](.claude/skills/alignment-review/SKILL.md) | "review this code" | Two-tier review (bugs + alignment flags) |
| [`pre-submit-pr`](.claude/skills/pre-submit-pr/SKILL.md) | "ready for PR?" | Comprehensive PR readiness check |
| [`rfc-check`](.claude/skills/rfc-check/SKILL.md) | "do I need an RFC?" | Determine if RFC required |
| [`generate-openenv-env`](.claude/skills/generate-openenv-env/SKILL.md) | "generate an env for textarena" | Build an OpenEnv env from a use case |

**TDD Workflow Skills:**

| Skill | Trigger | Definition |
|-------|---------|------------|
| [`work-on-issue`](.claude/skills/work-on-issue/SKILL.md) | "/work-on-issue #42" | Start TDD workflow from GitHub issue |
| [`sprint`](.claude/skills/sprint/SKILL.md) | "/sprint 67,68,69" | Parallel multi-issue batch (Agent Teams) |
| [`write-tests`](.claude/skills/write-tests/SKILL.md) | "/write-tests" | Write failing tests (Red phase) |
| [`implement`](.claude/skills/implement/SKILL.md) | "/implement" | Make tests pass (Green phase) |
| [`update-docs`](.claude/skills/update-docs/SKILL.md) | "/update-docs" | Fix stale docs after API changes |
| [`simplify`](.claude/skills/simplify/SKILL.md) | "/simplify" | Refactor after tests pass |
| [`watch-pr`](.claude/skills/watch-pr/SKILL.md) | "/watch-pr" | Monitor CI + Greptile review after PR |

### Available Subagents

Agents are defined in `.claude/agents/` and run in isolation:

**Review & Validation Agents:**

| Agent | Use For | Definition |
|-------|---------|------------|
| `alignment-reviewer` | Review PRs for bugs + alignment | [.claude/agents/alignment-reviewer.md](.claude/agents/alignment-reviewer.md) |
| `env-validator` | Validate environments end-to-end | [.claude/agents/env-validator.md](.claude/agents/env-validator.md) |
| `openenv-architect` | Design new features/environments | [.claude/agents/openenv-architect.md](.claude/agents/openenv-architect.md) |
| `build-validator` | Validate builds before merge | [.claude/agents/build-validator.md](.claude/agents/build-validator.md) |

**TDD Workflow Agents:**

| Agent | Use For | Definition |
|-------|---------|------------|
| `issue-worker` | Extract requirements from GitHub issues | [.claude/agents/issue-worker.md](.claude/agents/issue-worker.md) |
| `pr-planner` | Plan stacked PRs for complex features | [.claude/agents/pr-planner.md](.claude/agents/pr-planner.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/OpenEnv](https://github.com/huggingface/OpenEnv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
