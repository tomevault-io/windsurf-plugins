---
trigger: always_on
description: | Task | Model to use |
---

# VisualGasic — Copilot Instructions

## Model selection (check before starting)

| Task | Model to use |
|---|---|
| C++ parser / runtime bugs (M1) | **Claude Opus 4.8** — max mode |
| GDScript plugin architecture (M4 UI Forms) | **Claude Opus 4.8** — max mode |
| AST walker / algorithm design (M6) | **Claude Opus 4.8** — standard mode |
| Extending existing GDScript files (M3) | **Claude Sonnet 4.6** — standard mode |
| Examples audit / mechanical fixes (M2) | **Claude Sonnet 4.6** — standard mode |
| Narcea prompt engineering (M5) | **Claude Sonnet 4.6** — standard mode |
| README / docs / issue bodies | **Claude Sonnet 4.6** — standard mode |
| Quick questions / one-liners | **Claude Haiku** |

Switch the model in the Copilot chat dropdown before pasting a large prompt.

---

## Project context

- **Language:** VB6-style BASIC (`VisualGasic` / `.vg` files), tokenizer/parser/AST/VM in C++
- **Runtime:** Godot 4.6.1 plugin via GDExtension (`.gdextension`)
- **Primary IDE:** Godot's native Script editor — VG extends it with autocomplete, Code Navigator, dot-completion
- **VG custom IDE shell / legacy Form Designer:** **experimental Alpha** — **not** on the v6.0 EOY critical path; mothballed unless `vg/enable_experimental_plugins = true`. User-facing status: `docs/manual/VG_IDE_ALPHA.md` (standalone Form Designer repo extraction is **planned**, not shipped yet).
- **Godot IDE integration (active scope):** Toolbox panel, Properties window, Immediate window, Narcea Vibe Code, Code Navigator, autocomplete, dot-completion — these are docked inside Godot's editor and are **in scope for all milestones**
- **Focus until v6.0:** VG Script language quality + Godot IDE integration. Do NOT expand the VG standalone IDE shell.
- **AI providers:** `addons/visual_gasic/vg_ai_providers.gd` — Ollama, OpenAI, Claude, Gemini
- **Canonical Claude models (Anthropic direct API):** `claude-opus-4-8`, `claude-sonnet-4-6`, `claude-haiku-4-5` (verified 2026-06-29)
- **GitHub Copilot model dropdown:** also exposes `claude-opus-4-6`, `claude-opus-4-7` — these are Copilot-specific variants not on the direct API

## Milestone schedule

| Milestone | Due | Status | Focus |
|---|---|---|---|
| M1 | Jul 31 | ✅ DONE (Jun 29) | 4 critical bugs — all fixed |
| M2 | Aug 15 | ✅ DONE (Jun 30) | 44/44 corpus examples passing |
| M3 | Aug 31 | ✅ DONE (Jul 1) | Code Navigator upgrade (#7) |
| M4 | Sep 30 | ✅ DONE (Jul 1) | UI Forms experimental (#8–12) |
| M5 | Oct 15 | 🔄 NEXT | Narcea Vibe Code (#13) |
| M6 | Oct 31 | ✅ **Partial** | Causal Chain text-mode (#14) — C++ `vg_analyze_causal_graph`, Code Navigator button, Context Rail preview, 8 headless fixtures |
| M7 | Nov 15 | 🟡 **Close-out** | Python Library Integration — C2 typed msgpack **default-on** for new projects; large-array binary lane shipped (`test_py_large_array.vg`); Windows CI M7 smoke; macOS e2e still deferred |
| M8 | Nov 22 | — | Language parity (Try/Catch/Lambda/`?.` tests), `Let` keyword, C++ interop |
| M9 | Nov 28 | — | Asset Library submission, installer smoke test, 50+ corpus, docs |
| Stable v6.0 | Jan 1 2027 | — | — |

## Code conventions

- GDScript: `snake_case` for functions and variables, `PascalCase` for classes
- VG language: VB6-style — `Sub`, `Function`, `End Sub`, `Dim x As String`, `If ... Then ... End If`
- **Unicode text:** VG controls are Godot controls (Unicode-native). The VB6 pitfall where standard TextBox/Label show `???` for non-ANSI text does **not** apply. `StrConv(..., vbUnicode/vbFromUnicode)` are no-ops. Save `.vg` as UTF-8; use `vbCrLf`/`vbTab` for control chars in source; choose fonts with needed glyphs for CJK/emoji.
- C++: follow existing style in `src/` — no trailing whitespace, tabs not spaces
- C++ memory: strict manual management within GDExtension bounds; verify zero leaks on node destruction
- C++ errors: no generic panics — use clean, actionable error messages with exact line numbers
- Do not add docstrings, comments, or type annotations to code you didn't change
- Do not refactor or rename symbols outside the scope of the current task

## Documentation tone

- Technical, precise, highly instructional — no marketing language

## Known open bugs

- **AST evaluator missing Godot type-constructor dispatch (OPEN, found Jul 30 2026)** — the bytecode compiler recognizes `Vector2i`/`Rect2i`/`Color`/etc. as type constructors (`_godot_type_ctors[]` in `src/visual_gasic_compiler.cpp` ~line 6953, emits `OP_NEW_OBJECT`), but the AST tree-walk evaluator has no equivalent dispatch — calling e.g. `Rect2i(...)` in a Sub that's fallen back to AST interpretation (see ByRef fallback landmine in `/memories/repo/v6.0_blockers.md` §0) throws `Sub or Function not defined`. Found while fixing the C64 emulator's `BlitImage` bug below. Not yet fixed.
- **C64/GBA Emulator `BlitImage` no-op — FIXED Jul 30 2026** — both `samples/demos/C64_Emulator/c64_main.vg` and `samples/demos/GBA_Emulator/gba_main.vg` called `BlitImage` with `Rect2`/`Vector2` (float) instead of the required `Rect2i`/`Vector2i`/plain-int form, so the VIC/GPU framebuffer was never actually copied to the display texture (silent no-op, zero pixels copied, every frame, forever). Fixed by switching to `BlitImage`'s plain-integer 8-arg overload.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xgreenrx-star/VisualGasic](https://github.com/xgreenrx-star/VisualGasic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
