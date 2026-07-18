---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`rathole-manager` is a multi-location reverse-tunnel system built on **rathole + Nginx**. A single Iran server (behind one domain, one cert, one port 443) fronts many foreign "nodes" that connect back via reverse tunnel. User traffic is routed to nodes **by URL path** (`map $uri $backend_port` in nginx). The primary goal is censorship-resistant tunneling for Iran; almost all code comments and log strings are Persian transliterated into Latin ("Finglish", e.g. `tvlid khodkar` = auto-generate). Match that style when editing.

The repo root holds packaging/bootstrap scripts; the actual product lives in `rathole-manager/`.

## Three roles, three programs

The system has three distinct runtime roles. Read the file for the role you're touching:

- **Iran panel** → `rathole-manager/ratholectl` (bash). Runs the rathole **server** + nginx. Owns node inventory. Generates `/etc/rathole/server.toml` and `/etc/nginx/conf.d/rathole.conf`.
- **Foreign node** → `rathole-manager/ratholenode` (bash). Runs the rathole **client**. Generates `/etc/rathole/client.toml`.
- **Hub** → `rathole-manager/ratholehub/hub.py` (Python 3, **stdlib only, no pip**). Central web panel that drives many Iran servers + nodes over SSH. Listens on `127.0.0.1` (fronted by nginx under `/hub/`).

`rathole-manager/common.sh` is sourced by both bash tools (colors/logging, `kcp_profile`, `install_kcptun`, `apply_sysctl_tuning`, `fakeweb_service`).

## Central design principle: state → regenerate → hot-reload

Every mutation follows the same pattern — **never hand-edit the generated configs**; change state and regenerate:

- **ratholectl**: state is `/etc/rathole-manager/state.json` (jq-manipulated via `state_set`/`s_get`). Commands mutate state, then call `regenerate()` → `gen_server_toml()` + `gen_nginx_conf()` → `nginx -t` → reload. Configs are written **in place (preserving inode)** so rathole's `config_watcher` hot-reloads without dropping active tunnels. `regenerate` keeps a `.rathole-good.bak` and auto-reverts nginx if `nginx -t` fails.
- **ratholenode**: state is `/etc/rathole/node.env` (key=value, via `env_set`/`load_env`) + `/etc/rathole/services.conf` (`name|token|inbound` lines). `gen_client()` builds `client.toml`; `reload_svc` prefers hot-reload over restart (`restart_svc` only for transport changes like kcp on/off).

## Transport modes (the core complexity)

The same tunnel can carry traffic four ways; switching modes never changes user-facing services/tokens/paths — only the transport. Key invariant: **TLS is terminated only by nginx** — rathole server transport is always `tls = false`; the default client uses `tls = true` over websocket to nginx/443.

- **websocket + TLS** (default): client connects `wss://domain:443`, nginx splits root `/` between the fake site and the rathole control channel using `$http_upgrade` (rathole always uses `/` for control; path isn't configurable in rathole).
- **kcp** (`cmd_kcp` both sides): parallel UDP+FEC path via kcptun for lossy links (TCP-over-TCP mitigation). Additive — doesn't touch server/nginx/443. Profiles (`balanced`/`lossy`/`aggressive`) must match on both ends; defined in `common.sh:kcp_profile`. Multi-Iran nodes run independent kcp per upstream (`rathole-kcp-up-<id>`, local ports from 29901).
- **plain** (`cmd_plain` both sides): no-TLS websocket to a separate HTTP listener port (default 8880). Lighter, unencrypted tunnel path.
- **game / SNI** (`ratholectl game`, `gen_stream_conf`): when any node has an `sni`, port 443 switches to nginx **stream/SNI** mode (L4 passthrough) and the L7 path/WS vhost moves to an internal port (`internal_port`, default 8443). TLS for game traffic terminates on the **node** (real cert, VLESS+TLS+Vision). This is why `gen_nginx_conf` branches on `sni_count`.

## Path == node name == nginx map == Xray inbound

A node's `name` is simultaneously its URL path, its nginx `map` entry, and the Xray inbound path on the node. These three must stay identical. Each node has a data service; adding `--api-port` also creates a `<name>_api` service (bound to `127.0.0.1`) for panel↔node management over the tunnel.

## Hub (hub.py) specifics

- Single-file stdlib HTTP server; all UI (HTML/JS/CSS) and i18n (fa/en dicts) are inline. `Handler` is the router; `main()` serves `ThreadingHTTPServer`.
- **Security-critical:** it never runs raw strings on servers. `build_iran_cmd` / `build_node_cmd` map an `action` + validated args (via the `RE_*` regexes) to an **argv list**, executed over SSH with each arg passed separately (`run_on_server` → `_ssh_base`). When adding a server action, add it to the right `build_*_cmd` **and** the allow-list of actions, and validate every arg with a regex — do not interpolate user input into a shell string.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loopy-iri/RatholeEngine](https://github.com/loopy-iri/RatholeEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
