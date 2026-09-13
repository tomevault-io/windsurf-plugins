---
trigger: always_on
description: Guidance for AI coding agents working in the `viserys` repository.
---

# AGENTS.md

Guidance for AI coding agents working in the `viserys` repository.

---

## 0. Distribution Preference (read this first)

When the user mentions Viserys, asks how to activate it, or asks whether it works in another project, do NOT assume. Follow these steps:

1. **Check the global scope.** Look for an agent file at any of:
   - `~/.config/opencode/agent/viserys.md`
   - `~/.config/opencode/agents/viserys.md`

2. **If it does not exist**, explain that there are two ways to use Viserys, then **ask the user which they want**:
   - **Project scope** — already active automatically. In this repository, press `Tab` and select `viserys`. Nothing to install.
   - **Global scope** — copy `.opencode/agent/viserys.md` into `~/.config/opencode/agent/`, and register `skills.paths` in the user's global config. This activates Viserys in every project the user opens.

3. **If the user chooses global scope**, print the exact commands for them to run. Do **not** modify the user's global configuration yourself unless they explicitly ask you to.

4. **Never touch configuration outside this repository** without explicit permission.

If the global agent already exists, say so and confirm it is active. Do not offer to reinstall it.

---

## 1. Identity

`viserys` is a self-contained engineering workflow pack for AI coding agents. It packages the planning, decomposition, verification, and review discipline that senior engineers apply, so an agent follows the same process on every task instead of improvising.

- Version: `0.1.0`
- Plain Markdown plus a small set of validator scripts
- No runtime dependency, no network dependency
- 27 skills, 4 personas, 7 shared checklists

The pack does not generate code by itself. It gives agents a process. Your job when working here is to keep that process intact and consistent.

---

## 2. Repository Structure

| Path | Purpose |
|---|---|
| `skills/` | 27 `SKILL.md` workflows, plus per-skill `references/` and `scripts/` where needed |
| `agents/` | 4 reviewer personas (not active in OpenCode — see section 7) |
| `references/` | 7 shared checklists pulled in by skills on demand |
| `.claude/commands/` | 9 slash commands (Claude Code adapter) |
| `commands/` | 9 TOML commands (Antigravity adapter) |
| `.gemini/commands/` | 9 TOML commands (Gemini CLI adapter) |
| `.opencode/` | OpenCode adapter: the `viserys` primary agent and `skills.paths` registration |
| `.claude-plugin/`, `.codex-plugin/`, `.agents/plugins/` | Plugin and marketplace manifests for other harnesses |
| `evals/` | 25 eval cases and their fixtures |
| `scripts/` | 5 structural validators, the eval runner, and their test suites |
| `hooks/` | Session lifecycle hooks (Claude Code) |
| `docs/` | `README.md` (pack overview) and `skill-anatomy.md` (the `SKILL.md` spec) |
| `tasks/` | Internal working documents, not tracked in git |

`plugin.json` at the root is the generic plugin manifest. The harness-specific manifests live inside their adapter directories.

---

## 3. Lifecycle and Skills

The skills map onto six phases:

```
DEFINE -> PLAN -> BUILD -> VERIFY -> REVIEW -> SHIP
```

### DEFINE

| Skill | Use when |
|---|---|
| `get-prd` | Starting a new application, system, or feature and no requirements document exists yet. Deep interview plus PRD authoring in one pass |
| `idea-refine` | A concept is still vague and needs structured exploration |
| `interview-me` | The ask is underspecified and requirements must be extracted one question at a time |
| `spec-driven-development` | Starting a project, feature, or significant change with no spec yet |
| `constraint-driven-development` | No quality bar is written down, or checks are being silenced to get green |

### PLAN

| Skill | Use when |
|---|---|
| `get-tasks` | An approved PRD or spec exists and needs executable task files |
| `planning-and-task-breakdown` | You have a spec and need ordered, verifiable tasks with acceptance criteria |

### BUILD

| Skill | Use when |
|---|---|
| `incremental-implementation` | A change touches more than one file and should land in thin slices |
| `test-driven-development` | Implementing logic, fixing a bug, or changing behavior |
| `context-engineering` | Setting up a session, switching tasks, or output quality drops |
| `source-driven-development` | Correctness depends on current official documentation |
| `doubt-driven-development` | Stakes are high and a confident answer is cheaper to verify now than to debug later |
| `frontend-ui-engineering` | Building or modifying user-facing interfaces |
| `api-and-interface-design` | Designing APIs, module boundaries, or public interfaces |

### VERIFY

| Skill | Use when |
|---|---|
| `browser-testing-with-devtools` | Building or debugging anything that runs in a browser |
| `debugging-and-error-recovery` | A test fails, a build breaks, or behavior is unexpected |

### REVIEW

| Skill | Use when |
|---|---|
| `code-review-and-quality` | Before merging any change |
| `code-simplification` | Code works but is harder to read or maintain than it should be |
| `security-and-hardening` | Handling user input, auth, data storage, or external integrations |
| `performance-optimization` | Performance requirements exist or a regression is suspected |

### SHIP

| Skill | Use when |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rizqinrr/viserys-agent](https://github.com/rizqinrr/viserys-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
