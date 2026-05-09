---
trigger: always_on
description: **Name:** Shellcode IDE
---

# Shellcode IDE — agent.md

## Project overview

**Name:** Shellcode IDE

**Purpose:** A Binary Ninja plugin with a Qt GUI that helps users compose, analyze, optimize, and export shellcode across all architectures and platforms Binary Ninja supports. It combines Binary Ninja's binary/assembly APIs with a user-friendly GUI for rapid iteration and safe validation of shellcode suitable for exploitation and CTF use.

**Audience:** Reverse engineers, CTF players, exploit developers, and security researchers.

---

## Goals and scope

1. Provide two-way conversion between raw shellcode (hex / bytes) and assembly text.
2. Enable editing assembly and producing shellcode for any architecture supported by Binary Ninja.
3. Support multiple output formats for generated shellcode (inline asm, hex string, C array, Python bytes literal, Zig `[]u8`).
4. Show shellcode metadata (byte length, null-byte count, architecture, endianness).
5. Provide configurable bad-pattern detection (e.g. `\x00`, `\x0a`, `\xff`, sequences) and present results live.
6. Provide automatic peephole optimizations (user-toggleable) including the example transforms:

    - replace `push 0` → `xor reg, reg; push reg` (or `push 0x0` -> optimized sequence depending on arch)
    - replace `mov rax, 60` → `mov al, 60` when safe

7. Validate shellcode against constraints:

    - no variable references (no symbolic/local variables in assembly)
    - no direct absolute memory addresses or relocations
    - no null bytes unless allowed by user

8. Expose the plugin as a Binary Ninja toolbar/menu item and a floating Qt window with keyboard shortcuts.

---

## High-level architecture

1. **Frontend (Qt GUI):** Built with PySide2 or PyQt5 (choose one; PySide2 recommended because Binary Ninja typically bundles PySide2). GUI provides:

    - Input panel: paste raw hex or type assembly.
    - Output panel: shows generated shellcode in chosen format with copy buttons.
    - Live stats bar: architecture, length, null-count, bad-pattern count.
    - Bad-patterns config dialog: add/remove patterns (regex or hex bytes).
    - Optimization toggles and a "Run optimizer" button.
    - Validation / lint tab: shows violations with clickable items to jump to assembly line.
    - History / snippets manager: save named snippets per architecture.

2. **Backend (Binary Ninja plugin logic):** Uses Binary Ninja Python API for assemble/disassemble/architecture/platform discovery and for integration into the BN UI. Responsibilities:

    - Parse input (hex → bytes, assembly → tokens).
    - Call `Architecture.assemble()` for requested architecture and platform to get bytes from assembly.
    - Disassemble bytes into assembly text using Binary Ninja's `Architecture.get_instruction_text` or `BinaryView.get_disassembly` utilities.
    - Check produced bytes and assembly for rules (no addresses, no variables, no nulls).
    - Perform peephole optimizations on assembly text using pattern-match and rewrite passes (architecture-aware).

3. **Exporter module:** Formats bytes into: inline (escaped `\x..`), raw hex, C-style `unsigned char shellcode[] = {0x..}`, python-style `b"\x.."` or `bytes([...])`, and Zig-style `[]u8{0x..}`. Provide copy-to-clipboard and a save-to-file feature.

---

## Binary Ninja integration details

- **Plugin registration:** register plugin as a `UIAction` and a `DockWidget` / `Window` so it can open independently of a BinaryView. Provide menu entry under `Tools -> Shellcode IDE` and a toolbar icon.

- **Architecture / Platform selection:** Query `binaryninja.Architecture` and `binaryninja.Platform` registry: list all registered architectures and platforms. Default to the active BinaryView architecture/platform but allow the user to override.

- **Assemble flow:**

    1. User selects architecture+platform.
    2. `Architecture.assemble(asm_text, addr=0)` → returns bytes or raises `AssemblyError` with diagnostics.
    3. Show bytes, length, and formatted outputs.

- **Disassemble flow (hex -> asm):**

    1. Parse hex input into `bytes`.
    2. Create a temporary `BinaryView` backed by the bytes (in-memory) or use `Architecture.get_instruction_text` repeatedly with a cursor to decode instructions until the bytes are consumed.
    3. Render the disassembly in the output panel.

- **Error handling:** Show assembly errors with line/column if available. For disassembly, show where decoding failed.

---

## GUI details (UX)

### Main window layout

- **Top toolbar:** New, Open, Save, Copy, Assemble, Disassemble, Optimize (toggle), Validate.
- **Left pane (Input):** Tabs: "Hex/Bytes" and "Assembly". Each tab supports syntax highlighting (assembly using BN language or a simple highlighter). Paste detectors will auto-detect hex or base64.
- **Right pane (Output & Reports):** Tabs: "Disassembly / Assembly Output", "Formats", "Validation", "History".
- **Bottom status bar:** Selected architecture / platform, shellcode length, null byte count, bad pattern count, optimization status.

### Format output buttons

Each format output block includes: formatted text area, a Copy button, Save button, and a small preview showing the first N bytes.

### Validation tab


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CX330Blake/Shellcode-IDE](https://github.com/CX330Blake/Shellcode-IDE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
