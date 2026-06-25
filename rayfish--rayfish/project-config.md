---
trigger: always_on
description: P2P mesh VPN powered by [iroh](https://iroh.computer). Connects peers by cryptographic identity (EndpointId), not IP address. Dual-stack addressing: stable IPv4 in 100.64.0.0/10 (CGNAT, FNV-1a of identity) and stable IPv6 in 200::/7 (blake3 of identity, 120-bit, never rotates).
---

# Rayfish

P2P mesh VPN powered by [iroh](https://iroh.computer). Connects peers by cryptographic identity (EndpointId), not IP address. Dual-stack addressing: stable IPv4 in 100.64.0.0/10 (CGNAT, FNV-1a of identity) and stable IPv6 in 200::/7 (blake3 of identity, 120-bit, never rotates).

## Build

```bash
cargo -q build                 # add --features tor for Tor transport, --features otel for OTLP span export
cargo -q check
cargo -q test
cargo -q clippy
cargo bench                    # Criterion microbenchmarks of the per-packet data path (benches/forward.rs)
```

The crate splits into a library (`src/lib.rs`, daemon modules as `pub mod`) and a thin binary (`src/main.rs`, the `ray` CLI/IPC client, `use rayfish::…`). The split lets benchmarks (`benches/`) and integration tests reach the internal data path; `cargo install` builds the binary against the in-package library unchanged.

## Run

The daemon (`ray daemon`) owns the TUN device and iroh endpoint and runs as a system service. CLI commands talk to it over Unix-socket IPC.

```bash
sudo ray up                    # install+start the service, then activate the VPN
ray create [--open] [--name n] [--hostname h] [--tor]   # closed by default; --open = public network. Prints room id (public key)
ray join <room-id-or-invite> [--name alias] [--hostname h] [--auto-accept-firewall] [--tor]  # join by room id or one-time invite code; --auto-accept-firewall auto-installs suggested rules (managed node/server)
ray leave <net> | nuke <net>   # nuke = publish empty record then leave
ray hostname <net> <name>      # change hostname on existing network
ray status                     # all networks (works without daemon); per-host traffic, member count excludes self
ray <cmd> --json               # global flag: machine-readable JSON for status/firewall show/files/invite list/requests/admin list (color + spinners off)
ray report                     # bundle logs+metrics, open a pre-filled GitHub issue
ray up [--hostname h] | down   # activate / standby (TUN + DNS), daemon stays running; --hostname sets your default name

ray invite <net> [--expires 7d] [--hostname H] [--qr]   # coordinator-only: mint single-use invite; --qr prints a scannable QR; --hostname binds an authoritative name (overrides joiner choice, rejected on collision)
ray invite <net> --reusable [--expires 30d]          # mint a reusable (multi-use, expiring) key for unattended fleets; rides the signed blob, no hostname binding. Servers: ray join <key> --hostname H --auto-accept-firewall
ray invite <net> list|revoke <id>          # list / revoke invites (reusable keys tagged; revoke propagates via the blob)
ray requests <net>             # coordinator-only: peers awaiting live approval
ray accept <net> <id> | deny <net> <id>    # admit / reject a pending join request
ray connect <contact-id> [--hostname h]    # request a direct 2-peer connection by the peer's contact id (no room id/invite); blocks as pending until they approve
ray connections [approve <id>]             # list incoming connect requests (default) / approve one → mints a 2-peer network with the requester pre-approved
ray contact [id|rotate]        # print (default) or rotate your shareable contact id (also shown at the top of `ray status`)
ray admin <net> add <id> | list            # coordinator-only: grant the network key (co-coordinator) / list key-holders
ray firewall show|default|add|remove ...               # per-device local firewall. Default posture: inbound TCP/UDP denied, inbound ICMP allowed, outbound allowed. `firewall default allow|deny` sets the inbound default
ray apply <spec> [--prune] [--dry-run] [--invite-missing] [--example]   # declarative deploy (YAML only): create closed nets + suggest firewall + report membership gap
ray firewall suggest <net> --subject H [--allow peer:proto:ports] [--deny peer:proto:ports]  # coordinator-only: suggest rules on any network (rides the signed blob). Subject/peer `*` = all hosts / any peer. `--allow`/`--deny` value is `[peer:]proto:ports` — the `peer:` prefix is optional, so a bare `tcp:22` (or `icmp`) means "any peer" (parsed by `main::parse_suggest_token`: a leading protocol keyword ⇒ peer `*`). Token grammar: `proto:ports` (tcp:22, udp:53, tcp:*, any:*) or bare proto (icmp, any, tcp). An allow-list ⇒ whitelist (catch-all deny appended); denies-only ⇒ blacklist
ray firewall pending <net> | accept <net> | deny <net>  # review/accept/discard queued suggested rules. On a TTY, `pending` is an interactive picker (↑↓ · enter accept · d deny · a all · q done); piped/`--json` falls back to a static table
ray firewall auto-accept <net> on|off  # toggle this node's auto-install of suggested rules for a network (on = install current queue)
ray mdns on|off                # local peer discovery (default on)
ray send <file> <peer>         # file sharing; ray files [accept <id> [--output dir]]
ray pair [<ticket>|backup|restore <code>]              # multi-device identity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rayfish/rayfish](https://github.com/rayfish/rayfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
