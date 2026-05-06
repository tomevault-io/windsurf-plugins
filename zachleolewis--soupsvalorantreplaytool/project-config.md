---
trigger: always_on
description: This is a Python desktop application that enables VALORANT players to view any downloaded replay by temporarily swapping replay files during the loading process. The tool uses VALORANT's local APIs for authentication and metadata, with a modern dark-themed GUI.
---

````instructions# VALORANT Replay Tool - AI Coding Guidelines

# VALORANT Replay Tool - AI Coding Guidelines

## Project Overview

## Project OverviewThis is a Python desktop application that enables VALORANT players to view any downloaded replay by temporarily swapping replay files during the loading process. The tool uses VALORANT's local APIs for authentication and metadata, with a modern dark-themed GUI.

This is a Python desktop application that enables VALORANT players to view any downloaded replay by temporarily swapping replay files during the loading process. The tool uses VALORANT's local APIs for authentication and metadata, with a modern dark-themed GUI.

## Architecture

## Architecture

### Core Components

### Core Components- **`modern_replay_injector_ui.py`** - Main tkinter-based GUI with tabbed interface

- **`modern_replay_injector_ui.py`** - Main tkinter-based GUI with tabbed interface- **`replay_file_manager.py`** - File operations, backup/restore, and session monitoring

- **`replay_file_manager.py`** - File operations, backup/restore, and session monitoring- **`replay_metadata.py`** - VALORANT API integration for match data and metadata

- **`replay_metadata.py`** - VALORANT API integration for match data and metadata- **`region_config.py`** - Region-specific API endpoint management

- **`region_config.py`** - Region-specific API endpoint management- **`build_v1_1.py`** - PyInstaller build script for executable distribution

- **`build_v1_1.py`** - PyInstaller build script for executable distribution

### Key Design Patterns

### Key Design Patterns

#### 1. VALORANT Local API Integration

#### 1. VALORANT Local API Integration```python

```python# Always use lockfile authentication for local API access

# Always use lockfile authentication for local API accesslockfile_path = r"C:\Users\{username}\AppData\Local\Riot Games\Riot Client\Config\lockfile"

lockfile_path = r"C:\Users\{username}\AppData\Local\Riot Games\Riot Client\Config\lockfile"# Parse format: name:pid:port:password:protocol

# Parse format: name:pid:port:password:protocol# Use https://127.0.0.1:{port} with Basic auth

# Use https://127.0.0.1:{port} with Basic auth```

```

#### 2. Session State Monitoring

#### 2. Session State Monitoring```python

```python# Monitor VALORANT client state changes via session polling

# Monitor VALORANT client state changes via session pollingprevious_state = 'MENUS'

previous_state = 'MENUS'current_state = session.get('loopState')  # MENUS, REPLAY, etc.

current_state = session.get('loopState')  # MENUS, REPLAY, etc.if previous_state == 'MENUS' and current_state == 'REPLAY':

if previous_state == 'MENUS' and current_state == 'REPLAY':    perform_injection()

    perform_injection()```

```

#### 3. File Swapping Technique

#### 3. File Swapping Technique```python

```python# Backup original, copy injection file, restore after replay ends

# Backup original, copy injection file, restore after replay endsbackup_success = self.file_manager.backup_file(host_path)

backup_success = self.file_manager.backup_file(host_path)shutil.copy2(injection_file, host_file)  # Temporary swap

shutil.copy2(injection_file, host_file)  # Temporary swap# Auto-restore when replay ends via session monitoring

# Auto-restore when replay ends via session monitoring```

```

#### 4. Region-Aware API Calls

#### 4. Region-Aware API Calls```python

```python# Use region-specific endpoints for all VALORANT APIs

# Use region-specific endpoints for all VALORANT APIspd_base = f"https://pd.{region}.a.pvp.net"  # Player Data

pd_base = f"https://pd.{region}.a.pvp.net"  # Player Dataglz_base = f"https://glz-{region}-1.{region}.a.pvp.net"  # Game Lobby

glz_base = f"https://glz-{region}-1.{region}.a.pvp.net"  # Game Lobbyshared_base = f"https://shared.{region}.a.pvp.net"  # Shared services

shared_base = f"https://shared.{region}.a.pvp.net"  # Shared services```

```

## Critical Developer Workflows

#### 5. Callback-Based Event System

```python### Development Setup

# Register callbacks for session state changes```bash

self.session_monitor.add_callback('replay_start', self.on_replay_start)# Install dependencies

self.session_monitor.add_callback('replay_end', self.on_replay_end)pip install -r requirements.txt

self.session_monitor.add_callback('error', self.on_monitor_error)

```# Run development version

python modern_replay_injector_ui.py

## Critical Developer Workflows

# VALORANT must be running for API access and testing

### Development Setup```

```bash

# Install dependencies### Building Executables

pip install -r requirements.txt```bash

# Use PyInstaller for standalone .exe creation

# Run development versionpython build_v1_1.py

python modern_replay_injector_ui.py

# Includes all dependencies and assets

# VALORANT must be running for API access and testing# Output: releases/v1.1/SoupsValorantReplayTool_v1.1.exe

``````



### Building Executables### Testing Injection Flow

```bash1. Start VALORANT client


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zachleolewis/SoupsValorantReplayTool](https://github.com/zachleolewis/SoupsValorantReplayTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
