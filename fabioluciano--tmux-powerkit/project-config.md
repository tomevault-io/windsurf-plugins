---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Commit Rules

**CRITICAL**: When creating git commits for this repository:

1. **NEVER use `Co-Authored-By`** - Do not add any co-author lines to commits
2. **NEVER use emoji in commit messages** - Keep messages plain text
3. Use conventional commit format: `type(scope): description`
4. Valid types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

Example commit message:

```text
feat(defaults): change default theme to catppuccin/mocha

Catppuccin has surpassed Tokyo Night in popularity metrics.
Mocha is the most popular variant with excellent contrast.
```

## Migration Summary

**Status**: ✅ COMPLETE - All 46 plugins migrated to contract system

### Migration Statistics

- **Total Plugins**: 46
- **Migration Date**: January 2025
- **Architecture**: Contract-based plugin system with strict separation of concerns
- **Lines of Code**: ~5,500 lines (plugins only)

### Migrated Plugins (Alphabetical)

1. **appearance** - macOS light/dark/auto toggle with theme switching
2. **audiodevices** - Audio output device (macOS, SwitchAudioSource)
3. **battery** - Battery level with charge state (pmset/upower)
4. **bitbucket** - Pull requests count (API)
5. **bitwarden** - Vault lock status (bw CLI)
6. **bluetooth** - BT status + connected devices (blueutil/bluetoothctl)
7. **brightness** - Screen brightness (Linux only - sysfs/brightnessctl/light/xbacklight)
8. **camera** - Camera usage indicator (macOS, lsof)
9. **chezmoi** - Pending dotfile changes (chezmoi CLI)
10. **cloud** - Cloud provider profile (AWS/Azure/GCP)
11. **cloudstatus** - Service status monitoring (status APIs)
12. **cpu** - CPU usage with thresholds (sysctl/top)
13. **crypto** - Cryptocurrency prices (CoinGecko API)
14. **datetime** - Date/time with 15 format presets
15. **disk** - Disk usage with thresholds (df)
16. **external_ip** - Public IP address (ipify API)
17. **fan** - Fan speed (macOS: osx-cpu-temp/iStats, Linux: hwmon/dell_smm/thinkpad)
18. **git** - Branch + modified files status
19. **github** - Notifications/PRs/issues (gh CLI)
20. **gitlab** - Merge requests/todos (glab CLI)
21. **gpu** - GPU usage (NVIDIA: nvidia-smi, AMD: sysfs, Intel: frequency-based, macOS: powerkit-gpu)
22. **hostname** - System hostname
23. **iops** - Disk I/O operations (iostat)
24. **jira** - Assigned issues count (API)
25. **kubernetes** - Context + namespace (kubectl)
26. **loadavg** - Load average with cores (uptime)
27. **memory** - Memory usage with thresholds (vm_stat/free)
28. **microphone** - Mic mute status (macOS, osascript)
29. **netspeed** - Upload/download speed (ifstat/netstat)
30. **nowplaying** - Current music track (Music/Spotify)
31. **packages** - Pending updates (brew/apt/yum/pacman)
32. **ping** - Network latency with thresholds
33. **pomodoro** - Timer with work/break phases
34. **smartkey** - Custom environment variable display
35. **ssh** - SSH session indicator
36. **stocks** - Stock prices (Yahoo Finance API)
37. **swap** - Swap memory usage (sysctl/vm_stat/proc)
38. **temperature** - CPU temperature (macOS, osx-cpu-temp)
39. **terraform** - Workspace indicator
40. **timezones** - Multi-timezone display
41. **uptime** - System uptime
42. **volume** - System volume (macOS, osascript)
43. **vpn** - VPN connection status (tun/tap interfaces)
44. **weather** - Weather from wttr.in
45. **wifi** - WiFi SSID + signal strength
46. **yadm** - yadm dotfile repository status (modified, untracked, ahead/behind)

### Plugin Categories

- **System Monitoring** (13): battery, cpu, disk, fan, gpu, iops, loadavg, memory, swap, temperature, uptime, volume, brightness
- **Network** (7): external_ip, netspeed, ping, vpn, weather, wifi, ssh
- **Development** (10): git, github, gitlab, bitbucket, jira, kubernetes, terraform, cloud, chezmoi, yadm
- **Media** (4): nowplaying, audiodevices, camera, microphone
- **Productivity** (6): datetime, timezones, pomodoro, smartkey, bitwarden, appearance
- **Financial** (2): crypto, stocks
- **Services** (4): cloudstatus, packages, bluetooth, hostname

### Platform-Specific Plugins

- **macOS only**: volume, temperature, camera, microphone, audiodevices
- **Linux only**: brightness
- **Cross-platform**: All other plugins (including fan and gpu with platform-specific backends)

---

## Project Overview

PowerKit is a contract-based tmux status bar framework with strict separation of concerns:

- **Core**: Orchestration, lifecycle, cache, options
- **Plugin**: Data + semantics ONLY (no UI)
- **Renderer**: ALL UI decisions (colors, icons, formatting)
- **Theme**: Color definitions ONLY

**Target**: Bash 5.0+ (uses `$EPOCHSECONDS`/`$EPOCHREALTIME`) | **Architecture**: Contract-based plugin system

## Directory Structure

```text
tmux-powerkit/
├── tmux-powerkit.tmux              # TPM entry point
├── bin/
│   ├── powerkit-render             # Status-right renderer entry point
│   ├── powerkit-plugin             # Single plugin executor
│   ├── powerkit-icon               # Icon resolver
│   └── powerkit-binary-prompt      # Interactive macOS binary download prompt
├── src/
│   ├── core/                       # Core Framework

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabioluciano/tmux-powerkit](https://github.com/fabioluciano/tmux-powerkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
