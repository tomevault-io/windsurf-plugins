---
trigger: always_on
description: Armada is a multi-agent orchestration system that scales human developers with AI. It coordinates AI coding agents ("captains") to work on tasks ("missions") across git repositories ("vessels"). Written in C# (.NET), it exposes MCP tools for fleet, vessel, captain, mission, voyage, dock, signal, and merge queue management.
---

## Project Context
Armada is a multi-agent orchestration system that scales human developers with AI. It coordinates AI coding agents ("captains") to work on tasks ("missions") across git repositories ("vessels"). Written in C# (.NET), it exposes MCP tools for fleet, vessel, captain, mission, voyage, dock, signal, and merge queue management.

IMPORTANT -- Context Conservation: When using Armada MCP tools, use armada_enumerate with a small pageSize (10-25) to conserve context. Use filters (vesselId, status, date ranges) to narrow results. Only set include flags (includeDescription, includeContext, includeTestOutput, includePayload, includeMessage) to true when you specifically need that data -- by default, large fields are excluded and length hints are returned instead.

## Code Style
For C#: no var, no tuples, using statements instead of declarations, using statements inside the namespace blocks, XML documentation, public things named LikeThis, private things named _LikeThis, one entity per file, null check on set where appropriate and value-clamping to reasonable ranges where appropriate

# Mission Instructions

You are an Armada captain executing a mission. Follow these instructions carefully.

## Mission
- **Title:** Update MERGING.md, CLAUDE.md, and README.md — remove list_* references
- **ID:** msn_mms2pfqh_pt29d2cr7Eq
- **Voyage:** vyg_mms2pcu2_8pkQPfH9UGD

## Description
CONTEXT: Three additional files reference armada_list_* APIs which have been completely removed.

FILES TO MODIFY:
- docs/MERGING.md
- CLAUDE.md (project root)
- README.md (project root)

DO NOT modify any other files.

TASK 1 — docs/MERGING.md:
- Line 7: Replace `armada_list_merge_queue` in the intro paragraph with `armada_enumerate` with entityType 'merge_queue'. Example: "The merge queue is managed through MCP tools (`armada_enqueue_merge`, `armada_process_merge_queue`, `armada_enumerate` with entityType 'merge_queue', etc.)"
- Line 79: Replace the monitoring guidance. Change from mentioning both `armada_list_merge_queue` and `armada_enumerate` to just `armada_enumerate`: "Use `armada_enumerate` with entityType 'merge_queue' and status 'Failed' to check for entries that may need attention."
- Line 91: Remove `armada_list_merge_queue` from the tool reference table.
- Search the entire file for any remaining 'armada_list' references.

TASK 2 — CLAUDE.md (project root):
- Line 4: Rewrite the context conservation note. Remove all references to armada_list_*. The note currently says "prefer armada_enumerate over armada_list_* tools". Since list tools no longer exist, rewrite to simply state best practices for enumerate:
  "IMPORTANT — Context Conservation: When using Armada MCP tools, use armada_enumerate with a small pageSize (10-25) to conserve context. Use filters (vesselId, status, date ranges) to narrow results. Only set include flags (includeDescription, includeContext, includeTestOutput, includePayload, includeMessage) to true when you specifically need that data — by default, large fields are excluded and length hints are returned instead."

TASK 3 — README.md (project root):
- Line 503: Replace the tool examples. Change from mentioning `armada_list_missions` and `armada_list_events` to enumerate equivalents:
  FROM: "your MCP client can call tools like `armada_status`, `armada_dispatch`, `armada_list_missions`, `armada_cancel_voyage`, `armada_list_events`, and more."
  TO: "your MCP client can call tools like `armada_status`, `armada_dispatch`, `armada_enumerate`, `armada_voyage_status`, `armada_cancel_voyage`, and more."
- Search the entire file for any remaining 'armada_list' references.

## Repository
- **Name:** Armada
- **Branch:** armada/claude-code-1/msn_mms2pfqh_pt29d2cr7Eq
- **Default Branch:** main

## Rules
- Work only within this worktree directory
- Commit all changes to the current branch
- Commit and push your changes -- the Admiral will also push if needed
- If you encounter a blocking issue, commit what you have and exit
- Exit with code 0 on success
- Do not use extended/Unicode characters (em dashes, smart quotes, etc.) -- use only ASCII characters in all output and commit messages
- Do not use ANSI color codes or terminal formatting in output -- keep all output plain text

## Avoiding Merge Conflicts (CRITICAL)

You are one of several captains working on this repository. Other captains may be working on other missions in parallel on separate branches. To prevent merge conflicts and landing failures, you MUST follow these rules:

1. **Only modify files explicitly mentioned in your mission description.** If the description says to edit `src/routes/users.ts`, do NOT also refactor `src/routes/orders.ts` even if you notice improvements. Another captain may be working on that file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jchristn/Armada](https://github.com/jchristn/Armada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
