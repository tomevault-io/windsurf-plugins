---
trigger: always_on
description: `nbox` is a CLI + TUI for NetBox (DCIM/IPAM). For programmatic/agent use, drive the
---

# nbox for agents

`nbox` is a CLI + TUI for NetBox (DCIM/IPAM). For programmatic/agent use, drive the
CLI subcommands with machine-readable output. The interactive TUI (`nbox` with no
subcommand) is for humans; agents should always pass a subcommand. Pass `--no-tui`
to make that a hard guarantee: any invocation that would launch the TUI (a bare
`nbox`, or `nbox tui`) refuses with a usage error (exit 2) instead of blocking on a
terminal.

## Output

- `--json` / `-o json` — JSON to stdout (pretty by default).
- `--raw` — compact JSON (one line; pairs with `--json`).
- `--envelope` — wrap as `{ "schema_version": 1, "data": <payload> }` for stable parsing.
- `--fields a,b,c` — keep only those top-level fields (per element for arrays).
- `-o csv` — CSV for tabular/list results (e.g. `search`); arrays render as a table. Single objects are rejected (exit 2) — use `--json` or plain.

stdout carries only the requested data; logs/diagnostics/errors go to stderr.

Exit codes (stable):

| Code | Meaning                                   |
| ---- | ----------------------------------------- |
| 0    | success                                   |
| 1    | generic error (incl. other API failures)  |
| 2    | usage error (bad arguments)               |
| 3    | authentication / permission (HTTP 401/403)|
| 4    | not found (no object matched)             |
| 5    | ambiguous reference (more than one match) |

Recommended agent invocation: `nbox <cmd> ... --json --envelope` (add `--raw` to
minimize tokens, `--fields` to trim payloads).

## Commands

```
nbox device <name|slug|id>
                                  # optional write subcommand:
  device <name> set status <value> [--message "…"]
                                  #   --dry-run | --allow-writes --confirm
                                  #   (ADR-0001 safe write; read-only default)
nbox ip <address> [--vrf <name|slug|rd>]  # surfaces nat_inside/nat_outside (NetBox 4.6) when set
nbox ip reserve <cidr> [--vrf <name|slug|rd>] [--description "…"] [--dns-name "…"] [--count N]
                                  # write: reserve the next available IP (POST available-ips); --dry-run | --allow-writes --confirm [--message]
                                  # --count N: reserve N IPs atomically (one list-body POST, all-or-nothing); any failure exits 1, nothing created
nbox prefix <cidr> [--vrf <name|slug|rd>]
                                  # optional write subcommand:
  prefix <cidr> reserve [--length L] [--vrf <name|slug|rd>] [--description "…"]
                                  # write: reserve the next available child prefix (POST available-prefixes); --dry-run | --allow-writes --confirm [--message]
nbox next-ip <cidr> [--count N] [--vrf <name|slug|rd>]
nbox next-prefix <cidr> [--length L] [--vrf <name|slug|rd>]
nbox vlan <vid|name> [--site <name|slug>] [--group <name|slug>]
nbox interface <device> <interface>
                                  # optional write subcommand:
  interface <device> <interface> set description "…" [--message "…"]
                                  #   --dry-run | --allow-writes --confirm
                                  #   (ADR-0001 safe write; read-only default)
nbox site <name|slug>
nbox rack <name|id>
nbox rack-group <slug|name|id>      # NetBox 4.6+
nbox circuit <cid|id>                 # JSON: `terminations` (A/Z), each path hop a `device` ref + a `diagram`
nbox virtual-circuit <cid|id>        # JSON: `terminations` (multi-point interface refs); NetBox 4.2+
nbox provider <slug|name|id>
nbox vm-type <slug|name|id>          # NetBox 4.6+
nbox aggregate <cidr|id>
nbox asn <number>
nbox ip-range <start|id>
                                  # optional write subcommand:
  ip-range <start|id> reserve [--description "…"] [--dns-name "…"] [--count N]
                                  # write: reserve the next available IP in an IP range (POST available-ips); --dry-run | --allow-writes --confirm [--message]
                                  # --count N: reserve N IPs atomically (one list-body POST, all-or-nothing); any failure exits 1, nothing created
nbox tenant <slug|name|id>
nbox contact <name|id>
nbox vm <name|id>
nbox cluster <name|id>
nbox vrf <name|rd|id>
nbox route-target <name|id>
nbox mac <addr>                   # any common form (aa:bb:cc:dd:ee:ff, AABB.CCDD.EEFF, …) is normalized; reverse-resolves to the carrying interface(s)/device(s)
nbox search <query> [--limit N] [--status S] [--site <name|slug|id>] [--region <name|slug|id>] [--site-group <name|slug|id>] [--location <name|slug|id>] [--tenant SLUG] [--role SLUG] [--tag SLUG] [--owner <name>] [--owner-group <name>] [--vrf <id|rd|name>] [--cols a,b,c] [--partial]
nbox tags
nbox tagged <tag>               # objects carrying a tag, across kinds (NetBox 4.3+
                                  # `/api/extras/tagged-objects/`); tag = id|name|slug
nbox tag add <type> <name> <tag>  # write: add a tag to any object (PATCH tags array); --dry-run | --allow-writes --confirm [--message]
                                  # <type> = any read kind (device, ip, prefix, vlan, …); <tag> = id|name|slug; no-op if already present
nbox tag remove <type> <name> <tag>  # write: remove a tag from any object (PATCH tags array); same gate as tag add; no-op if already absent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lance0/nbox](https://github.com/lance0/nbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
