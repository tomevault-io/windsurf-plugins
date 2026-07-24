---
trigger: always_on
description: - Treat reverse engineering as evidence-driven work. Prefer proven behavior from runtime traces, dumps, debugger state, existing overrides, and documented observations over speculation.
---

# Cryogenic Workspace Instructions

## Workflow
- Treat reverse engineering as evidence-driven work. Prefer proven behavior from runtime traces, dumps, debugger state, existing overrides, and documented observations over speculation.
- When behavior is unclear, check live Spice86 state first if available. Spice86 now exposes MCP tooling and runtime inspection surfaces; use that, the debugger/GDB flow, or dump artifacts before inventing logic. Reference: https://github.com/OpenRakis/Spice86/blob/master/doc/mcp.md
- Use current repository evidence as the source of truth: `dump/spice86dumpExecutionFlow.json`, `dump/spice86dumpGhidraSymbols.txt`, `dump/CodeGeneratorConfig.json`, generated globals, and any targeted memory dumps captured by the project.
- Preserve observed game behavior, including quirks, unless you have strong runtime evidence that the current behavior is wrong.
- No new fallback behavior from Copilot: do not invent or add new silent fallback paths or degraded alternate implementations. Existing fallback behavior already present in the repository is allowed and should be preserved unless explicitly requested to change. If required evidence or functionality is missing, fail fast with explicit logging instead of masking the issue.
- Link to existing docs instead of restating them. Use `README.md` for architecture and run commands, and `CONTRIBUTING.md` for override workflow and coding examples.
- When approaching an architectural question (what functions share state, which data structures cross file boundaries, how a driver subsystem is structured), query the knowledge graph before grepping raw files. See the **Knowledge Graph** section below.

## Knowledge Graph (graphify)

graphify builds a persistent, queryable knowledge graph from the Cryogenic corpus — disassembly, C# override source, documentation, and dump artifacts. Use it as a structural index before diving into raw files.

**Install (already done):**
```bash
pip install graphifyy
python -m graphify copilot install   # writes skill to ~/.copilot/skills/graphify/SKILL.md
```

**Build the graph** — invoke the graphify skill from your AI assistant (VS Code Copilot Chat or GitHub Copilot CLI) with the relevant corpus paths. The skill orchestrates local AST extraction and semantic extraction in parallel:
```
/graphify docs/ dump/live/dnadp_full_disasm.asm src/Cryogenic/Overrides/ src/Cryogenic.AdpPlayer/Services/ --directed --mode deep

# Incremental update after new evidence or overrides change:
/graphify docs/ ... --update

# Generate navigable wiki articles per subsystem community:
/graphify docs/ ... --wiki
```

A `.graphifyignore` at workspace root excludes build outputs, binary dumps, and submodule trees.
See `graphify-out/GRAPH_REPORT.md` for the latest god-nodes and surprising-connection summary.

**Query the graph** (CLI, works after graph.json exists):
```bash
python -m graphify query "DNADP OPL scheduler tick dispatch" --graph graphify-out/graph.json
python -m graphify path "ProgramChange" "OplWriteInstrument" --graph graphify-out/graph.json
python -m graphify explain "RotateRight16" --graph graphify-out/graph.json
```

**Graph edges are tagged** EXTRACTED (found directly in source), INFERRED (confidence score attached), or AMBIGUOUS (flagged for review). Never treat INFERRED edges as ground truth — verify from runtime evidence.

## Architecture
- Cryogenic runs the original `DNCDPRG.EXE` inside Spice86 and replaces routines with C# overrides; always pass `--UseCodeOverride true` when launching or your code will never run.
- The entry chain is `src/Cryogenic/Program.cs` -> `DuneCdOverrideSupplier` -> `Overrides.Overrides`. Extend the existing partial `Overrides` class so override registration stays centralized.
- Segment fields in `Overrides` are address anchors for the entire project. Never change `cs1`, `cs2`, `cs3`, `cs4`, or `cs5` without auditing every `DefineFunction` and `DoOnTopOfInstruction` registration.
- Keep overrides grouped by domain in the existing partial files such as `MapCode`, `DialoguesCode`, `DisplayCode`, `VideoCode`, and `VgaDriverCode`.

## Reverse Engineering Rules
- Register new behavior through `DefineFunction` for call replacements and `DoOnTopOfInstruction` for inline hooks. Do not bypass those registration APIs.
- Keep the existing method naming pattern `{FunctionName}_{Segment}_{Offset}_{LinearAddress}` so each override remains traceable to the DOS disassembly.
- Return with `NearRet()` or `FarRet()` to match the original instruction. Getting this wrong corrupts the emulated stack.
- Treat stack operations as observable behavior: when original assembly uses `PUSH`/`POP` (including temporary register saves), preserve equivalent stack/register effects in overrides (prefer `Stack.Push16`/`Stack.Pop16` where practical).
- Preserve `PUSHF`/`POPF` semantics as well (full FLAGS contract), not only `InterruptFlag`, unless live evidence proves the extra flags are caller-ignored.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenRakis/Cryogenic](https://github.com/OpenRakis/Cryogenic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
