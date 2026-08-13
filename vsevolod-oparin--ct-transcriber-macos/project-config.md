---
trigger: always_on
description: 109 specialized AI agents for OpenCode. Agents are stored in `.opencode/agents/` as Markdown files with YAML frontmatter.
---

## Agents

109 specialized AI agents for OpenCode. Agents are stored in `.opencode/agents/` as Markdown files with YAML frontmatter.

**Using Agents:** Agents are automatically discovered. Invoke any agent with `@agent-name` in your message. Example: `@python-pro optimize this function`.

**Discovery:** Consult `INDEX.md` at repo root for the full categorized agent directory (109 agents grouped by domain). Pick the MOST specialized agent — domain-specific checklists and anti-patterns only work when the agent matches the domain.

### Agent Categories

| Category | Count | Examples |
|----------|-------|----------|
| Language Implementation | 22 | python-pro, golang-pro, rust-pro, typescript-pro |
| Web Frameworks | 10 | react-pro, nextjs-pro, django-pro, fastapi-pro |
| Architecture & Design | 9 | backend-architect, api-designer, microservices-architect |
| DevOps & Infrastructure | 11 | devops-engineer, kubernetes-architect, cloud-architect |
| Security | 6 | security-reviewer, penetration-tester, threat-modeling-pro |
| Database | 5 | postgres-pro, sql-pro, database-architect |
| Testing & Quality | 5 | code-reviewer, tdd-guide, test-automator |
| AI & ML | 5 | ai-engineer, ml-engineer, prompt-engineer |
| Frontend & Mobile | 5 | frontend-developer, ios-pro, ui-designer |
| Documentation | 7 | documentation-pro, technical-writer, docs-architect |
| Incident & Troubleshooting | 4 | incident-responder, debugger, devops-troubleshooter |
| Specialized | 20 | build-engineer, cli-developer, product-manager, web-searcher, etc. |

### Agent Selection

Most specialized wins (e.g., postgres-pro over database-optimizer). Split hybrid tasks into subtasks with different agents.

---

## Action Reporting

**Every action must be announced before execution.** Before calling any tool (Read, Edit, Write, Bash, Grep, Glob, Task, etc.), reading session or long term memory or any other action output a short one-line description of what you are doing and why. Format: `[action] — <short description>`. Examples:

- `Reading config.yml to check database settings`
- `Running npm test to verify changes`
- `Editing auth.ts to fix the null check`
- `Searching for all usages of deprecated API`
- `Spawning researcher agent to analyze dependencies`

This applies to ALL tool calls — file operations, shell commands, searches, agent spawns, everything. No silent tool usage.

---

## Memory System

**NEVER use MEMORY.md for anything.** MEMORY.md is the built-in auto-memory system and is completely separate from this project's memory system. Do not read, write, or reference MEMORY.md. Use only `knowledge.md` and `session.md` via the `memory.sh` tool.

Two-tier: **Knowledge** (`knowledge.md`) permanent, **Session** (`session.md`) temporary.

| Question | Use |
|----------|-----|
| Will this help in future sessions? | **Knowledge** |
| Current task only? | **Session** |
| Discovered a gotcha/pattern/config? | **Knowledge** |
| Tracking todos/progress/blockers? | **Session** |

### Knowledge

```bash
./.opencode/tools/memory.sh add <category> "<content>" [--tags a,b,c]
```

| Category | Save When |
|----------|-----------|
| `architecture` | System design, service connections, ports |
| `gotcha` | Bugs, pitfalls, non-obvious behavior |
| `pattern` | Code conventions, recurring structures |
| `config` | Environment settings, credentials |
| `entity` | Important classes, functions, APIs |
| `decision` | Why choices were made |
| `discovery` | New findings about codebase |
| `todo` | Long-term tasks to remember |
| `reference` | Useful links, documentation |
| `context` | Background info, project context |

**Tags:** Cross-cutting concerns (e.g., `--tags redis,production,auth`). **Skip:** Trivial, easily grep-able, duplicates.

**After tasks:** State "**Memories saved:** [list]" or "**Memories saved:** None"

**Other:** `search "<query>"`, `list [--category CAT]`, `delete <id>`, `stats`

### Session

Tracks current task. Persists until cleared.

**Categories:** `plan`, `todo`, `progress`, `note`, `context`, `decision`, `blocker`. **Statuses:** `pending` → `in_progress` → `completed` | `blocked`.

```bash
./.opencode/tools/memory.sh session add todo "Task" --status pending
./.opencode/tools/memory.sh session show                    # View current
./.opencode/tools/memory.sh session update <id> --status completed
./.opencode/tools/memory.sh session delete <id>
./.opencode/tools/memory.sh session clear                   # Current only
./.opencode/tools/memory.sh session clear --all             # ALL sessions
```

### Checkpoints

Save after every significant step. One active checkpoint (delete previous first). Under 500 chars.

```bash
./.opencode/tools/memory.sh session add context "CHECKPOINT: [task] | DONE: [steps] | CURRENT: [now] | NEXT: [remaining] | FILES: [key files] | DECISIONS: [choices] | BUILD/TEST: [commands]"
```

After compaction: run `./.opencode/tools/memory.sh session show` immediately to restore state.

**Rules:** One checkpoint at a time. Always include DONE and NEXT. Don't skip — losing state costs more.

### Multi-Session


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vsevolod-oparin/ct-transcriber-macos](https://github.com/vsevolod-oparin/ct-transcriber-macos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
