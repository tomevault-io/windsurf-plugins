---
trigger: always_on
description: This `CLAUDE.md` is the root context file for Claude Code sessions in this skills repo.
---

# OpenSite Agent Skills — Claude Code Context

This `CLAUDE.md` is the root context file for Claude Code sessions in this skills repo.
It covers the memory workflow, available skills, and model routing guidance.

> **Using this in your own project?** Replace the **Project Context** section below
> with your own codebase details — repos, tech stack, and critical rules specific to
> your project. Keep the Memory System section as-is; it works for any project.

---

## 🧠 Memory System — Core Workflow

This repo ships a four-skill persistent memory engine. **Use it every session, without
exception.** It is your primary mechanism for maintaining continuity across conversations.

### Session Protocol

```
START OF SESSION         →  /memory-recall
  ↓ (work happens)
END OF SESSION           →  /memory-write
  ↓ (weekly or after bulk writes)
MAINTENANCE              →  /memory-consolidate
```

**Never start non-trivial work without running `/memory-recall` first.** It loads your
working memory handoff, project context, architecture decisions, conventions, and recent
session history before a single line of code is touched.

**Never end a session without running `/memory-write`.** It extracts decisions, facts,
and outcomes from the conversation and writes them to the correct memory layer so the
next session can pick up exactly where this one left off.

### Memory Skills

| Skill | Role | Invoke When |
|-------|------|-------------|
| `memory` | Core store — schema, scripts, direct read/write/search | Anytime you need raw memory access |
| `memory-recall` | Context retrieval — loads all relevant history before work | **Every session start** / "do you remember" |
| `memory-write` | Session capture — extracts and persists session learnings | **Every session end** / "save this" / "remember this" |
| `memory-consolidate` | Maintenance — decay, dedup, compress old sessions | Weekly or after bulk writes |

### Memory Layers

| Layer | Path | What Goes Here |
|-------|------|----------------|
| Episodic | `memory/store/episodic/` | Session summaries, milestones, breakthroughs |
| Semantic | `memory/store/semantic/` | Project facts, tech notes, user preferences, domain knowledge |
| Procedural | `memory/store/procedural/` | ADRs, workflows, code conventions |
| Working | `memory/store/working/active.md` | Hot context handoff between sessions |

### Inline Memory Commands

You can also trigger memory operations mid-session:

```
"save this"              →  /memory-write   (captures current conversation state)
"remember this"          →  /memory-write
"do you remember X?"     →  /memory-recall  (targeted recall for X)
"store to memory"        →  /memory-write
```

### What to Write

**Always write:**
- Architecture decisions with rationale (ADRs) — even small ones
- Non-trivial bugs: root cause + fix + why it happened
- New file paths, data models, or API contracts established in this session
- User preferences expressed explicitly ("always use X over Y")
- Repeatable workflows or processes established this session

**Write selectively:**
- Technology facts (version numbers, library gotchas, confirmed behaviors)
- Project-specific knowledge that won't be obvious from the code

**Never write:**
- Trivial edits (typo fixes, formatting)
- Facts already in the codebase and easily found by grep
- Duplicates — always run `search_memory.py` before writing

### Data Privacy

The memory store (`memory/store/`) is gitignored and lives only on this machine. It is
never committed. Only the skill instruction files and Python scripts are versioned.

---

## Project Context

> **Fill this section in for your own project.**
> The memory skills will build up project-specific context over time, but seeding
> Claude with a few key facts here gives it a useful starting point on the first session.

```
## Repositories
- **Primary repo** — [description, language, framework, deployment target]
- **Secondary repo** — [description]

## Tech Stack
- [Language + framework]
- [Database]
- [Deployment]
- [Key libraries]

## Critical Rules
1. [Your most important rule]
2. [Second rule]
3. [Add as many as needed]
```

---

## Available Skills

Skills are available in `.claude/skills/`. Load with `/skill-name` or let Claude trigger
them automatically based on context.

### Memory (Core — Use Every Session)

| Skill | When to Use |
|-------|-------------|
| `memory-recall` | **Start of every session** — loads full context before work begins |
| `memory-write` | **End of every session** — persists decisions, facts, and outcomes |
| `memory` | Direct memory store access — search, read, or write entries manually |
| `memory-consolidate` | Weekly maintenance — decay, dedup, compress old sessions |

### Context Management

| Skill | When to Use |
|-------|-------------|
| `context-management` | Long sessions, large tool outputs, after compaction, context window running low. FTS5 indexing, BM25 search, output compression, session checkpointing. |

### Rust + Axum

| Skill | When to Use |
|-------|-------------|
| `octane-rust-axum` | Axum routes, handlers, middleware, and service patterns |
| `octane-soc2-hipaa` | Compliance, audit logging, PHI handling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensite-ai/opensite-skills](https://github.com/opensite-ai/opensite-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
