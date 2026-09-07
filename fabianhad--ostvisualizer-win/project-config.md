---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Project Snapshot

OST Visualizer is a Windows desktop app for reading On-Screen Takeoff `.mdb` files. The free edition is a read-only 2D viewer; licensed builds unlock 3D, editing, import, and export.

- Repository root is the desktop client root.
- App entry point: `Visualizer.py` -> `ost_visualizer/main.py`.
- MCP helper entry point: `McpServer.py` -> `ost_visualizer/mcp_server/main.py`.
- App package: `ost_visualizer`.
- Server-side license code lives outside this client checkout.

## Common Commands

Run PowerShell scripts from the repository root.

```powershell
.\scripts\setup.ps1
.\scripts\setup-cpp.ps1
.\scripts\run.ps1
.\scripts\build.ps1
.\scripts\build-visualizer.ps1
.\scripts\build-mcp.ps1
.\build-msi.ps1
```

Useful validation:

```powershell
.\venv\Scripts\python.exe -m unittest discover -s tests -p "test_mcp*.py" -v
.\venv\Scripts\python.exe -m unittest discover -s tests -v
python tools\check_architecture.py
python tools\check_architecture.py --changed-only
python -m unittest tests.test_plan_view_snap_helper
vulture ost_visualizer
```

C++ extensions require Visual Studio 2022, CMake, and Qt 6.10.2 at `C:\Qt\6.10.2\msvc2022_64`. If native snap code changes, rebuild `ost_snap` from the configured CMake build directory, usually `cpp_extensions/build`.
PySide6 6.10.2 and the downloaded PDFium/QPDF archives are exact-version inputs;
the other Python requirements intentionally remain unpinned. Native archive URLs
and SHA-256 digests must be updated together from the authoritative upstream
release when intentionally upgrading a dependency. Native dependency extraction
uses a sibling staging directory and commits the final directory only after its
installed version validates; an existing directory must pass the same exact
version check before reuse.

## Architecture Guardrails

The app follows a clean/hexagonal shape:

```text
presentation -> application -> domain
                         ^
infrastructure ----------|
```

- `domain/` is pure business logic. It must not import application, infrastructure, or presentation.
- `application/` orchestrates use cases, DTOs, services, events, and interfaces. It must not import infrastructure or presentation.
- `infrastructure/` implements ports and may import approved application interfaces/DTOs/events. It must not import presentation except for architecture-checker exceptions in factory/extension integration files.
- `presentation/` is PySide6 UI. Prefer application DTOs for new data flowing to UI.
- `config/di_config.py` is the composition root and may wire all layers.
- `presentation/main_window.py` is the presentation composition root. Other presentation files should receive dependencies by constructor injection.
- `ServiceContainer.get_by_interface()` is expected only for runtime discovery such as `IShutdownAware`.
- Declaration-only `Protocol` and `@abstractmethod` methods use `...`. Production
  exception and marker classes use a concise, meaningful docstring as their sole
  body. Intentional concrete no-op hooks use `pass`; an explicit `return None`
  is reserved for contracts where absence is the result. Test doubles follow the
  same no-op-versus-absence distinction, while empty test shell classes use
  `pass`.

Threading and events:

- UI work must stay on the main Qt thread.
- Worker threads must marshal back through existing Qt bridges before UI updates or EventBus publication.
- Do not publish EventBus events from worker threads.
- Subscribe in constructors/init paths and unsubscribe in `cleanup()`.
- Tentative application shutdown pauses shutdown-owned startup/UI continuations
  and prepares deferred persistence without destroying it. An aborted close
  resumes persistence and replays each valid continuation once in causal order;
  only terminal close discards callbacks and releases persistence dependencies.
  Cleanup owners retain failed EventBus unsubscriptions so a later cleanup can
  retry them instead of reporting a stale subscription as released. An
  in-flight EventBus publication skips subscriptions removed before their turn,
  including remove-and-readd cycles. Modal cleanup must tolerate Qt destroying
  a child with its closing parent before the nested event loop returns. Dialog-
  owned asynchronous completions and unparented `QTimer.singleShot` callbacks
  must validate the underlying Qt owner before touching widgets or projecting
  returned state.
- Native file dialogs run nested event loops. Capture the complete database/bid/
  page target before opening them, then reject the continuation if that context
  changed, the authoritative hierarchy entity was replaced even with the same
  UID, or the owning window was cleaned up or destroyed while the dialog was
  open.
- Context menus and internal Qt drags also run nested event loops. Bind delayed
  actions to the exact model or surface revision, selection, and authoritative
  object identities; cancel active tree drags and inline editors before deleting
  their items during a model reset. Menus opened by an embedded native render
  surface use its real top-level widget as their transient owner; a child
  `QWidgetWindow` is not a valid top-level owner. Clipboard payloads remain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fabianhad/OSTVisualizer-Win](https://github.com/Fabianhad/OSTVisualizer-Win) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
