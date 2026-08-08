---
trigger: always_on
description: This repository uses a strict file-safety workflow:
---

# Aivatar Project Notes

## Agent Operating Rules

This repository uses a strict file-safety workflow:

- Default to limited read-only mode: inspect directories, read files, search text, and propose edits.
- Do not modify, rename, move, format, refactor, overwrite, or delete existing files without explicit user approval for the exact patch or edit list.
- Before editing an existing file, describe the affected files and show the proposed patch/diff or a clear edit list, then wait for explicit confirmation.
- Creating new files and folders is allowed when requested; files created by the agent may be modified by the agent.
- Before deleting anything, list every file or directory, explain why it can be safely removed, and wait for explicit confirmation.
- Bulk deletion is not allowed. If cleanup would affect many files, propose a plan for the user to execute or adjust manually.
- Treat raw/original/source data as strictly read-only. Any processing should operate on clearly labeled copies or derived files.

## Project Goal

Aivatar is a Tauri 2 + React + TypeScript desktop companion for AI coding agents. It displays a retro pixel-style room where a customizable pixel companion lives, wanders, sleeps, works, plays, decorates its room, and reacts to live agent status in real time. The original/default companion is the pixel octopus, with additional appearance-only characters sharing the same runtime behavior model.

The product direction is a mix of:

- Desktop pet / electronic companion.
- Pixel room simulator with a cozy retro game feel.
- Live visual state monitor for Codex, Claude Code, and other AI apps/CLIs that can post status events.
- Extensible pet system with feeding, inventory, shop, placeable decor/furniture, room editing, autonomous activities, future room upgrades, skins, and content packs.

The MVP should prioritize the feeling that the avatar is alive, while still letting agent status strongly drive avatar behavior.

## Current Stack

- Desktop shell: Tauri 2
- Frontend: React 18 + TypeScript + Vite
- Rendering: HTML Canvas with pixel-art styling
- Runtime content: JSON config loaded from `public/config/aivatar.config.json`
- Local status integration:
  - WebSocket for Aivatar UI updates
  - HTTP bridge for external scripts/tools to POST generic AI agent status

PowerShell may block `npm.ps1`; use `npm.cmd` in this environment.

## Important Commands

Install dependencies:

```powershell
npm.cmd install
```

Run web UI:

```powershell
npm.cmd run dev
```

The web UI dev server and Tauri dev URL are currently unified on:

```text
http://localhost:1420/
```

Keep using `localhost` for development previews unless intentionally testing a separate origin. Browser `localStorage` is origin-scoped, so `http://127.0.0.1:1420/` and `http://localhost:1420/` have separate saves.

When the main OneDrive checkout already owns port `1420`, a Codex worktree
preview can be run on a separate port:

```powershell
npm.cmd run dev -- --host 127.0.0.1 --port 1421 --strictPort
```

Use `http://127.0.0.1:1421/` for that worktree preview. This origin has its
own `localStorage`, including save state and UI theme choice.
If `1421` is also occupied by another checkout or preview, use the next free
strict port, for example `1422`, and remember that this creates another
origin-scoped `localStorage` save/theme context.

If nearby worktree preview ports are already occupied, this worktree has been
previewed on:

```powershell
node .\node_modules\vite\bin\vite.js --host 127.0.0.1 --port 1427 --strictPort
```

Use `http://127.0.0.1:1427/` for the current Gray Tech Floor and Blue Persian
Rug shop preview. This origin has separate `localStorage` from `1420`, `1421`,
`1422`, `1423`, `1424`, `1425`, and `1426`.

Run desktop app:

```powershell
$env:PATH = "C:\Program Files\nodejs;$env:USERPROFILE\.cargo\bin;$env:PATH"
$env:CARGO_TARGET_DIR = "$env:TEMP\aivatar-cargo-target"
cmd.exe /c 'call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvars64.bat" && npm.cmd run tauri dev'
```

The Tauri desktop app attempts to start the local status bridge automatically during app setup. The retained internal Debug implementation includes a `Start bridge` action for QA builds, but the main-room Debug card is hidden in normal runtime UI. Starting the bridge from Tauri also attempts to start Codex Desktop session discovery.

The desktop shell registers the Tauri single-instance plugin before other plugins. A second app launch should focus/show the existing main window instead of keeping another full WebView process alive. Fixed-size room windows explicitly disable native maximize, and the side-panel resize path clears maximized state before applying Aivatar-owned window sizes so Windows restore bounds cannot corrupt the normal room size.

Run real local status bridge:

```powershell
npm.cmd run status:bridge
```

Run the bridge Origin/CORS smoke test:

```powershell
npm.cmd run status:bridge:smoke
```

Run the shop purchase pressure smoke test:

```powershell
npm.cmd run shop:purchase:smoke
```

Run the DOM-level shop UI smoke test:

```powershell
npm.cmd run shop:ui:smoke
```

Run Codex Desktop session discovery:

```powershell
npm.cmd run status:discover
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruiwuniu/Aivatar-Demo](https://github.com/ruiwuniu/Aivatar-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
