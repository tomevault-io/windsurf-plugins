---
trigger: always_on
description: For agents (Cursor, Claude Code, Codex, Windsurf, and similar) to use Uplink **non-interactively**.
---

# Agent Integration Guide

For agents (Cursor, Claude Code, Codex, Windsurf, and similar) to use Uplink **non-interactively**.

Install: `npm install -g uplink-cli` or `npx uplink-cli …`  
Package name: `uplink-cli` · Binary: `uplink`

## Auth

- `uplink tunnel create` automatically creates guest access when no token exists. Guest access includes **1 active tunnel**, expiring after **24 hours**.
- Use `AGENTCLOUD_TOKEN` (bearer). Prefer stdin over argv:
  ```bash
  echo "$TOKEN" | uplink --token-stdin …
  ```
- Humans: `uplink login --email you@example.com` then `--code 123456`. This upgrades current guest access, preserves its tunnel, and unlocks persistent features. Credentials are saved to `~/.uplink/credentials` (chmod 600).
- API base: `--api-base https://api.uplink.spot` or `AGENTCLOUD_API_BASE`.

## Explicit guest token (optional)

```bash
uplink signup --json
uplink signup --label "cursor-agent" --expires-days 30 --json
```

Save `token` from the JSON — it is shown only once. Then:

```bash
export AGENTCLOUD_TOKEN='…'
```

Explicit signup creates guest access. A later email login upgrades that guest account when the current token is available.

## Machine-mode contract

| Rule | Detail |
|------|--------|
| `--json` | stdout = JSON only; logs/errors go to stderr |
| Exit `0` | success |
| Exit `2` | usage / bad args |
| Exit `10` | auth missing/invalid |
| Exit `20` | network |
| Exit `30` | server / unknown |

Guest accounts can share one local port and use public domain search. Hosting, databases, aliases, and custom domains require a verified email account. Gate error: `ACCOUNT_VERIFICATION_REQUIRED`.

Premium aliases may return `ALIAS_NOT_ENABLED` / `ALIAS_LIMIT_REACHED`.

Free hosting (new accounts): **1 app**, **100 MB** of live artifacts, **no custom domains**, app **sleeps after 30 minutes idle**. Errors: `HOST_APP_LIMIT_REACHED`, `HOST_STORAGE_LIMIT_REACHED`, `HOST_DOMAIN_NOT_ENABLED`. Check quota: the `hosting` object on `GET /v1/me`.

## Tunnels (share localhost)

`tunnel create` **creates the API record and starts the local client** so the public URL works. Use `--api-only` only if you will start the client yourself.

```bash
# Create + start client (optional alias if enabled)
echo "$TOKEN" | uplink --token-stdin \
  tunnel create --port 3000 --alias myapp --json

# List (includes connected status)
echo "$TOKEN" | uplink --token-stdin tunnel list --json

# Alias on an existing tunnel
echo "$TOKEN" | uplink --token-stdin tunnel alias-set --id tun_xxx --alias myapp --json
echo "$TOKEN" | uplink --token-stdin tunnel alias-delete --id tun_xxx --json

# Stats / stop
echo "$TOKEN" | uplink --token-stdin tunnel stats --id tun_xxx --json
echo "$TOKEN" | uplink --token-stdin tunnel stop --id tun_xxx --json
echo "$TOKEN" | uplink --token-stdin tunnel stop --all --json
```

JSON create shape (representative):

```json
{
  "tunnel": { "id": "tun_…", "url": "https://abc.x.uplink.spot", "token": "…", "status": "…" },
  "alias": "myapp",
  "aliasError": null,
  "url": "https://myapp.uplink.spot",
  "client": { "pid": 12345, "started": true }
}
```

`connected` on `tunnel list` means the local client is attached to the relay.

## Hosting

Use `--json`. For prompts, pass `--yes`.

```bash
echo "$TOKEN" | uplink --token-stdin host setup \
  --path /path/to/app --name myapp --env-file /path/to/.env \
  --wait-timeout 900 --wait-interval 5 --yes --json

echo "$TOKEN" | uplink --token-stdin host deploy \
  --path /path/to/app --name myapp --wait --json

echo "$TOKEN" | uplink --token-stdin host analyze --path /path/to/app --json
echo "$TOKEN" | uplink --token-stdin host preflight --path /path/to/app --json
echo "$TOKEN" | uplink --token-stdin host list --json
echo "$TOKEN" | uplink --token-stdin host status --id app_xxx --json
echo "$TOKEN" | uplink --token-stdin host logs --id app_xxx --json
echo "$TOKEN" | uplink --token-stdin host delete --id app_xxx --yes --json
```

Notes:
- Next.js server hosting expects `output: "standalone"`. Vite/CRA → static `dist`/`build`.
- Prefer a `.uplinkignore` (`node_modules`, `.next`, `dist`, `*.log`, local `.db`, …).
- `host logs` may return `NOT_READY` until a deployment is running.
- `host delete` requires `--yes` (or typing `DELETE` interactively).

## Custom domains

Uplink is a hub for domains and hosting spread across providers: registrar inventory
(GoDaddy, Cloudflare, Hostinger, Namecheap, DreamHost) and hosted domains from any
**cPanel** account (Namecheap shared, Bluehost, HostGator, …) land in one `domains list`.
Attach/verify is under `host domains`.
The bare `uplink domains` command opens Find a domain. Agents should use JSON:

```bash
uplink domains search acme --json
uplink domains search ~space --json
# `.inc` TLD browse is interactive (TUI): uplink domains search
uplink domains check example.com --json
uplink domains list --json
uplink domains providers connect godaddy --token-env GODADDY_PAT --json
uplink domains providers connect cloudflare --token-env CF_API_TOKEN --json
uplink domains providers connect hostinger --token-env HOSTINGER_API_TOKEN --json
uplink domains providers connect dreamhost --token-env DREAMHOST_API_KEY --json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firstprinciplecode/uplink](https://github.com/firstprinciplecode/uplink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
