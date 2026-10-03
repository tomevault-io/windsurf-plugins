---
trigger: always_on
description: <!-- BEGIN WARDEN RULES (managed by `warden init` — do not edit inside this block) -->
---

<!-- BEGIN WARDEN RULES (managed by `warden init` — do not edit inside this block) -->
# Warden — Context Governance and Verification Layer

This project uses Warden, an MCP server that manages context across eight layers:
code intelligence, context selection, tool output pruning, agent memory, response
compression, file compression, MCP description compression, and session continuity.
It proves every optimization is safe via a shadow-mode eval gate.

## Session start (IMPORTANT — do this first)

At the start of every session in this project, do THREE things in order:

1. Call `warden_handoff` with `{ read: true, repoRoot: "<project-root>" }` to read the previous session's
 handoff document. This gives you the essential state from the last session —
 decisions made, tasks completed, files touched — so you pick up where the
 previous session left off instead of starting from scratch. Print a one-line
 summary to the user: "Previous session: X decisions, Y tasks, Z files touched."

2. Call `warden_status` with `{ repoRoot: "<project-root>" }` and print a one-line summary:
 "Warden active — X tokens saved so far, Y rules live."

3. Call `warden_memory_recall` with `{ query: "<task-relevant-query>", repoRoot: "<project-root>" }` to find
 relevant past decisions. Print any results that are relevant.

This gives the user visible proof that Warden is working and surfaces project
context from previous sessions.

### CRITICAL: Always pass repoRoot on memory/status/handoff calls

The MCP server is a long-running process whose working directory is fixed at
startup. If you switch projects, the server may still be using a different
project's database. To ensure memories, status, and handoffs are scoped to the
correct project, ALWAYS pass `repoRoot` (your current project root directory)
on these calls:
- `warden_status({ repoRoot: "..." })`
- `warden_memory_save({ ..., repoRoot: "..." })`
- `warden_memory_recall({ ..., repoRoot: "..." })`
- `warden_memory_list({ ..., repoRoot: "..." })`
- `warden_memory_forget({ ..., repoRoot: "..." })`
- `warden_record_outcome({ ..., repoRoot: "..." })`
- `warden_handoff({ ..., repoRoot: "..." })`
- `warden_outcome_stats({ repoRoot: "..." })`

Use the absolute path to your project root (the directory containing .git or
package.json). This ensures each project's memories stay isolated.

### If warden_status fails (transport error, tool not found, etc.)

If the warden_status call fails, the MCP server is not connected. Tell the user:
"Warden MCP server not connected. Restart your IDE or run `warden doctor` in a
terminal to diagnose." Then continue with your built-in tools — do NOT silently
skip Warden. The user needs to know it's not working so they can fix it.

## Layer 1: Before starting work — context selection

BEFORE diving into a task, call `warden_context_select` with the task description.
It scans and recommends which files to read, so you load only
relevant context instead of everything.

- Parameters: task (required), repoRoot, maxFiles
- Typical: warden_context_select({ task: "fix null pointer in auth.ts" })
- Read the recommended files first, then proceed with the task.

## Layer 2: During work — tool output pruning

ALWAYS use the Warden wrapper tools instead of your built-in equivalents. The
Warden tools do the same work AND prune the output automatically — no extra
step needed. This is not optional. Every tool call that could produce large
output should go through Warden.

### Enforcement hooks (automatic)

If `warden init` installed hooks (Step 2c), built-in Read/Grep calls are
automatically blocked and redirected to Warden wrappers. You don't need to
remember — the hook intercepts the call and tells you to use the Warden
wrapper instead. If you see a "BLOCKED: Use warden_file_read instead" message,
that's the hook working as intended. call the Warden wrapper tool.

1. **Searching code**: Use `warden_grep` INSTEAD OF your built-in grep/search.
 - It searches files and returns only the matches relevant to the current task.
 - Parameters: pattern (required), path, glob, ignoreCase, maxResults
 - Typical: warden_grep({ pattern: "function auth", path: "src", glob: "*.ts" })

2. **Reading files**: Use `warden_file_read` INSTEAD OF your built-in file read.
 - It reads the file and returns a pruned version (slice + outline for large files).
 - Parameters: filePath (required), startLine, endLine
 - Code is never rewritten — only included or excluded.

3. **Running tests**: Use `warden_run_tests` INSTEAD OF running tests directly.
 - It runs the test command and keeps failures + context, collapses passing noise.
 - Parameters: command (default: "npm test"), cwd

4. **Running commands**: Use `warden_run_command` INSTEAD OF running shell commands.
 - It runs the command and prunes low-signal output, keeping errors and relevant content.
 - Parameters: command (required), cwd, timeout

## Layer 3: After making decisions — memory

When you make a durable project decision (architecture choice, library selection,
convention, constraint), call `warden_memory_save` to persist it:

- Parameters: category, title, body, tags, source, repoRoot
- Categories: "decision" | "finding" | "pattern" | "constraint" | "preference"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rynald0cst0ltziam/Warden-AI](https://github.com/rynald0cst0ltziam/Warden-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
