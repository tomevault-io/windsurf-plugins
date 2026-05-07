---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code)
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code)
when working with code in this repository.

## Project

heinzel — Administration of Linux servers, FreeBSD
servers, and macOS machines via SSH or locally.
Supports any Linux distribution (Debian, Ubuntu,
RHEL, CentOS, Fedora, SUSE, and others), FreeBSD,
and macOS.

## How It Works

The user provides a server hostname and optionally a
user. SSH key-based auth is used (no
password/passphrase needed). All work on remote
machines happens over SSH.

### Local mode

When the target is `localhost`, the user's own
hostname, or otherwise clearly the local machine,
heinzel operates in **local mode**:

- **No SSH.** Commands run directly in the shell.
- **No user prompt.** Use the current OS user.
- **Sudo still applies.** Probe `sudo -n true` as
  usual. If sudo is unavailable, enter unprivileged
  mode (no root SSH fallback).
- Skip all remote-only steps: blacklist/read-only
  checks, DNS alias detection, SSH user lookup,
  root SSH fallback.

### Remote mode (SSH)

- **Default:** `ssh root@hostname` — only when root
  privileges are actually needed.
- **Normal user:** `ssh user@hostname` — when the
  user specifies a non-root account or when root is
  not required.
- **sudo:** When logged in as a normal user, use
  `sudo` for commands that require elevated
  privileges.
- **Unprivileged mode:** When neither `sudo` nor
  root SSH is available, do everything possible as
  the current user and produce a sysadmin report.

**Always use the least amount of privileges needed.**

**SSH as root is not a risky action that requires
confirmation.** The privilege principle applies to
*commands*, not to the SSH login itself.

**Before any remote command, follow
`rules/first-connection.md`.** No exceptions — not
for `df -h`, not for `uptime`, not for anything the
user frames as quick.

### SSH Options

Always use these options on every SSH and
SCP/rsync-over-SSH command:

    ssh -o BatchMode=yes -o ConnectTimeout=5 …

## Access Control (Blacklist & Read-Only)

Read `rules/access-control.md` for full details.
Check blacklist first, then read-only list, on every
remote connection before any other work.

## Critical Safety Rules

- **Never fabricate server facts.** Do not guess or
  make up hosting providers, data centers, hardware
  specs, network topology, or any other detail you
  have not directly observed or been told. If you
  don't know, say so.
- **You are working on live production servers.**
- **Always detect the OS first** before doing any
  work.
- **Ask before:** reboots, firewall changes, service
  restarts, any destructive command. **Reloads**
  (`systemctl reload`) auto-proceed by default when
  a config test passes — see
  `rules/service-reload.md` for the full policy and
  the opt-out / opt-in config in
  `memory/service-policy.md`.
- **Absolute taboos (never run without explicit user
  request):** any command that modifies the partition
  table. Read-only partition inspection (e.g.
  `lsblk`, `fdisk -l`, `gpart show`,
  `diskutil list`) is always allowed. Never modify
  `/etc/ssh/sshd_config`. Never delete or overwrite
  SSH keys. Never halt or power off a server.
- **Firewall & network:** Be extremely careful — a
  mistake cuts off SSH access. Discuss with the user
  first.
- **Never remove or block SSH port 22.** If the user
  asks, explain the risk and refuse. Offer
  alternatives (e.g. restricting to specific IPs).
- **Verify the default incoming policy is
  deny/drop.** See `rules/<family>.md`.
- **Use the appropriate non-interactive package
  manager** for the detected OS (`apt-get`,
  `dnf`, `yum`, `zypper`, `pkg`, `brew` — never
  with `sudo` on macOS).
- **Prefer stable/official repos only.**
- **Stick to stable release tracks.**
- **Test before applying.** Use dry-run/test modes
  when available.

## No Shortcuts for "Quick" Questions

Every remote connection runs the full onboarding
pipeline in `rules/first-connection.md` —
blacklist/read-only check, DNS alias detection, SSH
user lookup, OS detection, server memory file, and
activity check — **before** any user-requested
command, including trivial ones like `df -h`,
`uptime`, or `uname -a`.

There is no "quick question" exception. Do not skip
steps because the request seems small, because you
already know the server, or because the user
appears to want a fast answer. If the pipeline
reveals nothing new, the overhead is a few extra
commands — acceptable. Silently skipping the
pipeline is a bug, not an optimization.

If following the pipeline will visibly slow the
answer, say so up front ("first-contact onboarding
on this host — one moment") rather than skipping.

## Verify Before Running

Do not trust your training data for command syntax.
Before running any command on a server, verify it:

1. **Check `--help` first.** Run `command --help`
   or `command -h` to confirm flags and syntax
   exist on this specific version.
2. **Read the man page** when `--help` is
   insufficient — especially for complex tools
   like `iptables`, `firewall-cmd`, `certbot`.
3. **Search upstream docs** (official project docs,
   distro wiki) when behavior varies across
   versions or distros.
4. **Check the rule file** — use the exact syntax
   from the loaded `rules/<family>.md` file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wintermeyer/heinzel](https://github.com/wintermeyer/heinzel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
