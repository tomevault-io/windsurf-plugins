---
trigger: always_on
description: > This file is for AI coding agents (Claude Code, Cursor, Codex, Devin, OpenClaw).
---

# AeroFTP CLI — Agent Integration Guide

> This file is for AI coding agents (Claude Code, Cursor, Codex, Devin, OpenClaw).
> It describes how to use AeroFTP CLI for remote operations without credentials.

## Quick Start

```bash
# 1. See available servers (no credentials shown)
aeroftp-cli profiles --json

# 2. List files on a server
aeroftp-cli ls --profile "Server Name" /path/ --json

# 3. Upload a file
aeroftp-cli put --profile "Server Name" ./local-file.txt /remote/path/file.txt

# 4. Download a file
aeroftp-cli get --profile "Server Name" /remote/file.txt ./local-file.txt

# 5. Sync a directory
aeroftp-cli sync --profile "Server Name" ./local-dir/ /remote-dir/ --dry-run

# 6. Cross-profile copy between two saved servers
aeroftp-cli transfer "Source Server" "Destination Server" /src/path /dst/path --recursive
```

## How Credentials Work

You do NOT need passwords, tokens, or API keys. The user has saved their servers in an encrypted vault. Use `--profile "Name"` to connect — credentials are resolved internally by the Rust backend and never exposed to your process.

```bash
# WRONG — do not ask the user for passwords
aeroftp-cli ls sftp://user:password@host /path/

# RIGHT — use saved profiles
aeroftp-cli ls --profile "My Server" /path/
```

## Discovery

```bash
# List all saved servers with protocol, host, path
aeroftp-cli profiles --json

# Get full CLI capabilities as structured JSON
aeroftp-cli agent-info --json
```

The `profiles --json` output:
```json
[
  {
    "id": "srv_123",
    "name": "Production",
    "protocol": "sftp",
    "host": "prod.example.com",
    "port": 22,
    "username": "deploy",
    "initialPath": "/var/www"
  }
]
```

## Common Commands

### File Operations

| Command | Usage | Description |
|---------|-------|-------------|
| `ls` | `aeroftp-cli ls --profile NAME /path/ [-l] [--json]` | List directory contents |
| `get` | `aeroftp-cli get --profile NAME /remote/file [./local]` | Download file |
| `put` | `aeroftp-cli put --profile NAME ./local /remote/path` | Upload file |
| `cat` | `aeroftp-cli cat --profile NAME /remote/file` | Print file to stdout |
| `stat` | `aeroftp-cli stat --profile NAME /remote/file [--json]` | File metadata |
| `find` | `aeroftp-cli find --profile NAME /path/ "*.ext" [--json]` | Search files |
| `tree` | `aeroftp-cli tree --profile NAME /path/ [-d depth] [--json]` | Directory tree |

### Modify Operations

| Command | Usage | Description |
|---------|-------|-------------|
| `mkdir` | `aeroftp-cli mkdir --profile NAME /remote/new-dir` | Create directory |
| `rm` | `aeroftp-cli rm --profile NAME /remote/file` | Delete file |
| `rm -rf` | `aeroftp-cli rm --profile NAME /remote/dir/ -rf` | Delete directory recursively |
| `mv` | `aeroftp-cli mv --profile NAME /old/path /new/path` | Move or rename |

### Bulk Operations

| Command | Usage | Description |
|---------|-------|-------------|
| `get -r` | `aeroftp-cli get --profile NAME /remote/dir/ ./local/ -r` | Download directory |
| `put -r` | `aeroftp-cli put --profile NAME ./local/ /remote/dir/ -r` | Upload directory |
| `get glob` | `aeroftp-cli get --profile NAME "/path/*.csv"` | Download matching files |
| `put glob` | `aeroftp-cli put --profile NAME "./*.json" /remote/` | Upload matching files |
| `sync` | `aeroftp-cli sync --profile NAME ./local/ /remote/` | Bidirectional sync |
| `sync --dry-run` | `aeroftp-cli sync --profile NAME ./local/ /remote/ --dry-run` | Preview sync |
| `sync --immutable` | `aeroftp-cli sync --profile NAME ./local/ /remote/ --immutable` | Never overwrite existing files |
| `sync --files-from` | `aeroftp-cli sync --profile NAME ./local/ /remote/ --files-from list.txt` | Transfer only listed files |
| `sync --fast-list` | `aeroftp-cli sync --profile NAME ./local/ /remote/ --fast-list` | S3 recursive listing (fewer API calls) |
| `cleanup` | `aeroftp-cli cleanup --profile NAME /path/ [--force]` | Find/delete orphaned .aerotmp files |
| `dedupe` | `aeroftp-cli dedupe --profile NAME /path/ --mode list` | Find duplicate files |
| `transfer` | `aeroftp-cli transfer "SRC" "DST" /src /dst --recursive` | Cross-profile transfer between saved servers |
| `transfer-doctor` | `aeroftp-cli transfer-doctor "SRC" "DST" /src /dst --json` | Preflight plan and risk summary |

### Advanced Operations

| Command | Usage | Description |
|---------|-------|-------------|
| `mount` | `aeroftp-cli --profile NAME mount /mnt/point` | Mount remote as local FUSE filesystem |
| `ncdu` | `aeroftp-cli --profile NAME ncdu / [--json]` | Interactive disk usage explorer |
| `serve http` | `aeroftp-cli --profile NAME serve http` | Expose remote as HTTP server |
| `serve webdav` | `aeroftp-cli --profile NAME serve webdav` | Expose remote as WebDAV server |
| `serve ftp` | `aeroftp-cli --profile NAME serve ftp` | Expose remote as FTP server |
| `serve sftp` | `aeroftp-cli --profile NAME serve sftp` | Expose remote as SFTP server |
| `daemon start` | `aeroftp-cli daemon start` | Start background service |
| `jobs add` | `aeroftp-cli jobs add get --profile NAME /file` | Queue background transfer |
| `jobs list` | `aeroftp-cli jobs list` | List queued/running jobs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axpdev-lab/aeroftp](https://github.com/axpdev-lab/aeroftp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
