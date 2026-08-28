---
trigger: always_on
description: ﻿# Agent Guide: Localize Codex Desktop to Chinese (zh-CN)
---

﻿# Agent Guide: Localize Codex Desktop to Chinese (zh-CN)

## Mission

Install the Chinese UI localization for **Codex Desktop** on this Windows machine by running the scripts in this repository. The user asked for this in one sentence; do the whole job for them: fetch the latest repo, detect, install, restart, verify, and report.

The restart is a **transitional supervised restart** (a few seconds; no background process remains). The launcher (`scripts/launch-zh-cn.ps1`) writes `%USERPROFILE%\.codex\zh-cn-agent\launch-result.json`; the agent reads it after the window restarts.

## Hard safety boundaries (never violate)

- Never modify files under `C:\Program Files\WindowsApps` directly. The installer copies the app first.
- Never read, write, print, or send `auth.json`, API keys, tokens, or `config.toml` secrets.
- Never change `model_provider`, `model`, or any model/API settings in `config.toml`.
- Never make network calls to OpenAI. Installation works fully offline.
- The tool intentionally installs one logon task — the "入口自动切换助手" (entry auto-switch guard, `scripts/entry-guard.ps1`) — so Codex opens in Chinese from any entry point. Install it as the installer does, never remove it during install, and never create any additional persistent watchers. It is removed automatically by restore/uninstall.
- The installer stops and restarts Codex processes (including this session). That is expected; tell the user the window will restart in Chinese.

## Prerequisites

- Windows 10/11, Codex Desktop installed from Microsoft Store. Detect it:

```powershell
Get-AppxPackage | Where-Object { $_.Name -match 'Codex|OpenAI' } | Select-Object Name, Version, InstallLocation
```

- If not found: tell the user to install Codex from Microsoft Store first, then stop.

## Step 0 - Get the repository (always use the latest)

Try the sources below **in order** and stop at the first one that works.
While fetching, never change system network settings (proxy, hosts, DNS, firewall, routing) and do not install any network tool — previous sessions caused network loss this way. Use only the commands below.

1) Git clone (GitHub direct):

```powershell
git clone https://github.com/qibuliaoming/codex-zh-cn-agent.git
cd codex-zh-cn-agent
```

2) If `git clone` fails due to network, fall back to the GitHub zip archive (same content):

```powershell
$tmpZip = Join-Path $env:TEMP "codex-zh-cn-agent.zip"
Invoke-WebRequest -Uri "https://codeload.github.com/qibuliaoming/codex-zh-cn-agent/zip/refs/heads/main" -OutFile $tmpZip
Expand-Archive -Path $tmpZip -DestinationPath . -Force
cd codex-zh-cn-agent-main
```

3) If GitHub direct and codeload are both blocked, try mirror proxies (in order, first reachable wins):

```powershell
$mirrorUrls = @(
  "https://gh-proxy.com/https://github.com/qibuliaoming/codex-zh-cn-agent/archive/refs/heads/main.zip",
  "https://ghfast.top/https://github.com/qibuliaoming/codex-zh-cn-agent/archive/refs/heads/main.zip",
  "https://ghproxy.net/https://github.com/qibuliaoming/codex-zh-cn-agent/archive/refs/heads/main.zip"
)
$tmpZip = Join-Path $env:TEMP "codex-zh-cn-agent.zip"
foreach ($u in $mirrorUrls) {
  try {
    Invoke-WebRequest -Uri $u -OutFile $tmpZip -TimeoutSec 30
    if ((Get-Item $tmpZip).Length -gt 1000) { break }
  } catch { }
}
Expand-Archive -Path $tmpZip -DestinationPath . -Force
cd codex-zh-cn-agent-main
```

4) If all sources above fail: do **not** keep retrying and do **not** modify any network settings. Tell the user (in Chinese): GitHub 在你当前网络下无法访问。请从仓库 Release 页下载最新 `codex-zh-cn-agent-v1.3.2.zip`（可以请朋友帮忙下载后拷贝过来），解压后双击「安装汉化.bat」即可。Then stop.

## Step 1 - Notify the user before installing

Tell the user (in Chinese): 接下来会请求一次管理员权限（UAC 弹窗请点「是」）；安装完成后当前窗口会自动以中文版重启（过渡监督式，几秒完成），并安装一个登录自启的入口自动切换助手，以后从任何入口打开 Codex 都是中文版。

## Step 2 - Run the installer

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "scripts\install-zh-cn.ps1" -Action install -NoPause
```

The parent process writes `status: pending` to `%USERPROFILE%\.codex\zh-cn-agent\install-result.json`, requests UAC elevation, and **exits immediately**. Do not assume failure.

The elevated install process calls the supervised launcher at the end; the current window restarts as the localized app (a few seconds). The agent session may be interrupted during this restart; the thread resumes afterwards and the agent continues.

## Step 3 - Poll the result file (up to 5 minutes, every 10 seconds)

```powershell
$rf = Join-Path $env:USERPROFILE ".codex\zh-cn-agent\install-result.json"
$start = Get-Date
$r = $null
do {
    Start-Sleep -Seconds 10
    if (Test-Path $rf) { $r = Get-Content -Raw -Encoding UTF8 $rf | ConvertFrom-Json }
} until (($r -and $r.status -in @("ok","fail")) -or ((Get-Date) - $start).TotalMinutes -ge 5)
```

Interpret the result:

- `ok` → continue to Step 4.
- `fail` → read `$r.code`, `$r.message`, `$r.diagFile` and handle per the table below.
- Still `pending` after 5 minutes → the user most likely clicked **No** on the UAC prompt. Tell them: 请在 UAC 弹窗中点击「是」，然后重新发送这条指令。Then stop.

After the window restarts, also read `%USERPROFILE%\.codex\zh-cn-agent\launch-result.json`:

- `status: ok`（`LAUNCH_OK` or `ALREADY_RUNNING`）→ continue to Step 4.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qibuliaoming/codex-zh-cn-agent](https://github.com/qibuliaoming/codex-zh-cn-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
