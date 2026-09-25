---
trigger: always_on
description: Instructions for coding agents working with clipaste — both **installing it for a
---

# AGENTS.md

Instructions for coding agents working with clipaste — both **installing it for a
user** and **contributing to this repository**.

clipaste is a clipboard daemon that makes screenshot paste work in terminal AI
tools (Claude Code, Codex CLI, Cursor CLI), locally on macOS/Windows and across
SSH/WSL2 boundaries. Graphical Linux hosts provide read-only PNG clipboard
capture for the SSH bridge.

---

## Part 1 — Installing clipaste for a user

### The one command that tells you what to do

```bash
clipaste doctor --json
```

This is the entry point. It classifies the machine, runs the checks that are
meaningful for that machine, and returns `fix` commands or guidance where available.
Do not guess at the state of a clipaste install — ask `doctor`.

```json
{
  "version": "2.4.1",
  "os": "macos",
  "role": "clipboard-host",
  "status": "warn",
  "checks": [
    {
      "name": "daemon",
      "status": "ok",
      "detail": "daemon responding on 127.0.0.1:18340",
      "fix": null
    },
    {
      "name": "clipboard",
      "status": "warn",
      "detail": "no image staged yet — take a screenshot, then re-run doctor",
      "fix": null
    }
  ]
}
```

| Field | Contract |
|---|---|
| `role` | `clipboard-host` / `ssh-remote` / `wsl2` / `unsupported-host` decides which checks apply |
| `status` | worst of all checks: `ok`, `warn`, `fail` |
| `checks[].status` | `ok` / `warn` / `fail` |
| `checks[].fix` | a remediation command or guidance, or `null` when no single command applies |
| exit code | `0` usable (ok **or** warn), `1` broken, `2` bad arguments |

A `warn` is not a failure. "No screenshot on the clipboard yet" is the normal
state of a freshly installed machine — do not report it to the user as a
problem, and do not try to fix it.

Consumers of the JSON output must accept `unsupported-host` for platforms
without a backend. Do not apply it to Linux as a whole: Linux host diagnostics
must distinguish missing tools, missing session access, and missing compositor
data-control from consumer helper or bridge failures.

### Decide where you are before installing anything

clipaste has two sides and they install differently. Getting this wrong is the
single most common mistake.

| OS / context | Local clipboard-host daemon | Consumer of another host's clipboard |
|---|---|---|
| macOS | Supported | SSH remote via `clipaste-paste` |
| Windows | Supported | Via WSL2 |
| Native Linux desktop | Read-only `image/png` via Wayland data-control or X11/XWayland | Supported over SSH via shims / `clipaste-paste` |
| Headless Linux | No display; host startup fails with guidance | Supported with configured helpers / SSH |
| WSL2 | No; the Windows daemon is required | Supported via `wsl-setup` |

```text
Clipboard host                              Consumer
macOS / Windows / graphical Linux
  PNG cache -> loopback HTTP -> SSH tunnel -> SSH remote
  run ssh-setup on this host                 shims / clipaste-paste

Windows -> HTTP over WSL networking -------> WSL2
  Windows daemon                            run wsl-setup inside the distro

Local macOS / Windows: existing clipboard normalization and paste
                       (skipped with CLIPASTE_SERVER_ONLY=1; HTTP serving remains)
Local Linux: read-only PNG capture, no added text-path paste
```

`clipaste doctor --json` reports which side you are on as `role`. Trust it over
`uname`: an SSH session into a Mac is `ssh-remote`, not `clipboard-host`.

The role contract applies in this order:

1. WSL context: `wsl2`, even if SSH environment variables are also present.
2. An actual SSH session: `ssh-remote`, including SSH into macOS.
3. A local macOS or Windows machine, or graphical local Linux session:
   `clipboard-host`. Linux backend access still needs validation.
4. A headless Linux machine with a configured consumer helper: `ssh-remote`,
   even without SSH environment variables; keep checking its helper and bridge.
5. Linux without a display or consumer indicators: `clipboard-host` with a
   failing `backend` check and actionable graphical-session guidance. Other
   platforms without a backend or consumer indicators retain `unsupported-host`.

WSL remains a Windows consumer even when WSLg supplies display variables. SSH
session detection takes precedence over a Linux desktop's display variables.
Do not reclassify those consumers as hosts.

### Install on the clipboard host

macOS:

```bash
brew install hqhq1025/clipaste/clipaste
brew services start clipaste
clipaste doctor --json
```

Windows (PowerShell, no admin needed):

```powershell
irm https://raw.githubusercontent.com/hqhq1025/clipaste/main/install.ps1 | iex
clipaste doctor --json
```

Linux desktop (Ubuntu package example):

```bash
sudo apt install wl-clipboard xclip curl
```

Releases since v2.5.0 include static Linux archives for `x86_64-unknown-linux-musl`
and `aarch64-unknown-linux-musl`. Download the matching architecture, verify
against the release's `SHA256SUMS`, extract it, then:

```bash
install -Dm755 clipaste "$HOME/.local/bin/clipaste"
export PATH="$HOME/.local/bin:$PATH"
clipaste
```

Source installation with Rust/Cargo is also supported:
`cargo install --git https://github.com/hqhq1025/clipaste --tag v2.6.0 --locked`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hqhq1025/clipaste](https://github.com/hqhq1025/clipaste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
