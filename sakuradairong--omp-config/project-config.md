---
trigger: always_on
description: This file configures agent behavior for the omp environment.
---

# Oh My Pi — Agent Instructions

This file configures agent behavior for the omp environment.
Skills are installed at `~/.omp/skills/` and loaded via `skill://<name>`.

## Workflow: always work in this order

For any non-trivial task, progress through these phases.
Use `/skill:<name>` or `read skill://<name>` to load the full skill.

```
Brainstorm → Plan → Implement/Execute → Verify → Review → Finish
```

| Phase | Skill | When |
|---|---|---|
| **Brainstorm** | `brainstorming` | Before creative work, features, design decisions |
| **Plan** | `writing-plans` | Before multi-step or multi-file changes |
| **Isolate** | `using-git-worktrees` | Before starting feature work (if workspace needs isolation) |
| **Parallel** | `dispatching-parallel-agents` | When 2+ independent tasks exist |
| **Execute** | `executing-plans` / `subagent-driven-development` | When executing a written plan |
| **Debug** | `systematic-debugging` | On any bug, test failure, or unexpected behavior |
| **Test** | `test-driven-development` | Before writing implementation code |
| **Verify** | `verification-before-completion` | Before claiming work is done |
| **Review** | `requesting-code-review` | Before merging or PR |
| **Finish** | `finishing-a-development-branch` | When implementation is complete |
| **Code Review (receiving)** | `receiving-code-review` | When responding to review feedback |

## Available Skills (16 total)

### From Superpowers
- `skill://using-superpowers` — Entry point: how to find and invoke skills
- `skill://brainstorming` — Structured ideation before implementation
- `skill://writing-plans` — Bite-sized implementation plans with task list
- `skill://dispatching-parallel-agents` — Fan out independent tasks
- `skill://subagent-driven-development` — Execute plans with parallel subagents
- `skill://executing-plans` — Single-session plan execution with checkpoints
- `skill://systematic-debugging` — 4-phase root cause debugging
- `skill://test-driven-development` — RED-GREEN-REFACTOR cycle
- `skill://verification-before-completion` — Evidence-based completion checks
- `skill://requesting-code-review` — Quality gates before merging
- `skill://receiving-code-review` — Handle review feedback with rigor
- `skill://finishing-a-development-branch` — Merge/PR/cleanup decisions
- `skill://using-git-worktrees` — Isolated workspace management
- `skill://writing-skills` — Creating and editing skills

### From astrbot-plugin-dev
- `skill://astrbot-plugin-development` — AstrBot plugin: structure, pitfalls, AI patterns

### Pre-installed
- `skill://spike` — Throwaway experiments to validate approaches

## Tool Priority (OMP convention)

Always prefer OMP's dedicated tools over shell commands:

| Task | Tool | Not |
|---|---|---|
| Read code | `read` | `cat`, `head`, `tail`, `sed -n` |
| Search code | `search` | `grep`, `rg`, `git grep` |
| Find files | `find` | `ls **/*`, `fd` |
| Edit code | `edit` (hashline) | `sed -i`, `echo >>` |
| Structural rewrite | `ast_edit` | `sed`, `awk` |
| Symbol-aware rename | `lsp rename` | `ast_edit`, `sed` |
| Code intelligence | `lsp` | blind text search |
| Debug | `debug` (DAP) | ad-hoc print/echo |
| Fetch web | `read <url>` | `curl`, `wget` |
| Parallel work | `task` | sequential bash |

## OMP Features to Leverage

| Feature | Mechanism | Benefit |
|---|---|---|
| Plan mode | `/plan` or `resolve(action:apply, extras:{plan...})` | Planner drafts read-only; approve before execute |
| Session branching | `/branch`, `/fork` | Explore alternatives without losing history |
| Context compaction | `/compact [focus]` | Free context window, keep summary |
| Subagents | `task` tool + IRC | Parallel investigation, DM between agents |
| LSP | `lsp references/definition/rename` | Cross-file refactors safely |
| Memory | `read memory://root` | Cross-session project knowledge |

## GitNexus — MCP Code Knowledge Graph

GitNexus indexes git repos into a code knowledge graph and exposes it via MCP (16 tools + 8 resources).

Used for: impact analysis, code search, symbol context, refactoring, change detection.

**MCP Server already configured** in both Hermes and OMP (`command: "gitnexus"`, `args: ["mcp"]`).

Index a repo:
```bash
cd /path/to/repo
gitnexus analyze
```

Tools available (prefixed with `mcp_gitnexus_`):
- `context` — 360° symbol view
- `impact` — blast radius analysis  
- `query` — hybrid search (BM25 + semantic)
- `detect_changes` — git diff impact mapping
- `rename` — cross-file coordinated rename

**IMPORTANT:** If you see `backup_script` as the only indexed repo, the user needs to `gitnexus analyze` their actual repos.

## Deep Patterns (learned from docs.omp.sh)

### Edit workflow (fail-safe)
```
1. read <file>:<range>   # capture §PATH header + LINEID|content
2. edit input="§<file>   # reference anchors exactly
   ≔ <anchors>
     <new code>"
3. lsp diagnostics       # verify after edit
```
If `edit` returns a stale-anchor error: re-`read` the file slice (anchors changed), then re-emit.

### GitHub as virtual FS
```
read pr://1234                     # metadata + comments
read pr://1234/diff/all            # full diff (hashline-anchorable)
read issue://?state=open&label=bug # list open bugs
github op=pr_create fill=true      # PR from commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sakuradairong/omp-config](https://github.com/sakuradairong/omp-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
