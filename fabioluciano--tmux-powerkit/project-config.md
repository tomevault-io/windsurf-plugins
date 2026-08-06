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

**Status**: ✅ COMPLETE - All 51 plugins migrated to contract system

### Migration Statistics

- **Total Plugins**: 51
- **Migration Date**: January 2025
- **Architecture**: Contract-based plugin system with strict separation of concerns
- **Lines of Code**: ~5,500 lines (plugins only)

### Migrated Plugins (Alphabetical)

1. **aiquotas** - AI provider quotas, usage, balances, and rate limits
2. **appearance** - macOS light/dark/auto toggle with theme switching
3. **audiodevices** - Audio output device (macOS, SwitchAudioSource)
4. **battery** - Battery level with charge state (pmset/upower)
5. **bitbucket** - Pull requests count (API)
6. **bitwarden** - Vault lock status (bw CLI)
7. **bluetooth** - BT status + connected devices (blueutil/bluetoothctl)
8. **brightness** - Screen brightness (Linux only - sysfs/brightnessctl/light/xbacklight)
9. **camera** - Camera usage indicator (macOS, lsof)
10. **chezmoi** - Pending dotfile changes (chezmoi CLI)
11. **cloud** - Cloud provider profile (AWS/Azure/GCP)
12. **cloudstatus** - Service status monitoring (status APIs)
13. **connectivity** - Internet connectivity status (curl)
14. **cpu** - CPU usage with thresholds (sysctl/top)
15. **crypto** - Cryptocurrency prices (CoinGecko API)
16. **datetime** - Date/time with 15 format presets
17. **disk** - Disk usage with thresholds (df)
18. **docker** - Docker or Podman container status
19. **external_ip** - Public IP address (ipify API)
20. **fan** - Fan speed (macOS: osx-cpu-temp/iStats, Linux: hwmon/dell_smm/thinkpad)
21. **git** - Branch + modified files status
22. **github** - Notifications/PRs/issues (gh CLI)
23. **gitlab** - Merge requests/todos (glab CLI)
24. **gpu** - GPU usage (NVIDIA: nvidia-smi, AMD: sysfs, Intel: frequency-based, macOS: powerkit-gpu)
25. **hostname** - System hostname
26. **iops** - Disk I/O operations (iostat)
27. **jira** - Assigned issues count (API)
28. **kubernetes** - Context + namespace (kubectl)
29. **loadavg** - Load average with cores (uptime)
30. **memory** - Memory usage with thresholds (vm_stat/free)
31. **microphone** - Mic mute status (macOS, osascript)
32. **netspeed** - Upload/download speed (ifstat/netstat)
33. **nowplaying** - Current music track (Music/Spotify)
34. **packages** - Pending updates (brew/apt/yum/pacman)
35. **ping** - Network latency with thresholds
36. **pomodoro** - Timer with work/break phases
37. **smartkey** - Custom environment variable display
38. **ssh** - SSH session indicator
39. **stocks** - Stock prices (Yahoo Finance API)
40. **swap** - Swap memory usage (sysctl/vm_stat/proc)
41. **sysstatus** - Aggregated system health badge
42. **temperature** - CPU temperature (macOS, osx-cpu-temp)
43. **terraform** - Workspace indicator
44. **timezones** - Multi-timezone display
45. **topproc** - Process consuming the most CPU
46. **uptime** - System uptime
47. **volume** - System volume (macOS, osascript)
48. **vpn** - VPN connection status (tun/tap interfaces)
49. **weather** - Weather from wttr.in
50. **wifi** - WiFi SSID + signal strength
51. **yadm** - yadm dotfile repository status (modified, untracked, ahead/behind)

### Plugin Categories

- **System Monitoring** (15): battery, cpu, memory, swap, disk, loadavg, uptime, temperature, fan, gpu, iops, hostname, topproc, sysstatus, volume
- **Network** (8): netspeed, wifi, vpn, ping, external_ip, ssh, weather, connectivity
- **Media** (6): nowplaying, audiodevices, camera, microphone, bluetooth, brightness
- **Development** (14): aiquotas, git, github, gitlab, bitbucket, jira, kubernetes, terraform, cloud, cloudstatus, packages, chezmoi, yadm, docker
- **Productivity** (6): datetime, timezones, pomodoro, smartkey, bitwarden, appearance
- **Financial** (2): crypto, stocks

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabioluciano/tmux-powerkit](https://github.com/fabioluciano/tmux-powerkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
