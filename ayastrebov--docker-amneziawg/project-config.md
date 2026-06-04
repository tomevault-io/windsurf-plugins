---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Summary

Docker container for AmneziaWG VPN built on LinuxServer.io base images with s6-overlay process supervision. Two modes: **server** (auto-generates configs when `PEERS` is set) and **client** (uses manual configs from `/config/wg_confs/`). The container brings up ALL `.conf` files in `/config/wg_confs/` on startup.

## Build & Test

```bash
# Build image locally
docker build -t amneziawg-test .

# Run server mode smoke test (tunnel won't work without /dev/net/tun — expected)
docker run -d --name awg-test --cap-add NET_ADMIN \
  -e PEERS=2 -e SERVERURL=test.example.com \
  -v /tmp/awg-test:/config amneziawg-test

# Verify config generation
docker logs awg-test
docker exec awg-test cat /config/wg_confs/wg0.conf
docker exec awg-test cat /config/peer1/peer1.conf
docker exec awg-test /app/show-peer 1

# Cleanup
docker rm -f awg-test && rm -rf /tmp/awg-test
```

There is no automated test suite. CI runs smoke tests on PRs: binary presence, s6 structure, show-peer executable check.

## Architecture

### Dockerfile: 3-stage multi-arch build

| Stage | Base | Output |
|---|---|---|
| `go-builder` | `golang:1.24.4-alpine` | `/src/amneziawg-go` (static binary, CGO) |
| `tools-builder` | `alpine:3.21` | `/usr/bin/awg` (compiled C) + `/usr/bin/awg-quick` (bash script copied from `src/wg-quick/linux.bash`) |
| runtime | `ghcr.io/linuxserver/baseimage-alpine:3.21` | Production image |

Runtime creates compatibility symlinks: `wg → awg`, `wg-quick → awg-quick`, `/etc/wireguard → /config/wg_confs`.

### s6-overlay service chain

```
init-config (LSIO) → init-amneziawg-module (oneshot) → init-amneziawg-confs (oneshot) → svc-coredns (longrun) → svc-amneziawg (oneshot)
```

- **init-amneziawg-module**: Tests kernel support via `ip link add dev test type wireguard`. Falls back to `amneziawg-go` userspace (exports `WG_QUICK_USERSPACE_IMPLEMENTATION`).
- **init-amneziawg-confs**: Config generation using eval+heredoc template expansion from `/config/templates/`. Server mode generates keys, wg0.conf, peer configs, QR codes. Client mode disables CoreDNS.
- **svc-coredns**: Longrun CoreDNS service with `notification-fd 3` health checks. Auto-disabled if port 53 already bound or `USE_COREDNS=false`.
- **svc-amneziawg**: Oneshot service (up/down scripts). Validates `[Interface]` in each .conf, activates tunnels, saves active confs to `/run/activeconfs` via `declare -p`. Finish script tears down in reverse order.

Dependencies are declared via empty files in `dependencies.d/`. Services are registered via empty files in `user/contents.d/`.

### Config persistence

All env vars are saved to `/config/.donoteditthisfile` (LinuxServer pattern) for change detection on restart. AWG obfuscation params are additionally saved to `/config/server/awg_params` and loaded as fallback (via `grep`/`cut`, NOT `source` — to preserve env var priority). Configs only regenerate if any saved var differs from the current value.

## Key Development Patterns

### s6-overlay scripts
- Shebang: `#!/usr/bin/with-contenv bash`
- Add `# shellcheck shell=bash` directive
- Must be `chmod +x`
- Use `lsiown -R abc:abc /config` for ownership (LinuxServer helper), fallback to `chown`

### Adding a new environment variable
1. Set default in `init-amneziawg-confs/run` main logic section
2. If persistent: add to `save_vars()` (as `ORIG_X`) AND the change detection `if` block
3. For AWG params: also add to `generate_awg_params()` save block AND `load_awg_params()` grep section
4. For config output: add to templates in `root/defaults/` (eval+heredoc expanded), `append_awg_signatures()` (server conf — appends after template, before peer blocks), or `append_awg_signatures_to_interface()` (peer confs — inserts before `[Peer]` using awk)
5. Document in `docker-compose.yml` (commented example) and `README.md`

### AWG obfuscation parameters
All clients and server must use identical values. Key constraints:
- `AWG_VERSION`: `"2.0"` (default, S3/S4 random, I1 randomly generated as QUIC Initial/DNS/DTLS) or `"1.5"` (S3=S4=0, no I1-I5)
- `Jmin < Jmax`, `Jmax ≤ 1280`
- `S1 ≤ 1132`, `S2 ≤ 1188`, `S1+56 ≠ S2`, `S3 ≤ 64`, `S4 ≤ 32` (S4 is per-packet overhead — keep small)
- `H1-H4` must be unique, all ≥ 5 (values 1-4 are standard WireGuard headers). **AWG 2.0 generates non-overlapping quadrant range pairs by default** (e.g., `H1=90666522-140666522`) — the Amnezia app uses range format to identify AWG 2.0; single integers cause it to report AWG 1.5. AWG 1.5 keeps single integers.
- `I1-I5` (AWG 2.0 signatures) use tag syntax with `=` signs — parse with `cut -d= -f2-` not `-f2`
- Detailed parameter reference: `.claude/skills/docker-amneziawg/references/awg-parameters.md`

## Conventions

- Commit messages: conventional commits (`feat:`, `fix:`, `docs:`, `chore:`)
- Branch naming: `feature/your-feature-name`
- Indentation: 4 spaces for shell scripts and s6-overlay files, 2 spaces for Dockerfile and YAML (see `.editorconfig`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AYastrebov/docker-amneziawg](https://github.com/AYastrebov/docker-amneziawg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
