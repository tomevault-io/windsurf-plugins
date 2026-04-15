---
trigger: always_on
description: A minimal CLI for deploying apps to a VPS with automatic preview environments.
---

# toss

A minimal CLI for deploying apps to a VPS with automatic preview environments.

## Philosophy

- No Docker
- No Kubernetes
- No complex configuration
- Single VPS
- Git-based workflow
- Convention over configuration
- Runtime-agnostic

## Installation

```bash
curl -fsSL https://toss.dev/install.sh | sh
```

## Commands

```bash
toss init                              # interactive setup wizard (local + VPS)
toss deploy <env>                      # deploy to environment (e.g., prod, pr-42)
toss deploy <env> -s KEY=VALUE         # deploy with per-environment secret override
toss rollback <env> [release]          # roll back to previous or specific release
toss remove <env>                      # remove an environment (prod cannot be removed)
toss list                              # list running deployments
toss status                            # status summary (config + deployments + overrides)
toss logs <env>                        # tail logs for environment
toss logs <env> -n 100                 # show last 100 lines
toss logs <env> --since "1h"           # logs since time (e.g., 1h, 2025-01-01)
toss logs <env> -n 100 --follow        # show last 100 lines, then follow
toss ssh <env>                         # SSH into server, cd to current release directory
toss secrets push <env> --file <file>  # push secrets (env: prod or preview)
toss secrets pull <env> [--file <file>]# pull secrets from VPS (defaults to .env.prod/.env.preview)
```

All commands require explicit arguments—no defaults. This keeps deployments predictable.

### Rollbacks

toss keeps old releases on the server for prod environments (default: 3 releases, configurable via `keepReleases`).

**Quick rollback** (instant, uses existing release on server):
```bash
toss rollback prod
# You will be prompted to select a release.
# Or specify a release directly:
toss rollback prod 20260130_100000
```

This swaps the `current` symlink and restarts the service. Quick rollback
works when multiple releases are available (prod keeps several by default;
previews typically keep only one).

**Manual rollback** (SSH, same effect as toss rollback):
```bash
ssh root@64.23.123.45
cd /srv/myapp/prod/releases
ls -la  # see available releases
ln -sfn /srv/myapp/prod/releases/20260130_100000 /srv/myapp/prod/current
systemctl restart toss-myapp-prod
```

**Full rollback** (deploys from your local repo):
```bash
git checkout abc123f
toss deploy prod
```

The quick rollback is instant but requires SSH access. The full rollback creates a new release from your local code.

## Config File

`toss.json` in repo root:

```json
{
  "app": "myapp",
  "server": "root@64.23.123.45",
  "domain": "example.com",
  "startCommand": "npm start",
  "deployScript": [
    "npm ci",
    "npm run build"
  ],
  "dependencies": {
    "nodejs": "curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs",
    "bun": "curl -fsSL https://bun.sh/install | bash"
  },
  "persistentDirs": ["./data", "uploads"],
  "keepReleases": 5
}
```

Fields:
- `app` (required): app name (used for directories and service names)
- `server` (required): SSH target (e.g., `root@64.23.123.45`, `root@64.23.123.45:2222`, or IPv6 `root@[::1]:2222`)
- `domain` (optional): public domain; if omitted, toss uses sslip.io URLs (e.g., `prod.myapp.64-23-123-45.sslip.io`)
- `startCommand` (required): command to start the app (used in systemd unit file)
- `deployScript` (required): array of commands run on every deploy
- `dependencies` (optional): named install commands that run once per server (toss tracks what was applied)
- `persistentDirs` (optional): array of directories that persist across releases (e.g., `data`, `uploads`) — legacy `preserve` is still accepted
- `keepReleases` (optional): number of old releases to keep for prod (default: 3); previews always keep only the current release

`app` and `domain` are independent; toss does not derive one from the other.
There is no local `.toss/` folder; all config lives in `toss.json`. Server state is stored in `/srv/<app>/.toss/`.

## Environment Naming Rules

- Lowercase letters, numbers, and hyphens only (`a-z`, `0-9`, `-`)
- Must start with a letter
- Max 63 characters (DNS label safe)
- `prod` is reserved for the prod environment
- `production` is not supported; use `prod`

## Directory Structure on Server

```
/srv/myapp/
├── .toss/
│   ├── state.json          # ports, applied dependencies, lock, origin
│   └── secrets/
│       ├── prod.env  # prod secrets (base)
│       ├── preview.env     # preview secrets (base, shared by all non-prod)
│       └── overrides/
│           ├── pr-42.env   # per-environment overrides (merged with base)
│           └── pr-123.env
├── prod/             # prod environment
│   ├── releases/           # timestamped release directories
│   │   ├── 20260130_100000/
│   │   ├── 20260130_120000/
│   │   └── 20260130_140000/
│   ├── current -> releases/20260130_140000  # symlink to active release
│   └── preserve/           # persistent directories across releases
│       ├── uploads/
│       └── data/
├── pr-42/                  # preview environment
│   ├── releases/
│   │   └── 20260130_150000/
│   ├── current -> releases/20260130_150000
│   └── preserve/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidkarolyi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
