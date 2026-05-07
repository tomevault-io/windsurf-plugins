---
trigger: always_on
description: Catalog of all RE tools -- pick the right tool for the job
---


# Tool Catalog

**BEFORE FIRST USE**: Run `python verify_install.py` from the repo root. Do NOT proceed with any tool until every required check passes. If pyghidra/Ghidra shows as WARN, run `python verify_install.py --setup` to auto-download JDK 21 + Ghidra + pyghidra. Common failures: missing `git lfs pull` (LFS pointer stubs instead of binaries), missing `pip install -r requirements.txt`.

All tools work on PE binaries (`.exe` and `.dll`). `$B` = path to binary, `$VA` = hex address, `$D` = path to minidump `.dmp` file. Check tools help command for more info on usage.
Always consult this catalog before making any move to take the best decision on what to use with best bang for your buck.
Run all tools from the repo root directory using `python -m <module>` syntax (e.g. `python -m retools.search`). Do NOT modify files inside `retools/`, `livetools/`, or `graphics/` unless working on the tools themselves.

IMPORTANT: Collecting MORE INFORMATION per command run is encouraged over minor snippets of data/output that don't reveal the whole picture.

## Decision Guide

### Run Directly (main agent)

These are fast (<5s) and allowed inline:

- "What compiler built this?" → `python -m retools.sigdb fingerprint $B`
- "Is this a known library function?" → `python -m retools.sigdb identify $B $VA`
- "Get full context before reasoning about a function" → `python -m retools.context assemble $B $VA --project $P`
- "Clean up decompiler output with known names" → pipe through `python -m retools.context postprocess`
- "Read a typed value from the PE file" → `python -m retools.readmem $B $VA $TYPE`
- "What constant flows into this register?" → `python -m retools.dataflow $B $VA --constants`
- "Trace where this value comes from" → `python -m retools.dataflow $B $VA --slice TARGET_VA:REG`
- "Build an ASI patch DLL" → `python -m retools.asi_patcher build spec.json`

### Delegate to `static-analyzer` subagent

Everything else. Tell the subagent WHAT you need, not HOW to run it — it has the full tool catalog.

**D3D9-specific questions?** Check the DX analysis scripts section below first — they're faster and more targeted than general retools for D3D API usage, device calls, shader constants, and vertex formats.

- "What does this function do?" → decompile + callgraph + xrefs + dataflow --constants
- "Who calls this function?" → xrefs or callgraph --up
- "What does this function call?" → callgraph --down (add --indirect for vtable calls)
- "Who calls this virtual method?" → xrefs --indirect + filter by vtable slot offset
- "What constant reaches this call?" → dataflow --constants or --slice VA:REG
- "Resolve a switch/jump table" → cfg (auto-resolves MSVC switch patterns)
- "Find a string and who uses it" → string search with xrefs
- "Where is this global read/written?" → datarefs
- "Where is struct field +0x54 used?" → structrefs
- "What does this struct look like?" → structrefs --aggregate
- "What C++ class is this vtable?" → RTTI resolution
- "What type was a caught/thrown exception?" → RTTI throwinfo
- "Find instructions using a specific constant" → instruction search
- "What crashed and what was the error message?" → dump diagnosis + throwmap
- "Map all throw sites to error strings" → throwmap list
- "First time analyzing a binary?" → bootstrap (2-5 min) + pyghidra analyze (5-15 min) in parallel
- "Bulk signature scan" → sigdb scan (1-3 min)
- Any combination of the above

### Live tools (main agent, requires attached process)

- "Is this function reached at runtime?" → `livetools trace` or `collect`
- "What are the actual register values?" → `livetools trace --read` or `bp` + `regs`
- "How many draw calls happen?" → `livetools dipcnt`
- "Who writes to this memory address?" → `livetools memwatch`

### DX analysis scripts (main agent, fast first-pass)

These are targeted D3D9 scanners under `rtx_remix_tools/dx/scripts/`. They run in seconds and surface D3D-specific patterns that general-purpose retools would take longer to find. **Use these BEFORE retools** when the question is about D3D9 API usage, device calls, shaders, or vertex formats. Run as `python rtx_remix_tools/dx/scripts/<script> <args>`.

- "How does the game use D3D9?" → `find_d3d_calls.py <game.exe>` (imports + call sites)
- "Which VS constant registers hold matrices?" → `find_vs_constants.py <game.exe>` (SetVertexShaderConstantF call sites with register/count)
- "Which PS constant registers are used?" → `find_ps_constants.py <game.exe>` (SetPixelShaderConstantF/I/B with register/count)
- "Where does the game call the D3D device?" → `find_device_calls.py <game.exe>` (vtable call patterns + device pointer refs)
- "What render states does the game set?" → `find_render_states.py <game.exe>` (SetRenderState args: culling, blending, depth, fog)
- "How does the texture pipeline work?" → `find_texture_ops.py <game.exe>` (SetTexture stages, TSS ops, sampler filter/address modes)
- "Which transform types are used?" → `find_transforms.py <game.exe>` (SetTransform: World, View, Projection, Texture)
- "What surface formats does the game create?" → `find_surface_formats.py <game.exe>` (CreateTexture/RT/DS format extraction)
- "Does the game use state blocks?" → `find_stateblocks.py <game.exe>` (state block creation/recording/apply)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ekozmaster/Vibe-Reverse-Engineering](https://github.com/Ekozmaster/Vibe-Reverse-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
