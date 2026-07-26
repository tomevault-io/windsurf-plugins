---
trigger: always_on
description: Reverse engineering the Aula Android app (`com.netcompany.aulanativeprivate`) — Denmark's school communication platform by Netcompany A/S. APK version 2.15.4.
---

# Aula APK Decompilation Project

## Project Context
Reverse engineering the Aula Android app (`com.netcompany.aulanativeprivate`) — Denmark's school communication platform by Netcompany A/S. APK version 2.15.4.

## Key Files
- `re/prd.apk_decompile.md` — Project requirements document with 4 milestones
- `shell.nix` — Nix environment with all decompilation/analysis tools
- `re/apk_extract.sh` — Recursive APK/XAPK extraction script
- `re/milestone2_analysis.md` — (produced by milestone 2) file analysis findings
- `re/architecture.md` — (produced by milestone 4) architecture documentation

## Task Management
- Use `backlog` CLI tool (in `$PATH`) for task tracking
- Always use `--plain` flag for `backlog task view`, `backlog board`, etc. to avoid hanging TUI
- Do not edit backlog md files directly — only use the backlog tool
- Run backlog commands with timeout of 30 seconds

## Exploration and Tangents
- Any new avenue, tangent, or idea discovered during work MUST be captured as a new backlog task
- This includes: interesting code paths, potential security findings, undocumented APIs, unexpected technologies, follow-up investigations, etc.
- Use `backlog task create "Title" -d "Description"` to capture these immediately so nothing is lost

## Security: Tokens and PII
- Auth tokens, session cookies, credentials, and PII MUST be gitignored
- Store tokens/credentials in `secrets/` directory (gitignored)
- E2E tests requiring login should read tokens from `secrets/auth_token` or environment variables
- Never commit real user data, names, CPR numbers, or school identifiers to the repo
- Test fixtures should use synthetic/anonymized data only
- Run `/aula-pii-scan` before every push to verify no PII leaked into tracked files or git history
- The PII terms list and scan skill live outside the repo (in `~/.claude/projects/`) and must NOT be committed or distributed -- they contain the actual PII values to scan for

## Workflow
- Use `nix-shell --run 'tool args'` for all tool invocations
- Extracted/decompiled files are gitignored — only project files and analysis docs are committed
- Run `just e2e` before committing if a justfile exists

<!-- BACKLOG.MD GUIDELINES START -->
# Instructions for the usage of Backlog.md CLI Tool

## Backlog.md: Comprehensive Project Management Tool via CLI

### Assistant Objective

Efficiently manage all project tasks, status, and documentation using the Backlog.md CLI, ensuring all project metadata
remains fully synchronized and up-to-date.

### Core Capabilities

- ✅ **Task Management**: Create, edit, assign, prioritize, and track tasks with full metadata
- ✅ **Search**: Fuzzy search across tasks, documents, and decisions with `backlog search`
- ✅ **Acceptance Criteria**: Granular control with add/remove/check/uncheck by index
- ✅ **Definition of Done checklists**: Per-task DoD items with add/remove/check/uncheck
- ✅ **Board Visualization**: Terminal-based Kanban board (`backlog board`) and web UI (`backlog browser`)
- ✅ **Git Integration**: Automatic tracking of task states across branches
- ✅ **Dependencies**: Task relationships and subtask hierarchies
- ✅ **Documentation & Decisions**: Structured docs and architectural decision records
- ✅ **Export & Reporting**: Generate markdown reports and board snapshots
- ✅ **AI-Optimized**: `--plain` flag provides clean text output for AI processing

### Why This Matters to You (AI Agent)

1. **Comprehensive system** - Full project management capabilities through CLI
2. **The CLI is the interface** - All operations go through `backlog` commands
3. **Unified interaction model** - You can use CLI for both reading (`backlog task 1 --plain`) and writing (
   `backlog task edit 1`)
4. **Metadata stays synchronized** - The CLI handles all the complex relationships

### Key Understanding

- **Tasks** live in `backlog/tasks/` as `task-<id> - <title>.md` files
- **You interact via CLI only**: `backlog task create`, `backlog task edit`, etc.
- **Use `--plain` flag** for AI-friendly output when viewing/listing
- **Never bypass the CLI** - It handles Git, metadata, file naming, and relationships

---

# ⚠️ CRITICAL: NEVER EDIT TASK FILES DIRECTLY. Edit Only via CLI

**ALL task operations MUST use the Backlog.md CLI commands**

- ✅ **DO**: Use `backlog task edit` and other CLI commands
- ✅ **DO**: Use `backlog task create` to create new tasks
- ✅ **DO**: Use `backlog task edit <id> --check-ac <index>` to mark acceptance criteria
- ❌ **DON'T**: Edit markdown files directly
- ❌ **DON'T**: Manually change checkboxes in files
- ❌ **DON'T**: Add or modify text in task files without using CLI

**Why?** Direct file editing breaks metadata synchronization, Git tracking, and task relationships.

---

## 1. Source of Truth & File Structure

### 📖 **UNDERSTANDING** (What you'll see when reading)

- Markdown task files live under **`backlog/tasks/`** (drafts under **`backlog/drafts/`**)
- Files are named: `task-<id> - <title>.md` (e.g., `task-42 - Add GraphQL resolver.md`)
- Project documentation is in **`backlog/docs/`**
- Project decisions are in **`backlog/decisions/`**

### 🔧 **ACTING** (How to change things)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eisbaw/aulalibre](https://github.com/eisbaw/aulalibre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
