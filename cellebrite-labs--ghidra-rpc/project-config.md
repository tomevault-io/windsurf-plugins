---
trigger: always_on
description: `ghidra-rpc` is a skill that wraps Ghidra's reverse engineering capabilities in a
---

# Agent Instructions for ghidra-rpc

## What This Is

`ghidra-rpc` is a skill that wraps Ghidra's reverse engineering capabilities in a
CLI daemon communicating over Unix domain sockets. It lets an AI assistant analyze
binaries, decompile functions, trace cross-references, rename symbols, and annotate
findings — all by running `ghidra-rpc` commands from the shell.

## Project Layout

```
ghidra-rpc/
├── SKILL.md              — Skill prompt (what the assistant sees when triggered)
├── package.json          — pi package metadata
├── pyproject.toml        — Python package config (entry points, deps)
├── README.md             — Human-facing overview
├── TODO.md               — Open bugs, open features, verified-working table
│
├── docs/                 — Extended documentation (loaded on demand from SKILL.md)
│   ├── install.md        — Prerequisites, installation, GHIDRA_INSTALL_DIR setup
│   ├── quickstart.md     — First-session walkthrough
│   ├── troubleshooting.md — Common failure modes and fixes
│   ├── internals.md      — Implementation details: session persistence, known gotchas,
│   │                        Ghidra API reference, background-start mechanics
│   └── flows/            — Workflow guides for specific RE tasks
│       ├── binary-audit.md
│       ├── multi-binary.md
│       ├── patch-analysis.md
│       └── vulnerability-research.md
│
├── ghidra_rpc/           — Python package
│   ├── __init__.py       — Version
│   ├── cli.py            — Click CLI (ghidra-rpc entry point, all user commands)
│   ├── client.py         — Unix socket client (send_request, auto-restart logic)
│   ├── daemon.py         — Daemon lifecycle (start_blocking, start_background, stop)
│   │                       start_background() explicitly forwards GHIDRA_INSTALL_DIR
│   │                       from session or current env to the child process.
│   ├── session.py        — Session persistence
│   │                       • Socket path: /tmp/ghidra-rpc-<hash>.sock
│   │                       • Session file: <gpr-dir>/.ghidra-rpc-<hash>.json (default)
│   │                         or $GHIDRA_RPC_STATE_DIR/<hash>.json
│   │                       • Fields: mode, project_gpr, socket_path, ghidra_install_dir
│   │
│   └── server/           — Daemon internals (runs inside Ghidra's JVM via pyghidra)
│       ├── __init__.py
│       ├── main.py       — Socket server, request dispatch, handler registry
│       ├── launcher.py   — Ghidra init (headless & GUI), macOS framework Python re-exec
│       ├── _gui_launcher.py — GUI launcher (adapted from pyghidra-mcp)
│       ├── context.py    — HeadlessContext, GuiContext, DecompilerPool, ProgramInfo,
│       │                   _run_analysis() (supports timeout + best-effort cancel)
│       └── tools/        — Command handlers (one module per domain)
│           ├── __init__.py     — register_all_tools()
│           ├── analysis.py     — load (analyze=, analysis_timeout=), list_binaries,
│           │                     list_project_programs, save, functions (with
│           │                     address_min/max range filter and with_body),
│           │                     imports, exports, metadata, relocations,
│           │                     list_calling_conventions
│           ├── decompiler.py   — decompile, _find_function (name/address resolution)
│           ├── search.py       — strings, symbols, find_bytes (byte pattern search)
│           ├── xrefs.py        — xrefs_to, xrefs_from
│           ├── navigation.py   — goto (GUI-only)
│           ├── bookmarks.py    — set_bookmark, list_bookmarks, remove_bookmark
│           ├── memory.py       — read_bytes (raw memory inspection),
│           │                     write_bytes (raw memory patching),
│           │                     memory_map (list all memory segments/sections)
│           ├── disassembly.py  — disassemble (warning field when address skipped),
│           │                     assemble (SLEIGH assembler: text → bytes)
│           ├── cfg.py          — basic_blocks (CFG from BasicBlockModel),
│           │                     pcode (raw listing P-code or high SSA P-code)
│           ├── tags.py         — tag_function, untag_function, list_tags,
│           │                     functions_by_tag
│           ├── data_types.py   — create_struct, create_union, create_enum,
│           │                     modify_enum, modify_struct,
│           │                     clear_data_range, apply_data_type_range,
│           │                     list_labels, list_data_types,
│           │                     set_equate (with enum-linking), list_equates
│           ├── version_tracking.py — version_track (Auto VT + BSim between
│           │                     two loaded binaries), match_function (find
│           │                     corresponding function via correlators),
│           │                     decompile_all (bulk decompile all functions),
│           │                     function_diff (normalised unified diff)
│           ├── processor_context.py — get_processor_context (read ISA register
│           │                     values at an address), set_processor_context
│           │                     (write ISA register over range — ARM TMode fix)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cellebrite-labs/ghidra-rpc](https://github.com/cellebrite-labs/ghidra-rpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
