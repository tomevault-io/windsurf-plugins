---
trigger: always_on
description: Keyhac2 is the unification of **Keyhac for Windows** (`../keyhac-win`, v1.83) and
---

# CLAUDE.md — Keyhac2

## What this project is

Keyhac2 is the unification of **Keyhac for Windows** (`../keyhac-win`, v1.83) and
**Keyhac for macOS** (`../keyhac-mac`, v1.68): a keyboard customization / macro tool
whose behavior is scripted by the user in Python (`~/.keyhac/config.py`). It has **one
shared Python codebase** for both OSes, with thin per-OS platform modules. All UI is
built on **PuiKit** (`../puikit`), the author's portable Python UI toolkit.

Documentation:

- End-user docs: [README.md](README.md), [doc/](doc/) —
  [installation](doc/installation.md), [configuration](doc/configuration.md),
  [API reference](doc/config-api.md) (generated), migration guides from
  [keyhac-mac](doc/migration-from-keyhac-mac.md) /
  [keyhac-win](doc/migration-from-keyhac-win.md).
- Developer docs: [doc/dev/](doc/dev/) — [overview](doc/dev/overview.md),
  [architecture](doc/dev/architecture.md),
  [platform-layer](doc/dev/platform-layer.md),
  [design-notes](doc/dev/design-notes.md), [puikit](doc/dev/puikit.md),
  [packaging](doc/dev/packaging.md), [testing](doc/dev/testing.md).
- Remaining tasks and open decisions live in the **GitHub issues**
  (`gh issue list`), not in the docs. One exception:
  [doc/dev/next-major.md](doc/dev/next-major.md) is the ledger of breaking API
  changes the additive-only policy defers to the next major release — record
  such ideas there, not as issues.

## Sibling repositories (read-only references)

| Repo | What it is | What to learn from it |
|---|---|---|
| `../keyhac-win` | Python 3.13 + thin C++ launcher. UI via `ckit`, input via `pyauto` (external C++ extension repos, **not present in this checkout**). | Synchronous `WH_KEYBOARD_LL` hook semantics, modifier-state engine, one-shot/multi-stroke logic, hook-recovery, clipboard history, list window UX, full feature set. |
| `../keyhac-mac` | Swift/SwiftUI menu-bar app embedding CPython 3.13 via a C++ bridge. | CGEventTap semantics (re-enable, event-source filtering, deferred-event reordering), AX focus paths, the **modern snake_case config API that Keyhac2 adopted as its baseline**, ThreadedAction model. |
| `../puikit` | Pure-Python UI toolkit, PyPI `puikit`. Backends: curses / macOS (PyObjC+AppKit) / Windows (ctypes+Direct2D) / web / memory. | The UI layer for Keyhac2. Its `CLAUDE.md` documents a strict additive API-compatibility policy — all Keyhac2-driven extensions must follow it. |

Feature reference is frozen at win 1.83 / mac 1.68; changes upstream after that are
reviewed one-off.

## Key decisions (rationale in doc/dev/)

1. **Languages**: Python 3.14 for everything at runtime. Platform bindings via
   **ctypes** (Windows) and **PyObjC** (macOS) — no custom compiled extension modules.
   The only non-Python code is a tiny PEP 587 embedding **launcher** per OS, needed
   for packaging and a stable app identity (macOS Accessibility permission is granted
   per bundle). See [doc/dev/packaging.md](doc/dev/packaging.md).
2. **Key hook**: both OS hooks are *synchronous consume-decisions* running on the main
   thread; the differences (tap re-enable + injected/real event reordering on macOS;
   silent-unhook recovery on Windows) are encapsulated behind one `InputHook`
   interface. See [doc/dev/platform-layer.md](doc/dev/platform-layer.md).
3. **Config API**: keyhac-mac's snake_case API is the base, extended with portable
   focus conditions (`app=`, `title=`) and the keyhac-win features it lacked. One
   `config.py` runs on both OSes; keyhac-win configs require migration.
4. **PuiKit is extended additively** per its compatibility policy; everything Keyhac2
   needs is in the PyPI release `pyproject.toml` pins. See
   [doc/dev/puikit.md](doc/dev/puikit.md).
5. **Single process, main-thread rule**: the native event loop on the main thread
   services the hook *and* all PuiKit windows. Slow work goes to `ThreadedAction`;
   results come back via `call_on_main_thread`.

## Source layout

```
keyhac/
  core/        # OS-independent: keymap engine, key expressions, input context,
               # actions, clipboard history, replay, config loader, settings, logging
  actions.py   # action objects needing platform/UI wiring (MoveWindow, choosers, ...)
  platform/    # base.py interface definitions + fake.py test doubles
    win/       # ctypes: WH_KEYBOARD_LL/WH_MOUSE_LL, SendInput, UIA, Win32 windows
    mac/       # PyObjC: CGEventTap, CGEventPost, AXUIElement, NSWorkspace, NSPasteboard
  ui/          # PuiKit-based: console, chooser, balloon, tray, runtime (backend holder)
  main.py      # bootstrap: loop setup, hook install, config load
  _config.py   # the config.py template copied on first run
windows_app/   # Keyhac.exe launcher + bundle build (build.ps1)
macos_app/     # Keyhac.app launcher + bundle build (build.sh, create_dmg.sh)
art/           # hand-maintained SVG icon sources (rendered by tools/make_icons.py)
tools/         # icon pipeline, release scripts, hook_echo diagnostic
```

## Conventions

- Public API is snake_case (keyhac-mac style). No new camelCase API.
- `keyhac/core/` must not import OS modules (`ctypes.windll`, `AppKit`, `Quartz`,
  Win32 constants). All OS access goes through `keyhac/platform/` interfaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crftwr/keyhac](https://github.com/crftwr/keyhac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
