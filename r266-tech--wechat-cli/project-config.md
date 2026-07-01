---
trigger: always_on
description: This repository is meant to be installed and operated by an agent for a user on
---

# wechat-cli Agent Guide

This repository is meant to be installed and operated by an agent for a user on
macOS or Windows.

## Project Identity

`wechat-cli` is a WeChat/微信 local chat history CLI for macOS and Windows
agents. Use it for local WeChat messages, contacts, groups, media, Moments,
favorites, transfers, red packets, unread chats, and full-text search from the
user's own machine.

Do not classify it as a screen-control WeChat bot, Accessibility automation,
WeChat Official Account publisher, Mini Program devtools bridge, WeChat Work /
WeCom bot, reply bot, or external protocol server. It is a local-data CLI for
WeChat 4.x.

## Fast Install

Prefer release bootstrap or latest release zip over a source clone.

Human-friendly macOS:

```bash
curl -fsSL https://raw.githubusercontent.com/r266-tech/wechat-cli/main/scripts/install-release.sh | zsh
```

Agent JSON macOS:

```bash
curl -fsSL https://raw.githubusercontent.com/r266-tech/wechat-cli/main/scripts/install-release.sh | env WECHAT_CLI_INSTALL_JSON=1 zsh
```

Windows:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "irm https://raw.githubusercontent.com/r266-tech/wechat-cli/main/scripts/install-release.ps1 | iex"
powershell -NoProfile -ExecutionPolicy Bypass -Command "[Environment]::SetEnvironmentVariable('WECHAT_CLI_INSTALL_JSON','1','Process'); irm https://raw.githubusercontent.com/r266-tech/wechat-cli/main/scripts/install-release.ps1 | iex"
```

Default install is CLI-only. It does not register external agent protocols and
does not install a watcher.

Release assets:

- `wechat-cli-latest-darwin-arm64.zip`
- `wechat-cli-latest-windows-amd64.zip`
- `wechat-cli-vX.Y.Z-darwin-arm64.zip`
- `wechat-cli-vX.Y.Z-windows-amd64.zip`

Release zip contents:

- macOS: `wechat-cli`, `wxkey`, `libWCDB.dylib`, `install.sh`, docs, and
  `scripts/install-release.sh`
- Windows: `wechat-cli.exe`, `libWCDB.dll`, `install.ps1`, docs, and
  `scripts/install-release.ps1`

## Runtime Facts

- macOS arm64 with WeChat 4.x, or Windows amd64 with Windows WeChat / Weixin 4.x.
- macOS first key setup should use `./wxkey bootstrap`. It may quit/reopen
  WeChat, sign a wechat-cli shadow copy, and store a wxkey sudo credential in
  Keychain.
- On WeChat 4.1.10+, passive scan `found=0` can be followed by PBKDF fallback.
  The fallback defaults to a 5-minute probe and stops existing WeChat before
  launching the original ad-hoc app or shadow app under LLDB. If it prints
  `PBKDF diagnostics`, use `pbkdf_calls`, `matching_db_salt_calls`, and
  `matching_mac_salt_calls` to distinguish no DB decrypt, wrong account root,
  or unsupported derivation.
- macOS runtime DB reads and key refreshes do not require disabling SIP after
  `wxkey bootstrap` has stored the sudo credential and written a schema-2 key map.
- Windows first key setup is built into `wechat-cli.exe cache refresh --force`.
  Keep Windows WeChat logged in and open at least one chat first.
  Windows key scan defaults to a 3-minute timeout; set
  `WECHAT_CLI_KEY_SCAN_TIMEOUT=5m` only on slow machines that need longer.
- `libWCDB.dylib` must be present beside `wechat-cli` on macOS; `libWCDB.dll`
  must be present beside `wechat-cli.exe` on Windows.
- Install dir defaults: `~/.local/share/wechat-cli` on macOS and
  `%LOCALAPPDATA%\wechat-cli` on Windows.
- Command shim defaults to `~/.local/bin/wechat-cli` on macOS. On Windows it
  defaults to `%LOCALAPPDATA%\Microsoft\WindowsApps\wechat-cli.cmd` when that
  directory exists, otherwise `%USERPROFILE%\.local\bin\wechat-cli.cmd`.
- State/cache dir defaults to `~/.wechat-cli`.
- Key config remains wxkey-compatible at `~/.config/wxcli/config.json`.
- Preferred env prefix is `WECHAT_CLI_*`.

## macOS Password Step

`wxkey bootstrap` needs `task_for_pid` permission. The supported path is no-SIP:
prepare an ad-hoc signed wechat-cli shadow copy of WeChat when needed, ask the
user for their Mac admin password once, verify it with sudo, and store it in the
user's macOS Keychain.

Agents may run:

```bash
./install.sh --all --yes --json
```

The user only answers the hidden local password prompt. Later metadata cache
refreshes, DB decryption, image-key refreshes, and key re-scans use the stored
sudo credential unattended.

## Update

For an existing release install, prefer the first-class updater:

```bash
wechat-cli update
```

```powershell
wechat-cli update
```

`wechat-cli update` downloads the latest GitHub release zip, verifies sha256
when the checksum asset is present, then runs the bundled installer in update
mode. On macOS it waits for completion and returns the installer result inside
the normal CLI JSON envelope. On Windows it starts a background updater because
Windows cannot overwrite the running `.exe`; inspect `data.log` if verification
fails.

If an installed macOS binary immediately prints `zsh: killed` even for
`wechat-cli --help`, the existing CLI may not be able to self-update. Use the
release bootstrap command below. Current macOS installers copy/build into a
temporary file, ad-hoc codesign it, then atomically rename it into place so an
update does not in-place overwrite the running Mach-O.

For old installs that do not have `wechat-cli update` yet, run the release
bootstrap again:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r266-tech/wechat-cli](https://github.com/r266-tech/wechat-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
