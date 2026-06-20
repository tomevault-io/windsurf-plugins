---
trigger: always_on
description: Deploy a Phantun (UDP→fake-TCP obfuscator) + WireGuard tunnel between a home router (OpenWrt or MikroTik RouterOS 7+ with a container runtime) and a Debian/Ubuntu VPS. Use when the user asks to set up such a tunnel, when their ISP throttles/blocks UDP, when WireGuard handshake is unstable due to QoS, when they want to disguise a self-hosted WG egress as TCP, or when they want to deploy multiple parallel phantun tunnels to different VPSs. Bridges the WG endpoint into daed/OpenClash/Mihomo/sing-bo
---


# Phantun + WireGuard Tunnel: Home Router ⇄ Linux VPS

A battle-tested deployment guide covering the **non-obvious gotchas** that consume hours when followed naively from upstream READMEs. Covers two router platforms:

- **OpenWrt 24.10+** (APK + fw4 / nftables) — same kernel, single netns, simplest
- **MikroTik RouterOS 7.4+** with container runtime — multi-netns, has extra gotchas around routing and firewall

Reference plan file: `~/.claude/plans/openwrt-daed-vps-debian13-phantun-wireg-wise-bird.md`.

## When to use this skill

The user wants to:
- Build a WG tunnel where the WG UDP is wrapped as fake TCP (Phantun) to bypass ISP UDP QoS / blocking
- Self-host a WG egress on a Linux VPS reachable from a Chinese-network home router
- Bridge that WG endpoint into a proxy framework:
  - OpenWrt: **daed** (via sing-box SOCKS5 because daed lacks WG outbound), **OpenClash / Mihomo** (native WG support), or **sing-box** alone (transparent proxy)
  - RouterOS: **RouterOS native WireGuard** (simplest) or **Mihomo** running in another container
- Run phantun-client inside a **RouterOS container** (x86_64 or aarch64) because they don't want an extra OpenWrt device
- Deploy **multiple parallel tunnels** to different VPSs (one container per VPS)
- Diagnose an existing phantun+WG setup that "should work but doesn't"

## Architecture

```
LAN client ──► OpenWrt proxy framework (daed/OpenClash/sing-box)
                 ↓ as WG client
              UDP 127.0.0.1:4567  ──► phantun-client (TUN + raw socket)
                                        ↓ fake TCP packet
                                      pppoe-wan / eth-wan
                                        ↓ encrypted-looking TCP
                                      VPS public IP : 4567
                                        ↓
                                      phantun-server (TUN + raw socket)
                                        ↓ decapsulated UDP
                                      127.0.0.1 : 51820
                                        ↓
                                      WireGuard kernel wg0  (10.200.0.1)
                                        ↓ NAT MASQUERADE
                                      Internet
```

**Layering rationale**: Phantun does only UDP↔TCP shape-shifting (no encryption). WG provides crypto/auth. Two WG keypairs are needed (server + client). One Phantun keypair is implicit (Phantun has no auth).

## Critical gotchas — read this section first, every time

These five issues caused the longest debugging cycles:

### G1. VPS phantun-server REQUIRES a DNAT rule

`phantun-server` reads packets from its TUN interface, **not** from the eth0 raw socket directly. Without DNAT, incoming TCP SYNs on eth0:4567 never reach phantun-server, no SYN-ACK is ever generated, and connections silently time out. **You will see SYNs in `tcpdump -i eth0` but no replies, and journalctl will be silent — no error.**

Required rule (substitute your phantun TCP port and tun-peer IP):
```bash
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 4567 -j DNAT --to-destination 192.168.200.2
```

`192.168.200.2` is `phantun-server --tun-peer` (default). Confirm via `phantun-server --help`.

**Misleading observation**: Testing the VPS port from a Windows machine via `bash /dev/tcp/<vps>/4567` may show "connection succeeded" even when phantun is broken — middlebox TCP optimization on the *client* side fakes a SYN-ACK locally. Always verify the SYN-ACK is actually emitted from VPS via `tcpdump -i eth0`.

### G2. OpenWrt 24.10+ uses APK and fw4 (nftables) — old guides break

- `opkg` is gone, use `apk add <pkg>`
- Default firewall is **fw4** (nftables `inet fw4` table), policy is `forward DROP`
- iptables-nft compat layer works but **fw4 reload wipes nft rules in the `inet fw4` table** (e.g. our `nft insert rule inet fw4 forward iifname "tun-ph" accept` was lost on every firewall reload)
- iptables rules in legacy `ip filter` / `ip nat` tables (e.g. `OUTPUT` chain RST drop, MASQUERADE in `POSTROUTING`) DO survive fw4 reload (different table)

**Fix**: register the phantun TUN as a UCI firewall zone so fw4 includes it natively:
```sh
uci set firewall.phantun=zone
uci set firewall.phantun.name='phantun'
uci set firewall.phantun.input='ACCEPT'
uci set firewall.phantun.output='ACCEPT'
uci set firewall.phantun.forward='ACCEPT'
uci set firewall.phantun.mtu_fix='1'
uci add_list firewall.phantun.device='tun-ph'

uci set firewall.phantun_to_wan=forwarding
uci set firewall.phantun_to_wan.src='phantun'
uci set firewall.phantun_to_wan.dest='wan'
uci commit firewall && /etc/init.d/firewall reload
```

The wan zone's existing `masq=1` will NAT outgoing traffic — no need for a separate iptables MASQUERADE.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lilei9634/phantun-wireguard-tunnel-skill](https://github.com/lilei9634/phantun-wireguard-tunnel-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
