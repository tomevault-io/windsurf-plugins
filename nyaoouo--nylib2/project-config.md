---
trigger: always_on
description: Guide for coding agents (Claude, Copilot, Gemini, etc.) working in this
---

# AGENTS.md

Guide for coding agents (Claude, Copilot, Gemini, etc.) working in this
repository. Human-facing documentation lives in `readme.md`.

This file is intentionally self-contained: read it before touching code,
and read `.agent-memory/` for deeper conventions if it exists locally.

---

## 1. What `nylib` is

A Windows-only Python toolkit for game / process inspection, code
injection, function hooking, and on-screen Dear ImGui overlays. Most
modules are thin, typed `ctypes` wrappers over Win32 plus a few
hand-rolled C extensions.

- **Platform:** Windows x64. Many APIs (`WriteProcessMemory`,
  `Detours`, `IsUserAnAdmin`, ...) make no sense elsewhere.
- **Python:** 3.11 - 3.14. The shipped pyimgui `.pyd` filenames encode
  the ABI tag, so consumers must build their own against their CPython.
- **Install:** `pip install .` from a checkout, or
  `pip install -e .` for editable use. See section 5.

---

## 2. Repository layout

```
nylib/                       importable package
  __init__.py                (empty re-export shell)
  __pyinstaller/             PyInstaller hooks (auto-discovered via entry point)
  ctype/                     remote-process aware ctypes accessors
  hook/                      Microsoft Detours wrapper (`Hook`, `create_hook`)
  imguiutils/                widgets & FA icon TTFs for the pyimgui frontend
    debug_view/              Dear ImGui debug view (disasm/hex/BP/tools/console)
  logging.py                 color console logging, multiline + std->file tee
  mono/                      Unity / Mono runtime inspector
  pattern.py                 IDA-style "AA BB ?? CC" scanner
  process/                   `Process` class - memory R/W, module enum, scan
  pyimgui/                   stub package - native `.pyd` built by the user
  structs/                   PDB / DWARF struct helpers
  tkinter_/                  small tkinter extras
  utils/                     pip bootstrap, eventloop, threading, web, ...
  vmware_wp/                 VMware Workstation Pro backend (own README.md)
  winapi/                    typed ctypes for kernel32 / ntdll / user32 / ...
  winutils/                  high-level Win32 helpers (see section 4)
    breakpoint/              x64 HW/SW breakpoint primitive (VEH + debugger)

scripts/                     dev-only tools (NOT installed; not in the wheel)
  breakpoint_demo/           manual smoke scripts for winutils.breakpoint
  dxtest/                    DirectX 9/10/11/12 injection smoke tests
  generate_bind/             pybind11 binding generator from C headers
  ida/                       IDA Pro loaders & sig workers
  pyimgui/  pyimgui2/        build scripts for the `pyimgui` C extension
  sig_thief/                 signature thief (PE Authenticode)
  test_inject/               sample injection target
  windows_driver_policy_manage/  PowerShell cross-cert driver-policy manager

pyproject.toml               package metadata + PyInstaller entry point
readme.md                    user-facing intro, feature map & examples
license.txt                  GPL v3
.agent-memory/               local-only notes for AI agents (git-ignored)
```

`scripts/` is **outside** the installable package on purpose. Anything
under `nylib/` ships in the wheel; anything under `scripts/` is a
development tool and is only available from a checkout.

---

## 3. Module reference

Treat this section as the public API map. Imports shown are the
canonical entry points.

### `nylib.process`

```python
from nylib.process import Process
```

- `Process.from_name(name)` / `Process.from_id(pid)` - construct from a
  process name (bytes or str) or PID.
- `Process.current` - the calling process.
- Memory: `alloc`, `free`, `read(addr, type)`, `write(addr, value)`,
  `virtual_query`, `virtual_protect`, `iter_memory_region`,
  `alloc_near`.
- Modules: `enum_ldr_data()`, `get_module(name)`, `base_address`.
- Scanning: cached `CachedRawMemoryPatternScanner` and
  `StaticPatternSearcher` via `Process` helpers (see `pattern.py`).

### `nylib.winapi`

Typed `ctypes` wrappers around the Win32 surface used internally:
`kernel32`, `ntdll`, `user32`, `advapi32`, `shell32`, `msvcrt`, `ole32`,
`propsys`, `version`. Plus `utils.py` with helpers such as
`DEFAULT_ENCODING` and small `byref` wrappers. Prefer importing the symbol
you need directly:

```python
from nylib.winapi import OpenProcess, ReadProcessMemory, MEMORY_BASIC_INFORMATION
```

### `nylib.winutils`

High-level Win32 helpers; subpackages are loaded lazily.

| Module | Purpose |
| --- | --- |
| `winutils.process` | `enable_privilege()`, `run_admin()`, `iter_processes()`, `pid_by_executable()`, `create_suspend_process` |
| `winutils.breakpoint` | x64 HW/SW breakpoint primitive (`BreakPoint`, `BP_E`); VEH + debugger backends - see section 3 |
| `winutils.ensure_env` | locate / install Visual Studio + Windows SDK + LLVM toolchains |
| `winutils.msvc` | `load_vcvarsall(arch)` and friends |
| `winutils.llvm_pdb` | PDB symbol helpers via LLVM |
| `winutils.driver` | service-control (`SCManager`, `Service`) helpers for loading kernel drivers |
| `winutils.inline_hook` | hot-patch trampoline based on `keystone-engine` + `capstone` |
| `winutils.pe_unmap` | rebuild an in-memory PE back to its on-disk layout |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyaoouo/NyLib2](https://github.com/nyaoouo/NyLib2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
