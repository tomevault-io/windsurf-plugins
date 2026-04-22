---
trigger: always_on
description: This document provides implementation guidance for Claude Code (and other coding agents) working in this repository.
---

# CLAUDE.md

This document provides implementation guidance for Claude Code (and other coding agents) working in this repository.

## 1. Project Overview

PyQtInspect is a runtime inspector for Qt Widgets applications (PyQt5/PyQt6/PySide2/PySide6). Its architecture is split into two cooperating runtimes:

- **Inspector Server (GUI)**: a desktop GUI (`pqi-server`) that developers operate.
- **Debuggee Client (Injected Runtime)**: code running inside the target Python Qt process (`PyQtInspect.pqi` + Qt monkey patches).

At a high level:

1. The client connects to the server over TCP.
2. The client patches Qt widgets/events, tracks widget metadata, and reacts to server commands.
3. The server sends user-driven commands (start inspect, request widget info, execute code, etc.).
4. The client sends structured responses/events back (selected widget info, tree, props, execution result, etc.).

---

## 2. Runtime Architecture and Responsibilities

### 2.1 Server side (GUI)

Primary modules:

- `PyQtInspect/pqi_server_gui.py`: main window and UI orchestration.
- `PyQtInspect/pqi_gui/workers/pqy_worker.py`: TCP listener accepting clients and creating per-client dispatchers.
- `PyQtInspect/pqi_gui/workers/dispatcher.py`: per-connection bridge with reader/writer threads.
- `PyQtInspect/pqi_gui/settings/controller.py`: settings persistence via `QSettings` + INI format.
- `PyQtInspect/pqi_gui/windows/settings_window.py`: settings dialog with GroupBox-based sections.

Key behavior:

- Server can host **multiple client connections** in detached mode.
- Each accepted socket creates one `Dispatcher`, identified by dispatcher ID.
- UI actions route to a target dispatcher (or all dispatchers) via `PQYWorker` methods.
- Incoming network messages are decoded and dispatched by command ID in `PQIWindow.onWidgetInfoRecv`.

### 2.2 Client side (debuggee runtime)

Primary modules:

- `PyQtInspect/pqi.py`: core debugger/client runtime (`PyDB`) and command handlers.
- `PyQtInspect/_pqi_bundle/pqi_monkey_qt_helpers.py`: Qt patching logic and event interception.
- `PyQtInspect/_pqi_bundle/pqi_comm.py`: network protocol, reader/writer threads, command factory.

Key behavior:

- Client connects to server (`start_client`) and initializes reader/writer threads.
- After successful connect, it sends `CMD_PROCESS_CREATED`.
- After Qt monkey patch success, it sends `CMD_QT_PATCH_SUCCESS` with PID.
- Client stores widget references by `id(widget)` and serves info on demand.
- On server commands, client performs operations like enable inspect, highlight widget, select widget, execute code, return properties/tree/children.

### 2.3 Mode differences

- **Detached mode**: server started explicitly; clients connect to configured host/port.
- **Direct mode**: debuggee launches a colocated server process and exits together with it.

### 2.4 Settings system

Settings are persisted in `settings.ini` using Qt's `QSettings` (INI format, UTF-8).

- **`SettingsController`** (`pqi_gui/settings/controller.py`): singleton that manages all settings. Uses the `SettingField` descriptor pattern — each setting is declared as a class-level descriptor with key, type, and default value, providing clean property-style access.
- **Settings keys** are organized in nested classes inside `SettingsController.SettingsKeys` (e.g., `IDE.Type`, `Highlight.Color`).
- **Settings window** (`pqi_gui/windows/settings_window.py`): a `QDialog` containing `QGroupBox`-based sections. Each section is a standalone class (e.g., `IDESettingsGroupBox`, `HighlightSettingsGroupBox`) with its own getter/setter methods and optional validation via `isValid()`. To add a new settings section:
  1. Create a new `QGroupBox` subclass with getter/setter methods.
  2. Add it to `SettingWindow.__init__` layout (before the stretch).
  3. Wire it in `loadSettings()` and `saveSettings()`.

#### Server-to-client settings transmission

Some settings need to reach the client (debuggee) process. Rather than adding new command IDs, the preferred pattern is to **piggyback on `CMD_ENABLE_INSPECT`'s extra data dict**:

1. Server builds the extra dict (see `PQIWindow._buildInspectExtraData()`), which is sent as JSON payload with `CMD_ENABLE_INSPECT`.
2. Client stores it in `PyDB._inspect_extra_data` and exposes individual settings as properties (e.g., `mock_left_button_down`, `highlight_color`).
3. Client-side code (e.g., monkey patches) reads these properties from `get_global_debugger()`.

This approach is additive and backward compatible — older clients ignore unknown keys and fall back to defaults.

---

## 3. Client/Server Interaction Contract

### 3.1 Transport and framing

- Default protocol is **quoted-line** (one command per line):
  - `cmd_id\tseq\tpayload\n`
- Payload is URL-quoted in `NetCommand`; decoded with `unquote` on read.
- JSON payloads use compact `json.dumps(..., separators=(',', ':'))`.

Implementation anchors:

- `NetCommand` serialization + encoding in `pqi_comm.py`.
- `ReaderThread` parses by newline, then tab-splitting into `cmd_id`, `seq`, `text`.

### 3.2 Command IDs and message semantics

Message IDs are defined in `PyQtInspect/_pqi_bundle/pqi_comm_constants.py`.

Common flows:

- Lifecycle:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JezaChen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
