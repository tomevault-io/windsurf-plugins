---
trigger: always_on
description: This guide is **mainly for an agent operating a MoaV server** on the owner's
---

# AGENTS.md — running & operating a MoaV server with an AI agent

This guide is **mainly for an agent operating a MoaV server** on the owner's
behalf — installing it, adding users, checking health, reading logs, handing the
owner their dashboards. If you are instead **editing the code** (fixing a bug,
adding a protocol), jump to [Developing MoaV](#developing-moav) and the dev docs.

Humans: start with [README.md](README.md); public docs are at
[moav.sh/docs](https://moav.sh/docs). A one-fetch index for agents is
[llms.txt](llms.txt).

> **Safety first.** Set the server up over an **SSH key**, never a password, and
> never paste the owner's `ADMIN_PASSWORD`, private keys, or a user's share-link
> into anything public (issues, PRs, pastebins). Treat everything under `state/`
> and `outputs/` as secret. Don't run destructive commands (`uninstall`,
> `user revoke`, `docker volume rm`) without the owner's explicit go-ahead.

## What MoaV is

A single-host, Docker-Compose, multi-protocol Internet-censorship-circumvention
stack. `moav.sh` is a bash dispatcher over `lib/*.sh` modules; it bootstraps
keys/certs, generates per-user bundles (configs, QR codes, a V2Ray
subscription), and runs 16+ transports plus an optional Prometheus/Grafana
monitoring stack. Protocol servers run as containers; provisioning and the CLI
are bash. The install lives in **`/opt/moav`** by default.

## Install & run

Install the global `moav` command, then run the guided setup (asks for domain,
ACME email, admin password, then bootstraps keys/certs and starts the stack):

```bash
curl -fsSL https://moav.sh/install.sh | bash    # or: git clone … && ./moav.sh install
moav                                             # guided config + bootstrap + start
```

The `moav.sh/install.sh` one-liner installs the latest **stable** release. To
fresh-install the **latest dev** code instead (e.g. to test an unreleased fix),
fetch and run `install.sh` from the branch and tell it to clone that branch:

```bash
curl -fsSL https://raw.githubusercontent.com/MotherofallVPNs/MoaV/dev/install.sh | bash -s -- -b dev
```

(Swap `dev` for a tag like `v2.0.0-rc.3` to install a specific release candidate.)

**Upgrade in place:** `moav update -b main && moav build && moav start`
(see [docs/V2-MIGRATION.md](docs/V2-MIGRATION.md) for the 1.9.x → v2 path).

## Everyday operations

Run these as the owner's operator. Most take an optional service/profile name;
with none they act on the whole stack.

| Command | What it does |
|---|---|
| `moav` | Interactive menu (safe to explore; nothing changes until you pick an action) |
| `moav status` | Per-container health and which profiles are up |
| `moav start [svc\|profile]` / `moav stop [svc]` / `moav restart [svc]` | Bring services up/down. `moav start all` = everything |
| `moav user list` | List provisioned users |
| `moav user add <name> [--package]` | Create a user (keys, configs, QR). `--package` also builds the downloadable zip |
| `moav user add --batch N [--prefix NAME] [--package]` | Bulk-create N users |
| `moav user package <name>` | (Re)build one user's downloadable **zip bundle** in `outputs/` |
| `moav user sub <name>` | Print that user's base64 **subscription** — the blob phone apps (Streisand, v2rayNG, NekoBox, Hiddify) import |
| `moav user base64 <name>` | Print that user's text-only bundle **zipped, then base64** — input for moav-client's e2e `bundle_b64`. **Not** a subscription; client apps reject it |
| `moav user revoke <name>` | **Destructive** — remove a user and their access. Confirm with the owner first |
| `moav regenerate-users` | Rebuild every bundle from stored state (keys unchanged); users just re-download |
| `moav logs [svc]` | Tail container logs — **first stop for debugging** |
| `moav doctor` | Diagnostics: DNS, ports, certs, service health. Run this when something's off |
| `moav test <name>` | End-to-end protocol validation for a user (proves the tunnels actually pass traffic) |
| `moav cert` | Certificate status / renewal (Let's Encrypt) |
| `moav admin password` | Generate/reset the admin **and** Grafana password (`ADMIN_PASSWORD`) |
| `moav export` / `moav import` | Full backup / restore (keys, users, configs) — used for server migration |
| `moav update` | Pull new code (follow with `moav build && moav start`) |

Full reference: [moav.sh/docs/CLI](https://moav.sh/docs/CLI/). Debugging flow when
a user reports "it stopped working": `moav status` → `moav doctor` →
`moav logs <svc>` → `moav test <user>`.

## Dashboards & access — what to hand the owner

Both dashboards are HTTPS on their own ports and share the single
`ADMIN_PASSWORD` from `.env`. The URLs are derived from `DOMAIN` (falling back to
`SERVER_IP`); the CLI prints them after bootstrap, and you can reproduce them:

| Dashboard | URL | Login | Notes |
|---|---|---|---|
| **Admin dashboard** (FastAPI) | `https://<domain>:9443` (`PORT_ADMIN`) | HTTP Basic — **any username**, password = `ADMIN_PASSWORD` | Add/revoke users, download bundles, live stats. **Self-signed cert** (expect a browser warning). Optional `ADMIN_IP_WHITELIST` (comma-separated IPs/CIDRs) restricts access; empty = open. Locks itself (503) if `ADMIN_PASSWORD` is unset/insecure |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shayanb/MoaV](https://github.com/shayanb/MoaV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
