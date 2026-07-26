---
trigger: always_on
description: LLM-friendly static and dynamic analysis toolkit for x86/x64 PE binaries (`.exe` / `.dll`), combining static analysis (`retools`), dynamic analysis (`livetools` via Frida), and D3D9 frame tracing. Work is organized around per-game knowledge base files (`kb.h`) that accumulate discoveries and feed back into richer decompilation.
---

# Vibe Reverse Engineering — Agent Instructions

LLM-friendly static and dynamic analysis toolkit for x86/x64 PE binaries (`.exe` / `.dll`), combining static analysis (`retools`), dynamic analysis (`livetools` via Frida), and D3D9 frame tracing. Work is organized around per-game knowledge base files (`kb.h`) that accumulate discoveries and feed back into richer decompilation.

This file is the canonical, harness-agnostic instruction set. Claude Code loads it via `.claude/CLAUDE.md`; Cursor, Copilot, Codex, and other agents read it directly. Deeper references live under `.claude/` and apply to every harness:

- **Tool catalog** (every retools / livetools / dx9tracer command, syntax, caveats): `.claude/references/tool-catalog.md`
- **Tool dispatch** (which tool for which question; what runs inline vs delegated): `.claude/rules/tool-dispatch.md`
- **Analysis workflow** (bootstrap ordering, Ghidra backend, index/query, parallel patterns): `.claude/rules/subagent-workflow.md`
- **Skills** (`dx9-ffp-port`, `dynamic-analysis`): `.claude/skills/` — canonical copies. Non-Claude harnesses self-install them (see "Skill Setup" below).
- **Custom agents** (`static-analyzer`): `.claude/agents/` — canonical definitions. `.cursor/agents/`, `.github/agents/`, and `.kiro/agents/` hold thin harness-native mirrors that defer to the canonical files, so only the `.claude/agents/` copies get edited.

## Skill Setup (do this first on non-Claude harnesses)

Claude Code reads `.claude/skills/` natively — skip this section. Every other harness: check whether the `dx9-ffp-port` and `dynamic-analysis` skills are available to you. If not, install them into your own skills directory before starting work:

```bash
npx skills add ./.claude/skills -a <your-agent> -y   # e.g. -a cursor, -a copilot, -a kiro-cli
```

Omit `-a` to let the CLI auto-detect your harness. The source path must be `./.claude/skills` — a bare `.` finds nothing (the CLI skips the current project's agent directories).

If `npx` is unavailable or your harness isn't supported by the CLI, copy manually: each `.claude/skills/<name>/` folder goes verbatim into your harness's skills directory (Cursor: `.cursor/skills/`, Copilot: `.github/skills/`, Kiro: `.kiro/skills/`, `.agents/skills/` for agents following that convention).

Installed copies land in git-ignored paths (`.agents/`, `.cursor/skills/`, `.github/skills/`, `.kiro/skills/`, `skills-lock.json`) — never commit them, and never edit them: the canonical copies in `.claude/skills/` are the only ones that get edited. Re-install after pulling changes that touch `.claude/skills/`.

## Read-Only Templates

These directories are **shared tooling and templates**. Do not modify them for game-specific work — per-game changes go in `patches/<GameName>/`.

- `rtx_remix_tools/dx/remix-comp-proxy/` — proxy framework **template** (copied per-game)
- `rtx_remix_tools/dx/scripts/` — DX9 analysis scripts (shared tooling)
- `retools/` — static analysis toolkit (shared tooling)
- `livetools/` — Frida-based dynamic analysis (shared tooling)
- `graphics/` — DX9 tracer framework (shared tooling)

**Per-game work goes in `patches/<GameName>/`.** When starting a new game, copy `rtx_remix_tools/dx/remix-comp-proxy/` (excluding `build/`) to `patches/<GameName>/` and edit the copy. If the user says "edit remix-comp-proxy code" without specifying, ask whether they mean the template or a game copy.

Shared tooling can be modified to improve the tools themselves — just not for game-specific customization.

## Project Workspace

Use `patches/<project_name>/` (git-ignored) for all project-specific artifacts: knowledge base files (`kb.h`), one-off analysis scripts, ASI patch specs and builds, notes, logs, and collected trace data. Create the project subfolder on first use.

### Backups

Before modifying project files (proxy source, kb.h, proxy.ini, build scripts, ASI specs), create a timestamped backup capturing the last known-good state:

```
patches/<project>/backups/YYYY-MM-DD_HHMM_<short-description-slug>/
```

Copy ALL files being modified into the backup folder before making changes.

### Knowledge Base

Maintain `patches/<project>/kb.h` while reverse engineering a binary. Format: C types (no prefix), functions (`@` prefix), globals (`$` prefix):

```c
struct Foo { int x; float y; };
@ 0x401000 void __cdecl ProcessInput(int key);
$ 0x7C5548 Object* g_mainObject
```

Update the KB when you: identify a function's purpose, reconstruct a struct, identify a global, find magic constants (define an enum), or resolve RTTI class names. Always pass `--types patches/<project>/kb.h` to the decompiler once a KB exists.

## Working Method


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ekozmaster/Vibe-Reverse-Engineering](https://github.com/Ekozmaster/Vibe-Reverse-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
