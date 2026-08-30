---
trigger: always_on
description: When `.ostk/` exists in the project run `ostk boot` first to load the kernel.
---

# Kernel
When `.ostk/` exists in the project run `ostk boot` first to load the kernel.
Run `ostk shutdown` (canonical, no `kernel` prefix) to cleanly terminate.

If `ostk help` or `[ctx]` envelopes show fewer resident tools than this doc
lists (e.g. `bash, fs_ops, read, lock, spawn, interact, tack, help, session`
missing), the project's `.language` is stale (pre-→1489 schema, blank column
10). Run `ostk mcp diag` to confirm, then `ostk shutdown` triggers the
→1597 backfill, OR delete `.ostk/.language` and re-boot to seed fresh. The
fix is in shutdown's `compile_language`; it is idempotent and silent.

## Self-documenting surface (read this when something is missing)
The kernel emits its own canonical reference; trust it over older docs:
  - `ostk --agents`     — agent-facing one-stop guide (syscalls, signals, vocab, do_not)
  - `ostk --help`       — full CLI tree, organized by section: SYSTEM ABI / DAILY WORK
                          / WORK PIPELINE / IDENTITY & SECURITY / RECOVERY & FLEET /
                          DOCUMENTATION & SPECS / INFRASTRUCTURE / DIAGNOSTICS / SYSTEM SETUP
  - `ostk <verb> --help` — per-verb help (every verb above)
  - `ostk mcp diag`     — verify tool surface integrity (seed × present × core × abi)

The structural split: **System ABI** verbs (frozen, agent-callable, exposed
as MCP tools) live under the SYSTEM ABI section. Operator-overlay verbs
(`show`, `work`, `decide`, `trace`, `commit`, `recall`) live under DAILY
WORK / WORK PIPELINE — they are CLI-only by design (per →1589 epic). The
`mcp__ostk__*` namespace mirrors the System ABI exactly. If you need a
verb that isn't in the MCP namespace, reach for `ostk <verb>` via `bash`.

In-flight ABI alignment work: EPIC →1821 (`align ABI verb contracts across
CLI, MCP, docs, and dispatch`) tracks remaining drift between these
surfaces. If you hit a `kernel verb X has no internal handler` or a
schema mismatch, check the children of →1821 before fixing locally —
several are already filed (→1638 fs_ops dispatch gap, →1820 arrive
schema, →1822/→1826 recall surface, →1824 verb_load).

# Tool routing — kernel tools replace Claude native tools (→1287, →1344, →1326)

The MCP-namespaced ostk tools are the kernel-equivalent replacements for
Claude Code's native tools. Underneath they track file generations, compress
output, detect OCC conflicts, and write audit rows. Your muscle memory
(Bash, Read, Edit, Write, Grep, Glob) lands on the right tool.

| reach-for (native) | use (MCP-kernel)                   | simplest call                                       | what the kernel adds |
|--------------------|------------------------------------|-----------------------------------------------------|----------------------|
| Bash               | `mcp__ostk__bash`                  | `bash(cmd="cargo test", cwd="src/")`                | audit + compression + gen_table invalidation; pass `cwd` instead of `cd X && …` |
| Read               | `mcp__ostk__read`                  | `read(path="src/main.rs")`                          | gen_table tracking + elision-aware output |
| Edit               | `mcp__ostk__fs_ops` (CAS edit)     | `fs_ops(path="src/main.rs", old_str="X", new_str="Y")` | CAS str_replace with OCC conflict detection |
| Write              | `mcp__ostk__fs_ops` (write/create) | `fs_ops(path="src/new.rs", new_str="<content>")`    | gen_table + audit |
| Grep               | `mcp__ostk__search` (mode=content) | `search(query="fn main")`                           | full search substrate (code + memory + transcripts) |
| Glob               | `mcp__ostk__search` (mode=files)   | `search(query="*.rs", mode="files")`                | file-pattern search on the same engine |

`fs_ops` is the single file-mutation verb. Quick mode uses `path` + `old_str` +
`new_str` for CAS edits, or `path` + `new_str` (no `old_str`) for create/write.
Batch mode (below) stacks multiple mutations in one call.

Legacy aliases (`shell`, `fs_read`, `fs_write`, `find`, `grep`, `glob`, `recall`,
`pitchfork`, `context_search`, `near`, `related`, `investigate`, capitalized
`Bash`/`Edit`/`Read`/`Write`/`Grep`/`Glob`/`WebRead`/`WebLinks`/`WebStatus`)
all still resolve via `normalize_tool_name` — they do NOT appear in
`tools/list` anymore, but calls using the old name still execute. Plan to
migrate; don't plan to rely on them.

If the harness shows you BOTH native and MCP-kernel tools, prefer the
MCP-kernel versions when touching files the kernel tracks. The native ones
bypass audit, gen_table, and OCC — you'll pay that cost in silent staleness
and missing history.

## Subagent policy
Never spawn `Explore` subagents — they use built-in Read/Grep/Glob which bypass the kernel.
Use `general-purpose` agents with explicit ostk tool instructions in the prompt instead.
All subagent prompts MUST include: "Use ostk MCP tools (read, search, bash, fs_ops) — kernel-equivalent of native tools with audit and gen_table. File edits go through fs_ops; code/file search goes through search."

## File I/O
  read(path="src/main.rs")                                   — read file (gen_table tracked, elision-aware)
  read(path="src/main.rs", enrich=true)                      — read with driver diagnostics (inline errors)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [os-tack/ostk-cache](https://github.com/os-tack/ostk-cache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
