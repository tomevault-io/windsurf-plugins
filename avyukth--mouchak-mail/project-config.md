---
trigger: always_on
description: Validates test suite effectiveness by introducing mutations and checking if tests catch them:
---

# AGENTS.md — Universal Operating Manual for AI Coding Agents

> **Quick Start**: Run `cm context "<your task>"` before starting work. Run `bd ready` to find unblocked issues. Run `bd sync` before ending your session.

This document provides standardized instructions for ANY AI coding agent (Claude, Gemini, GPT, Codex, etc.) working in this codebase. It is divided into:

1. **Layer 0**: Inviolable safety rules (NEVER break these)
2. **Layer 1**: Universal tooling (works across all projects)
3. **Layer 2**: Session workflow (how to start, work, and end sessions)
4. **Layer 3**: Project-specific configuration (filled in per-project)
5. **Layer 4**: Language/stack-specific instructions (filled in per-project)

---

## ⛔ LAYER 0: INVIOLABLE SAFETY RULES

These rules are ABSOLUTE and apply to ALL agents in ALL contexts.

### Rule 1: NO FILE DELETION WITHOUT EXPLICIT PERMISSION

```
YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS WRITTEN PERMISSION.
```

- Applies to ALL files: user files, test files, temporary files, files you created
- You must ASK and RECEIVE clear written permission before ANY deletion
- "I think it's safe" is NEVER acceptable justification
- This includes: `rm`, `unlink`, file system APIs, git clean operations

### Rule 2: NO DESTRUCTIVE GIT/FILESYSTEM COMMANDS

**Forbidden commands** (unless user provides exact command AND acknowledges consequences in same message):

| Command | Risk |
|---------|------|
| `git reset --hard` | Destroys uncommitted work |
| `git clean -fd` | Deletes untracked files |
| `rm -rf` | Recursive deletion |
| `git push` | **NEVER**|
| `git push --force` | Overwrites remote history |
| Any command that deletes/overwrites code or data | Potential data loss |

**Mandatory protocol for ANY destructive operation:**

1. **Safer alternatives first**: Use `git status`, `git diff`, `git stash`, or copy to backups
2. **Explicit plan**: Restate the command verbatim + list exactly what will be affected
3. **Wait for confirmation**: Do NOT proceed without explicit user approval
4. **Document**: Record user authorization text, command run, and execution time
5. **Refuse if ambiguous**: If ANY uncertainty remains, refuse and escalate

### Rule 3: PROTECT CONFIGURATION FILES

- **`.env` files**: NEVER overwrite—they contain secrets and local configuration
- **Lock files**: Do not delete `package-lock.json`, `Cargo.lock`, `go.sum`, etc.
- **Database files**: Never delete `.db`, `.sqlite`, or data files without explicit permission

---

## 🧠 LAYER 1: UNIVERSAL TOOLING

These tools work across ALL projects. Learn them once, use them everywhere.

### Quick Reference: Tool Selection

| Task | Tool | Command |
|------|------|---------|
| **Start new task** | cm | `cm context "<task>"` |
| **Find ready issues** | bd | `bd ready --json` |
| **Search past sessions** | cass | `cass search "query" --robot` |
| **Graph analysis of issues** | bv | `bv --robot-insights` |
| **AI-supervised execution** | vc | `vc run` / `vc status` |
| **Quality gates / TDG** | pmat | `pmat analyze tdg` / `pmat mutate` |
| **Bug scan before commit** | ubs | `ubs $(git diff --name-only --cached)` |
| **Structural code search** | ast-grep | `ast-grep run -l <lang> -p 'pattern'` |
| **Text search** | ripgrep | `rg "pattern"` |
| **Multi-agent coordination** | Mouchak Mail | `file_reservation_paths(...)` |
| **AI capability discovery** | mouchak-mail | `--robot-help`, `--robot-examples`, `--robot-status` |

---

### 📚 cm (CASS Memory System) — Context Hydration

**What it does**: Provides "procedural memory" across coding sessions. Before starting ANY non-trivial task, hydrate your context.

**Primary command**:
```bash
cm context "<your task description>"
```

This returns:
- Relevant rules from the project playbook (scored by task relevance)
- Anti-patterns to avoid (things that caused problems before)
- Historical context from past sessions

| Command | Purpose |
|---------|---------|
| `cm context "<task>"` | **START HERE** — Get context for your task |
| `cm doctor` | System health check (exit 0 = healthy) |
| `cm init` | Initialize playbook for new project |
| `cm mark <id> --helpful` | Positive feedback on a rule |
| `cm mark <id> --harmful` | Negative feedback on a rule |
| `cm stats` | Playbook health metrics |
| `cm similar "<query>"` | Find similar rules |
| `cm top` | Top N rules by effectiveness score |
| `cm forget <id>` | Deprecate a harmful rule |
| `cm why <id>` | Explain reasoning behind a rule |
| `cm diary` | Record session as diary entry |
| `cm project` | Export playbook to AGENTS.md/CLAUDE.md |

**Output conventions**: stdout = data, stderr = diagnostics. Exit 0 = success.

---

### 🔎 cass (Cross-Agent Session Search) — History Search

**What it does**: Indexes conversation histories from ALL AI coding agents (Claude, Codex, Cursor, Gemini, Aider, ChatGPT, etc.) into a unified, searchable archive.

**Before solving a problem from scratch, check if ANY agent already solved something similar.**

⚠️ **CRITICAL**: NEVER run bare `cass` — it launches an interactive TUI. Always use `--robot` or `--json`.

#### Pre-Flight Health Check

```bash
cass health --json
# Exit 0 = healthy (proceed with searches)
# Exit 1 = unhealthy (run: cass index --full)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Avyukth/mouchak-mail](https://github.com/Avyukth/mouchak-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
