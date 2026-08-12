---
trigger: always_on
description: This is the single contributor and coding-agent guide for **sysinfo-cli**. Keep it concise, practical, and aligned with the current Bash codebase.
---

# Repository Guidelines

This is the single contributor and coding-agent guide for **sysinfo-cli**. Keep it concise, practical, and aligned with the current Bash codebase.

For project context and philosophy, see `README.md` / `README_zh.md`; for release notes, see `CHANGELOG.md`.

## What This Project Is

**sysinfo-cli** is a lightweight **Bash** system-status dashboard for **Linux** SSH login and live terminal monitoring.

Core capabilities:

- SSH login banner via `/etc/profile.d` and zsh profile hooks.
- Real-time `sysinfo` monitor without `watch` respawning.
- Traffic accounting with monthly quotas.
- Linux `tc` throttling when quota thresholds are reached.
- NAT display.
- Bark push notifications and rule engine.
- Bilingual English / Chinese UI.

Target distros include Debian, Ubuntu, RHEL, Fedora, Alpine, Arch, openSUSE, Gentoo, OpenWrt, and similar Linux systems. macOS is useful for editing, but it is not a supported runtime target.

## Project Structure & Module Organization

```text
sysinfo-cli/
├── src/                       # Source of truth; edit here, not installed copies
│   ├── sysinfo.sh             # CLI entry: args, -c/-r config apply, live loop
│   ├── sysinfo_core.sh        # Engine: metrics, traffic, tc throttling, rendering
│   ├── sysinfo_notify.sh      # Bark push notifications + rule engine
│   ├── sysinfo_banner.sh      # SSH login banner (bash /etc/profile.d)
│   └── sysinfo_banner_shim.sh
├── scripts/                   # Development aids (docker-cmd.sh, test_throttle.sh)
├── tests/                     # test_sysinfo.sh, server_validate.sh, docker_distros.sh
├── docs/                      # Static site (index.html, wiki.html, wiki.css/js, assets/)
├── docker/                    # Per-distro Dockerfiles and build helpers
├── install.sh / uninstall.sh  # One-shot installer and remover
├── config.yaml.example
├── Dockerfile
├── Makefile
└── AGENTS.md                  # This file
```

Do not commit local scratch files such as `CODEBUDDY.md` or generated test artifacts under `tests/*.log`.

## Runtime & Installed Paths

Runtime config lives in `/etc/sysinfo/config.yaml`; UI language lives in `/etc/sysinfo-lang`.

| Installed artifact | Path |
|-------------------|------|
| CLI wrapper | `/usr/local/bin/sysinfo` |
| Main config | `/etc/sysinfo/config.yaml` |
| UI language marker | `/etc/sysinfo-lang` |
| Traffic/throttle state | `/etc/sysinfo-traffic.json` |
| Flat traffic config | `/etc/sysinfo-traffic` |
| NAT mappings | `/etc/sysinfo-nat` |
| Login banner | `/usr/local/lib/sysinfo/sysinfo_banner.sh` |

## Build, Test, and Development Commands

All common commands are dispatched through the `Makefile`.

| Task | Command | Notes |
|------|---------|-------|
| Show all targets | `make` | Help with per-target descriptions |
| Run one-shot dashboard | `make run` | Uses `RUN_TIMEOUT`, default 5s |
| Run live interactive monitor | `make run-live` | Ctrl+C to exit |
| CLI help | `make help-cli` | Runs `src/sysinfo.sh -h` |
| Bash syntax check | `make syntax` | Covers `src/*.sh`, installer, scripts, tests |
| Unit tests | `make test` | Runs `tests/test_sysinfo.sh` |
| Full validation | `make validate` | Runs `tests/server_validate.sh` |
| Install (keep config) | `sudo ./install.sh` | Linux target |
| Install + reset config | `sudo ./install.sh --overwrite-config` | Backs up old config |
| Reload system config | `sudo sysinfo -r` | Applies `/etc/sysinfo/config.yaml` |
| Throttle diagnostic | `bash scripts/test_throttle.sh` | Manual diagnostic |
| Notification test | `sysinfo --notify-test` | Requires configured Bark key |
| Notification rule check | `sysinfo --notify-check` | Cron-friendly |
| HTML docs preview | `make docs-serve` | Serves on port 8099 and auto-frees if busy |
| Single-distro Docker test | `make docker-test` | Docker required |
| Multi-distro smoke test | `make docker-test-distros` | Uses `scripts/docker-cmd.sh` |
| Build per-distro images | `make docker-build-distros` | Includes OpenWrt |
| Deploy locally | `make deploy` | test -> validate -> install |
| Ship | `make push` | Pushes current branch |

## Architecture Overview

`sysinfo.sh` is the CLI entrypoint. It sources `sysinfo_core.sh` for rendering, metrics, traffic state, and throttling. It optionally sources `sysinfo_notify.sh` for Bark notifications.

Configuration is YAML-only and parsed with mikefarah `yq`, preferring `/usr/local/bin/yq`. The installer downloads the correct yq binary per `uname -m`.

`-c` and `-r` flatten YAML into plain state files under `/etc/`. The core reads those state files with simple `grep` extraction rather than a JSON parser. During config application, `get_applied_config()` reads only from the file being applied so `-c` / `-r` cannot silently fall back to `~/.config`.

Throttling uses Linux `tc`: HTB + fq_codel for upload, IFB for download. Gateway hosts are skipped unless `network.force_gateway_throttle: true`; throttle rates are floored at 64 kbit.

Notifications are edge-triggered with cooldown. State is kept in `/var/tmp/sysinfo-notify-state-<user>`.

## Install Script Notes

Important installer functions:

- `require_linux()` rejects unsupported non-Linux runtime installs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baixiaoshengofficial/sysinfo-cli](https://github.com/baixiaoshengofficial/sysinfo-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
