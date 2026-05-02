---
trigger: always_on
description: Reverse-engineered Linux port of Claude Desktop's Cowork (Local Agent Mode).
---

# Claude Cowork Linux

Reverse-engineered Linux port of Claude Desktop's Cowork (Local Agent Mode).
Replaces macOS VM + Swift addon with direct process spawning on Linux.

## Architecture Overview

```
                        ┌─────────────────────────────────┐
                        │     Claude Desktop (Electron)    │
                        │         app.asar (minified)      │
                        └──────────────┬──────────────────┘
                                       │
                       ┌───────────────┴───────────────────┐
                       │                                   │
            ┌──────────▼──────────┐             ┌──────────▼──────────┐
            │  ipc-handler-setup  │             │ Swift Stub (index.js)│
            │  (IPC handlers,     │             │ stubs/@ant/claude-   │
            │   session state)    │             │ swift/js/index.js    │
            └──────────┬──────────┘             └──────────┬──────────┘
                       │                                   │
            ┌──────────▼──────────┐             ┌──────────▼──────────┐
            │  stubs/cowork/      │             │   vm.spawn() / kill  │
            │  (15 orchestration  │             │   filterEnv()        │
            │   modules: IPC tap, │             │   path translation   │
            │   session store,    │             └──────────┬──────────┘
            │   process mgr, ...) │                        │
            └─────────────────────┘             ┌──────────▼──────────┐
                                                │   Claude Code CLI   │
                                                │  (~/.local/bin/     │
                                                │        claude)      │
                                                └─────────────────────┘
```

**Critical**: The asar's own `LocalAgentModeSessionManager` drives the spawn lifecycle.
It calls `vm.spawn()` on the Swift stub directly. The stubs/cowork/ modules provide
supporting orchestration (EIPC discovery, session persistence, process management).

## Current Status

- Regular Claude chat: WORKING
- Cowork (Local Agent Mode): WORKING (auth fixed 2026-02-13)
- Session persistence between restarts: IMPLEMENTED (env var path fix 2026-02-13)

## Key Files

| File | Purpose | Modified by us? |
|------|---------|-----------------|
| `stubs/@ant/claude-swift/js/index.js` | **THE** critical stub. Replaces macOS Swift VM addon. Handles `vm.spawn()`, `filterEnv()`, path translation, mount symlinks, process I/O | YES -- primary |
| `stubs/@ant/claude-native/index.js` | Auth (xdg-open), keyboard constants, platform helpers | YES -- primary |
| `stubs/cowork/session_orchestrator.js` | Session lifecycle: start, stop, message routing, transcript coordination | YES -- primary |
| `stubs/cowork/ipc_tap.js` | EIPC channel prefix auto-discovery from runtime handler registration | YES -- primary |
| `stubs/cowork/asar_adapter.js` | Asar file operations with path traversal protection | YES -- primary |
| `stubs/cowork/dirs.js` | XDG Base Directory paths, macOS-to-XDG path aliasing | YES -- primary |
| `stubs/cowork/process_manager.js` | Process spawning with argument arrays, lifecycle management | YES -- primary |
| `stubs/cowork/session_store.js` | Session persistence (sessions.json), hydration | YES -- primary |
| `stubs/cowork/credential_classifier.js` | Credential detection patterns for token filtering | YES -- security |
| `stubs/cowork/sessions_api.js` | Sessions API with CRLF guards, FD bounds checking | YES -- security |
| `stubs/frame-fix/frame-fix-wrapper.js` | Early bootstrap: TMPDIR fix, platform spoofing, graceful shutdown | YES -- primary |
| `launch.sh` | Launch script: password-store detection, Wayland/Ozone flags, Code tab binary fixup, asar repacking | YES |
| `fetch-dmg.js` | Auto-download Claude DMG via Node.js (replaces Python/rnet) | YES |

## Critical Path Chains

### Chain 1: Spawn → CLI Execution

```
User sends message in webapp
  → webapp calls IPC: LocalAgentModeSessions_$_sendMessage
  → asar's LocalAgentModeSessionManager handles it
  → calls vm.spawn() on Swift stub with:
      - command: /usr/local/bin/claude
      - args: [--resume <ccId>, --output-format stream-json, ...]
      - envVars: {CLAUDE_CONFIG_DIR: "/sessions/<name>/mnt/.claude", CLAUDE_CODE_OAUTH_TOKEN: <token>, ...}
      - additionalMounts: {".claude": {path: ".../.claude", mode: "rwd"}, ...}
  → Swift stub spawn():
      1. createMountSymlinks() — sets up mnt/ symlinks to host dirs
      2. resolveClaudeBinaryPath() — finds ~/.local/bin/claude
      3. Translates /sessions/ paths in ARGS → SESSIONS_BASE host paths
      4. Translates /sessions/ paths in ENVVARS → SESSIONS_BASE host paths  ← NEW FIX
      5. filterEnv() — allowlist + asar env vars merged
      6. nodeSpawn(command, args, {env, cwd})
  → Claude Code CLI runs on host, authenticates via CLAUDE_CODE_OAUTH_TOKEN
  → CLI streams JSON to stdout → stub relays via _onStdout → asar processes
```

**Where things can break**:
- If envVars aren't path-translated, `CLAUDE_CONFIG_DIR` points to wrong dir (transcripts lost)
- If `CLAUDE_CODE_OAUTH_TOKEN` is missing/corrupted, CLI gets 401
- If binary path resolution fails, spawn errors silently


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnzfitch/claude-cowork-linux](https://github.com/johnzfitch/claude-cowork-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
