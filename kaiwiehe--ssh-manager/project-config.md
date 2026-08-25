---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
# App starten
python ssh_manager.py

# Tests ausführen
python -m pytest tests/

# Einzelnen Test ausführen
python -m pytest tests/test_logic.py::test_build_wt_command_single_session

# Syntax prüfen
python -m py_compile ssh_manager.py ssh_manager_app/*.py

# Syntax prüfen unter PowerShell (Wildcard wird sonst nicht expandiert)
$files = @('ssh_manager.py') + (Get-ChildItem -Path ssh_manager_app -Filter *.py | ForEach-Object { $_.FullName }); python -m py_compile @files

# Portable Windows-EXE bauen (auf Windows)
powershell -ExecutionPolicy Bypass -File scripts\build_windows.ps1
```

App-Laufzeit: keine externen Abhängigkeiten – nur Python-Standardbibliothek (`tkinter`, `winreg`, `subprocess`, `pathlib`, `socket`, `threading`, `xml.etree.ElementTree`).  
Aus tkinter genutzte Module: `tk`, `ttk`, `messagebox`, `simpledialog`, `filedialog`.

## Architektur

Die App war ursprünglich fast komplett in `ssh_manager.py`, wurde inzwischen aber vorsichtig in Module aufgeteilt. `ssh_manager.py` bleibt eine dünne Bootstrap-Shell mit `SSHManagerApp`, Fenster-Icon und zentraler Verdrahtung.

### Datenfluss

```
Registry (WinSCP)      ──┐
~/.ssh/config          ──┼──► storage/core ──► actions_ui.build_visible_sessions(app) ──► SessionTree.populate()
FileZilla sitemanager  ──┤
app_sessions.json      ──┤
notes.json / settings  ──┘
```

`SSHManagerApp` lädt beim Start WinSCP, SSH Config, FileZilla und eigene App-Sessions, filtert sie anhand der Anzeige-Einstellungen und gibt die sichtbaren Sessions an `SessionTree` weiter. Der "SSH Config"-Ordner wird immer oben sortiert, FileZilla landet unter `FileZilla Config`.

### Schichten / Module

**`ssh_manager.py`**
- enthält primär `SSHManagerApp` plus `set_window_icon()`
- zentrale Verdrahtung zwischen UI, Tree, Dialogen, Storage und Terminal-/Registry-Logik
- muss weiter direkt per `python ssh_manager.py` startbar bleiben

**`ssh_manager_app/constants.py`**
- App-Konstanten, Dateipfade, Standardordnernamen

**`ssh_manager_app/models.py`**
- `Session`
- Settings-Dataclasses (`ToolbarSettings`, `WindowsTerminalSettings`, `SourceVisibilitySettings`, `ImportSettings`, `AppearanceSettings`, `AppSettings`)
- `color_tag()`
- `default_settings()` / `settings_to_dict()`

**`ssh_manager_app/storage.py`**
- Laden/Speichern von Settings, UI-State, Notes, App-Sessions
- Loader für FileZilla und SSH Config

**`ssh_manager_app/core.py`**
- `parse_session_key()`
- `build_wt_command()`
- `build_jump_wt_command()`
- `build_remote_command_wt_command()`
- `build_remote_script_wt_command()` für lokale/remote Skript-Runbooks per SSH
- `build_ssh_copy_id_command()`
- `build_ssh_remove_key_command()`
- `build_ssh_tunnel_command()`
- `check_host_reachable()`
- `_create_checkbox_images()`
- `TerminalLauncher`
- `RegistryReader`

**`ssh_manager_app/tree.py`**
- `SessionTree(ttk.Frame)` – Haupt-Treeview mit Checkboxen, Farben, Kontextmenüs und Notes-Tooltip/Info-Anbindung

**`ssh_manager_app/dialogs_base.py`**
- `UserDialog`
- `_build_quickselect_buttons()`
- `resolve_user_dialog_defaults()`

**`ssh_manager_app/dialogs_move_folder.py`**
- `MoveFolderDialog`

**`ssh_manager_app/dialogs_remote.py`**
- `JumpHostDialog`
- `RemoteCommandDialog` für Remote-Befehle und Skript-Runbooks
- `RemoteFavoriteEditDialog` für Runbook-Favoriten mit Name, Notiz und Pin-Status
- `RemoteCommandConfirmDialog`
- `SshCopyIdDialog`
- `SshRemoveKeyDialog`
- `SshTunnelDialog`

**`ssh_manager_app/dialogs_session_edit.py`**
- `SessionEditDialog`

**`ssh_manager_app/dialogs_settings_misc.py`**
- `SettingsView`
- `SshConfigInspectDialog`

**`ssh_manager_app/dialogs_toast.py`**
- `ToastNotification`

**`ssh_manager_app/dialogs.py`**
- nur Kompatibilitäts-Aggregator für bestehende Imports

**`ssh_manager_app/ui.py`**
- `configure_app_styles()` / Theme-Styling inkl. Dark Mode
- `refresh_checkbox_images()`
- `build_main_ui()`

**`ssh_manager_app/themes.py`**
- Theme-Paletten für Light/Dark/Midnight

**`ssh_manager_app/version.py`**
- `APP_NAME` / `APP_VERSION` für App-Name und Packaging

**Packaging**
- `assets/ssh-manager.ico` – App-/EXE-Icon
- `assets/ssh-manager.png` – 256px Runtime-Icon für `iconphoto()`
- `assets/SSH-Logo.svg` – maßgebliche Icon-Quelle, aktuell ein simples schwarzes `>_`-Prompt-Logo
- `assets/SSH-Logo.png` – PNG-Fallback der Icon-Quelle
- `scripts/generate_icon.py` – generiert PNG/ICO-Frames; für das simple Prompt-Logo gibt es einen Pillow-Renderpfad ohne CairoSVG
- `ssh_manager.spec` – PyInstaller-Spec für portable Einzel-EXE
- `scripts/build_windows.ps1` – Windows-Build-Script
- `packaging/ssh_manager_version_info.txt` – Windows-Version-Metadaten

**Datenquellen:**
- `RegistryReader` – liest WinSCP-Sessions aus `HKCU\Software\Martin Prikryl\WinSCP 2\Sessions`
- `load_ssh_config_sessions()` – parst `~/.ssh/config` manuell (kein Parser-Import)
- `load_filezilla_config_sessions()` – liest FileZilla-Sites aus `%APPDATA%\FileZilla\sitemanager.xml`
- `app_sessions.json` – eigene Sessions (source=`app`) und SSH-Alias-Kopien (source=`ssh_alias`)
- `notes.json` – Session-Notizen für alle Quellen, nur app-intern gespeichert


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaiWiehe/ssh_manager](https://github.com/KaiWiehe/ssh_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
