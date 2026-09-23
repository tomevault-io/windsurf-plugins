---
trigger: always_on
description: Read this first when picking up work in this repository. It captures
---

# AGENTS.md — context for AI coding assistants

Read this first when picking up work in this repository. It captures
the non-obvious invariants and gotchas that aren't visible from a single
file or from the README alone.

For the human-facing tour: see `README.md` (overview), `docs/quickstart.md`
(deploy/run), `docs/fabric-design.md`, `docs/design-mrc.md`,
`docs/spray-protocol.md`, and `docs/design-appendix.md`.

---

## What this lab is

A 4-plane SRv6 fabric on docker-sonic-vs + Containerlab. The default
topology is `4p-4x8` (4 planes × 4 spines × 8 leaves per plane = 32
fabric nodes, plus 16 alpine hosts: 8 green + 8 yellow). A larger scale
design `4p-8x16` (4 planes × 8 spines × 16 leaves = 128 fabric
nodes + 32 hosts) is available via `make TOPO=4p-8x16 …` or
`SRV6_TOPO=…/4p-8x16/topo.yaml`. It demonstrates the MRC + SRv6-spray
model: one logical flow fans out across all 4 planes by varying *only*
the outer SID list.

Tenants:

- **green** — hybrid: leaf does encap+uDT6 decap in `Vrf-green`. Anycast
  inner dst `2001:db8:bbbb:<NN>::2` on all 4 NICs (`nodad`).
- **yellow** — host-based: leaf does encap+uA(host-port); host runs 4
  `seg6local End.DT6` (one per plane NIC) for decap. Anycast inner dst
  `2001:db8:cccc:<NN>::2` on all 4 NICs and on `lo` (`nodad`). Mirrors
  green's anycast plan with `bbbb`→`cccc` (Phase 1a).

## Interface naming and topology structure

### Containerlab ↔ SONiC interface mapping

Docker-sonic-vs uses `Ethernet0, Ethernet4, Ethernet8, ...` (increments
of 4). Containerlab uses `eth1, eth2, eth3, ...` (starting at 1).

**Mapping formula**: `clab eth{N}` → `SONiC Ethernet{(N-1)*4}`

Examples:
- `eth1` → `Ethernet0`
- `eth2` → `Ethernet4`
- `eth9` → `Ethernet32` (leaf downlink to green host)
- `eth10` → `Ethernet36` (leaf downlink to yellow host)

### Physical topology (4p-4x8)

**Spine connections** (NO direct host connections):
- `eth1..eth8` → downlinks to 8 leaves within the plane
- `p0-spine00:eth1` (Ethernet0) ↔ `p0-leaf00:eth1` (Ethernet0)
- `p0-spine00:eth2` (Ethernet4) ↔ `p0-leaf01:eth1` (Ethernet0)
- ...
- `p0-spine00:eth8` (Ethernet28) ↔ `p0-leaf07:eth1` (Ethernet0)

**Leaf connections**:
- `eth1..eth4` → uplinks to 4 spines (one per plane)
  - `p0-leaf00:eth1` (Ethernet0) → `p0-spine00`
  - `p0-leaf00:eth2` (Ethernet4) → `p0-spine01`
  - `p0-leaf00:eth3` (Ethernet8) → `p0-spine02`
  - `p0-leaf00:eth4` (Ethernet12) → `p0-spine03`
- `eth9` (Ethernet32) → green host downlink (in `Vrf-green`)
- `eth10` (Ethernet36) → yellow host downlink (default VRF)

**Host connections**:
- `eth1..eth4` → one uplink per plane to corresponding leaf
  - `green-host00:eth1` → `p0-leaf00:eth9` (plane 0)
  - `green-host00:eth2` → `p1-leaf00:eth9` (plane 1)
  - `green-host00:eth3` → `p2-leaf00:eth9` (plane 2)
  - `green-host00:eth4` → `p3-leaf00:eth9` (plane 3)

All link information is in `topologies/<topo>/topology.clab.yaml` under
the `links:` section with `endpoints: ["node-a:ethN", "node-b:ethM"]`
format.

## Repo layout

```
srv6_mrc/           Python package
  topo.py              fabric constants + addressing helpers (reads topo.yaml)
  topology.py          typed Topology accessor (Refactor 1 in progress;
                       parallel to topo.py during migration)
  runner.py, policy.py, reorder.py, netem.py, report.py
  encap.py             shared raw-socket SRv6 outer-packet builder
                       (used by runner.py and mrc/transport.py)
  cli/spray.py         userspace SRv6 packet generator (CLI: `spray`)
  cli/routes.py        static SRv6 route management   (CLI: `routes`)
  cli/srctl.py         kubectl-shaped lab CLI         (CLI: `srctl`)
  mrc/
    run.py             scenario orchestrator           (CLI: `run-scenario`)
    daemon.py          per-host MRC daemon: single SO_REUSEPORT reply-
                       socket owner + per-flow snapshot writer
    scenario.py        scenario YAML schema + executor (incl. `mrc:` block)
    agent.py           SenderMrcAgent / ReceiverMrcAgent + env-loader
    transport.py       MrcTransport ABC + Srv6RawTransport +
                       LoopbackUdpTransport
    ev_state.py        EVStateTable + per-plane state machine
    probe.py           PROBE / PROBE_REPLY / LOSS_REPORT wire format
    probe_clock.py     in-flight probe tracking + timeout sweep
    loss_window.py     receiver per-(flow,plane) loss accounting
    loss_compute.py    sender-side fusion of LOSS_REPORT with sent windows

generators/fabric.py   parameterized generator (reads topo.yaml)

topologies/<name>/
  topo.yaml            declarative single source of truth for one variant
  topology.clab.yaml   containerlab topology (generated)
  config/              per-node SONiC + FRR configs   (generated)
  scenarios/           MRC scenario YAMLs
  routes/              route-spec YAMLs for `routes apply`

host-image/Dockerfile  alpine + scapy + pip-installed srv6_mrc
scripts/config.sh      push configs to running containers
tests/                 329 unit tests mirroring srv6_mrc/ layout
docs/                  consolidated design + runbook docs
results/               scenario output JSON (gitignored)
Makefile               operator workflow entry point
```

## Source of truth

`topologies/<name>/topo.yaml` declares the topology: planes, spines,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [segmentrouting/srv6-mrc-emulator](https://github.com/segmentrouting/srv6-mrc-emulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
