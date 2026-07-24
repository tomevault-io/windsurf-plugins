---
trigger: always_on
description: <!-- AUTO-GENERATED — DO NOT EDIT. Built by `ai update-md` from AGENTS-Source.md (this folder) + AGENTS-Suffix.md (AgentCli). To change anything below, edit the source file(s) and re-run `ai update-md`. -->
---

<!-- AUTO-GENERATED — DO NOT EDIT. Built by `ai update-md` from AGENTS-Source.md (this folder) + AGENTS-Suffix.md (AgentCli). To change anything below, edit the source file(s) and re-run `ai update-md`. -->

# Project-specific Rules for ActualLab.Fusion

**YOU MUST READ [CODING_STYLE.md](CODING_STYLE.md) before writing or
modifying any C# code.** It's not optional. This project
**deviates from standard .NET conventions** on several points (notably:
no `Async` suffix on async methods; no XML docs on members; mixed brace
style). Default instincts from elsewhere will produce code that gets
rejected. If you haven't opened that file yet in this session, stop and
read it now.

**You MUST NOT write a single comment, docstring, or XML doc** without
first reading [CODING_STYLE.md → "Regular comments, docstrings, XML
documentation comments"](CODING_STYLE.md#regular-comments-docstrings-xml-documentation-comments).
You have a strong tendency to over-comment and to restate what the code
already says; that section explains exactly when a comment is justified
and when it isn't. Re-read it any time you're tempted to add a `//` or `///`.

# Git workflow — don't branch unless asked

Commit your changes directly to the default branch (`master`). **You typically
should NOT create a feature branch in this repo unless the user explicitly asks
for one.** Small, self-contained changes (docs, fixes, tweaks) belong on
`master`; a needless branch only adds a merge step later.

# Type Catalog — Reuse Existing Abstractions (CRITICAL)

This codebase is mature. **Reusing what already exists is more important
than writing something new.** A new helper that duplicates an existing one
is a defect, not a feature. **Always look first.**

Use [`docs/api-index.md`](docs/api-index.md) to discover existing
abstractions before writing new code. For the complete list, see
[`docs/api-index-full.md`](docs/api-index-full.md).

## Planning rule (mandatory)

**Every implementation plan MUST include a "Reuse" section** with two parts:

1. **Existing abstractions to reuse.** Research first. List the concrete
   types/functions you intend to call from the indexes above. 
   If you cannot find a fit, say so explicitly — silence is not acceptable.

2. **Reusability of new components.** For every new component the plan
   introduces, ask: *is this likely useful elsewhere?* If yes, the plan
   **must list an option to put it in a shared project** instead of the
   feature-specific one:
   - **C#**: `ActualLab.Core`
   - **TypeScript**: `ts/actuallab-core`.

   The plan should compare the local-vs-shared placement and recommend
   one. Default to shared when in doubt — promoting later is harder than
   placing correctly the first time.

If the work is small enough that you skip a written plan, you still owe
yourself the "look first" step: search the indexes for keywords related
to what you're about to write.

# Fusion documentation & source MCP

This repo backs a public, read-only MCP server at
`https://fusion.actuallab.net/mcp` (see [`docs/mcp-server.md`](docs/mcp-server.md)).
When it's connected, prefer it over guessing: it can search and read the Fusion
**docs** (mental model, glossary, API index) and the Fusion **source** — find
files or declarations by regex and read exact line ranges, or ripgrep across
`src` / `samples` / `tests`. The individual tools are self-describing once the
MCP is enabled, so they are not enumerated here.

# Use cross-platform PowerShell

`pwsh` (cross-platform PowerShell) command is available on any OS you run, so use it.

Before starting any task, read AGENTS.md files in every directory starting from the current one and above, up to the root one (project directory).

# Execution policy after plan approval

Once a plan is approved and the open questions in it have been resolved,
**push it to completion without stopping for confirmation between steps.**
Don't ask permission to move from one pre-approved step to the next.
Don't pause to summarize "I'm about to do X" between pre-agreed phases.
Don't ask the user to choose when the choice has minimal impact.

You stop and ask only when **all** of these are true:

1. You hit a **real obstacle** you can't resolve from context alone.
2. The choice **likely obsoletes the plan or forces significant rework** —
   not "minor implementation detail," but "the path branches into two very
   different futures."
3. Your best guess at the right answer has a **non-trivial chance of being
   wrong in a way that's hard to revert**.

Concretely, do NOT ask when:
- The next step is a mechanical consequence of an earlier approved step.
- Two options exist and either is reversible in a few minutes.
- One option is clearly best (≥ ~80% probability) on the available evidence.
- You're already mid-plan and the next step is just "keep going."
- The build is broken between phases and the user already said that's fine.

When in doubt, **act**, then briefly note the choice in the result so the
user can correct course if needed. A short "I picked X because Y; flag if
you'd prefer Z" beats a question that stalls progress.

# Building


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ActualLab/Fusion](https://github.com/ActualLab/Fusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
