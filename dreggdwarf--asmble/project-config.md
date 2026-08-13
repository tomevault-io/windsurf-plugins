---
trigger: always_on
description: Interactive step-by-step x86-64 assembly debugger for learning.
---

# ASMBLE — x86-64 Pedagogical Debugger

## Overview

Interactive step-by-step x86-64 assembly debugger for learning.
Paste a snippet, step through it, see registers/flags/stack change in real-time.
Frontend + backend fully functional in live mode.
GitHub: https://github.com/DrEggdwarf/ASMBLE

## Current State

**Frontend**: Complete (React 19 + TypeScript 5.6 + Vite 6)
- Editor with syntax highlighting (12 token types), linter, autocomplete, code folding, jump arrows, undo/redo
- Registers with clickable sub-registers (rax → eax/ax/ah/al), r8-r15 collapsible, diff mode (old→new + signed delta)
- 7 flags (ZF, CF, SF, OF, PF, AF, DF) with active/inactive state + flag hints
- Stack view with RSP/RBP markers, watchpoints (live)
- Right panel: collapsible, stacked stack+memory sections (no tabs)
- Console GDB: slide-in drawer (380px, live mode)
- Expression evaluator: floating popover (local + GDB)
- Build & Run (execute) + Step-by-step debug + Continue + Step Over/Out/Back
- Keyboard shortcuts: F5 Run/Continue, F10 StepOver, F11 StepInto, F9 Breakpoint, Shift variants
- Auto-step with configurable speed (100ms–2000ms)
- Terminal: program stdout/stderr, errors, connection status, interactive stdin input
- Terminal modes: docked drawer (resizable) or floating popover (detachable)
- Lexicon: ~45 instructions + ~15 syscalls with full-text search
- Convention SysV AMD64 + 9 addressing modes
- Display modes: hex/dec/bin for registers and stack, multi-format tooltips on hover
- Code history (localStorage, 10 recent programs)
- Snippet templates drawer (6 templates: Hello World, Boucle, Fonction, Stack Frame, Conditions, Tableau)
- Live/mock toggle via `VITE_LIVE_MODE` env var
- Connection indicator dot in header
- Security panel: checksec badges (RELRO, NX, PIE, Canary), vmmap table, GOT table, exploit tools (cyclic patterns, ROP gadgets, telescope, search)
- Theme: dark/light toggle with prefers-color-scheme auto-detection, 90+ CSS custom properties
- Responsive: @media breakpoints for 1200px (compact controls) and 768px (vertical stack layout)
- FAB: Floating Action Button with 5 quick actions (run, ref, theme, terminal, GDB console)

**Backend**: Fully operational (Python 3.12+ / FastAPI / pygdbmi)
- FastAPI WebSocket endpoint (`/api/ws`) — 27 message types
- pygdbmi → StepSnapshot bridge (GDB/MI3 protocol)
- PTY-based inferior I/O for interactive stdin support
- Step ~170ms, reset ~1s, build & run ~1.2s
- Off-by-one fix: highlight matches instruction that caused changes
- Program exit detection + auto-reset on step after exit
- Breakpoints (conditional), watchpoints, reverse step
- Annotations pédagogiques FR auto-générées (14 dynamic + ~25 static)
- Multi-assembler: NASM, GAS, FASM, YASM
- Sandbox: nsjail (mount/network/IPC namespaces) + rlimit fallback
- Session management: max 5 sessions, auto-cleanup idle
- Pydantic v2 models for all data structures
- Security analysis: checksec (pyelftools), vmmap (/proc/pid/maps), GOT entries
- pwndbg native tools: cyclic, rop, telescope, search, checksec via GDB bridge
- Custom exploit_tools.py kept as fallback when pwndbg unavailable
- Auto-checksec after assembly

## Files

```
asmble/
├── main.tsx                # Vite dev entry point
├── index.ts                # mount/unmount factory for lab integration
├── index.html              # HTML shell
├── vite.config.ts          # Vite config + proxy (/api → :8000)
├── Dockerfile              # Multi-stage build (node → ubuntu)
├── docker-compose.yml      # Orchestration développement
├── src/
│   ├── App.tsx             # AsmDebugger component (~800 lines, all state + UI)
│   ├── components/
│   │   ├── RegCard.tsx     # RegCard + RegExtRow (register display)
│   │   ├── GuidedTour.tsx  # Onboarding tour (7-step spotlight overlay)
│   │   ├── panels/
│   │   │   ├── StackPanel.tsx    # Stack view component
│   │   │   ├── MemoryPanel.tsx   # Memory sections component
│   │   │   ├── ConsolePanel.tsx  # GDB console drawer
│   │   │   ├── EvalPanel.tsx     # Expression evaluator
│   │   │   └── SecurityPanel.tsx # Checksec + vmmap + GOT
│   │   └── editor/
│   │       ├── AsmEditor.tsx    # IDE editor (textarea + highlight overlay)
│   │       ├── tokenizer.ts     # x86-64 tokenizer (12 token types)
│   │       ├── linter.ts        # Real-time linter (operand counts, labels)
│   │       ├── completions.ts   # Autocomplete data + instruction info
│   │       └── foldRegions.ts   # Code folding (sections, labels)
│   ├── data/
│   │   ├── index.ts        # Barrel re-export
│   │   ├── types.ts        # StepSnapshot, LexiconInstr, SubReg, etc.
│   │   ├── lexicon.ts      # ~45 instructions + ~15 syscalls
│   │   ├── patterns.ts     # 17 C→ASM patterns
│   │   ├── registers.ts    # Register lists + sub-register utilities
│   │   ├── convention.ts   # SysV AMD64 calling convention
│   │   ├── addressing.ts   # 9 addressing modes
│   │   └── memory.ts       # Mock .text section (fallback)
│   ├── hooks/
│   │   ├── useColResize.ts     # 3-column drag resize
│   │   ├── useTermResize.ts    # Terminal panel drag resize
│   │   └── useGdbSession.ts    # WebSocket GDB client (state machine)
│   └── styles/
│       ├── index.css       # CSS entry point
│       └── asmble.css      # All styles, prefixed `asm-`
├── backend/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrEggdwarf/ASMBLE](https://github.com/DrEggdwarf/ASMBLE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
