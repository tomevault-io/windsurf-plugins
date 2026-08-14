---
trigger: always_on
description: ﻿<!-- agents-install-guide -->
---

﻿<!-- agents-install-guide -->
<!-- mcp-name: io.github.bimwright/rvt-mcp -->

# AGENTS.md — rvt-mcp install guide for AI agents

This file is machine-readable install instructions for AI coding agents (Claude Code, Cursor, Cline, VS Code Copilot, Gemini CLI, Antigravity, OpenCode, Codex, and any other stdio MCP client). A user pointed you here because they want rvt-mcp (a Revit MCP server) wired up without hand-editing config files themselves.

**What you can do:**

- Install the client setup ZIP from GitHub Releases.
- Deploy the bundled plugin into `%APPDATA%\Autodesk\Revit\Addins\<year>\` for every installed Revit year.
- Wire supported MCP hosts with the bundled self-contained server executable.
- Verify the handshake and roll back on failure.

**What you cannot do:**

- Install Revit, manage Revit licensing, or launch Revit for the first time. If Revit 2022–2027 is not installed, stop and tell the user.
- Install .NET 8 SDK, clone the repo, restore NuGet packages, or build source for a normal client install. If the client setup ZIP is unavailable, stop and report that the client installer is not available.

---

## Rules for agents

**Read these before touching anything. They exist so rvt-mcp stays predictable, auditable, and reversible.**

1. **Preview every change.** Use `-WhatIf`, `--dry-run`, or a printed diff before any write. Tell the user the exact file path and the exact change.
2. **Use the client setup ZIP for client machines.** Do not fall back to `dotnet tool install`, source build, or repo clone unless the user explicitly asks for developer installation.
3. **Two explicit approval gates — do not collapse without the user saying so:**
   - Before running `install.ps1` without `-WhatIf`.
   - Before editing any MCP host config file outside the setup installer's own preview/apply flow.
4. **Never bypass the Revit undo stack at runtime.** rvt-mcp's design guarantee is that every edit is reviewable and reversible. Don't advise users to work around transaction wrapping or disable `batch_execute` safety.
5. **On any failure, offer rollback.** Config edits are auto-backed up to `<file>.bimwright.bak`. The full stack comes off with the bundled `uninstall.ps1 -Yes`.
6. **Verify before claiming done.** After wiring, run `tools/list` in the host and confirm the single `rvt-mcp` entry responds, then call `get_current_view_info` with no args.

If the user explicitly says "skip the prompts, just install" — still do gate 1 (preview) and gate 5 (verify), but collapse gates 2 and 3 into a single upfront approval. **Never silently skip preview or verify.**

### Baked-tool routing

When the user's request may match a personal baked tool, call list_baked_tools first.
In v0.3.x baked tools never appear directly in native tools/list.
Run accepted tools through run_baked_tool name=<tool_name>.

---

## Prerequisites (check first, stop if any are missing)

| Requirement | How to check (PowerShell) | If missing |
|---|---|---|
| Windows | `[System.Runtime.InteropServices.RuntimeInformation]::IsOSPlatform('Windows')` | Stop — Revit is Windows-only. |
| Revit 2022–2027 | `Get-ChildItem 'HKLM:\SOFTWARE\Autodesk\Revit\' -ErrorAction SilentlyContinue` | Tell the user to install Revit. You cannot. |
| PowerShell ≥5.1 | `$PSVersionTable.PSVersion` | Prompt: <https://aka.ms/powershell>. |

If Revit is not running when the user first tries a tool call, that's fine — the server only needs Revit alive at tool-call time, not at install time.

---

## Step 1 — Download the client setup ZIP

```powershell
$tag = (Invoke-RestMethod https://api.github.com/repos/bimwright/rvt-mcp/releases/latest).tag_name
$zip = "$env:TEMP\RvtMcp.Setup-$tag-win-x64.zip"
$dir = "$env:TEMP\RvtMcp.Setup-$tag-win-x64"
Invoke-WebRequest "https://github.com/bimwright/rvt-mcp/releases/download/$tag/RvtMcp.Setup-$tag-win-x64.zip" -OutFile $zip
Expand-Archive $zip -DestinationPath $dir -Force
```

If the setup asset is not present on the release, stop. Do not clone, build, or install the .NET SDK for a client machine.

---

## Step 2 — Preview, then install

```powershell
powershell -ExecutionPolicy Bypass -File "$dir\install.ps1" -WhatIf
powershell -ExecutionPolicy Bypass -File "$dir\install.ps1"
```

The installer detects Revit years, installs all matching plugin ZIPs, copies the bundled server to `%LOCALAPPDATA%\RvtMcp\rvt\server\<version>\`, and wires detected Codex/OpenCode/Claude configs with one auto-detect entry named `rvt-mcp`.

Use `-Client codex`, `-Client opencode`, `-Client claude`, or `-Client none` when the user wants a specific config behavior.

---

## Step 3 — Wire the MCP host

Pick the host the user is actually running. The default wiring is one MCP entry named `rvt-mcp`; the server auto-detects the running Revit instance. The installer still deploys plugins for every detected Revit year:

```powershell
$years = Get-ChildItem 'HKLM:\SOFTWARE\Autodesk\Revit\' -ErrorAction SilentlyContinue |
         ForEach-Object { if ($_.PSChildName -match '^(\d{4})$' -and [int]$Matches[1] -ge 2022 -and [int]$Matches[1] -le 2027) { $Matches[1] } }
```

### Canonical snippet (7 of 9 hosts)

Most hosts use `{ "mcpServers": { ... } }` with this per-server shape:

```json
{
  "mcpServers": {
    "rvt-mcp": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bimwright/rvt-mcp](https://github.com/bimwright/rvt-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
