---
trigger: always_on
description: Copyright © 2026 mindicator & silicon bags quartet.
---

<!--
Copyright © 2026 mindicator & silicon bags quartet.
SPDX-License-Identifier: AGPL-3.0-or-later
This file is part of Mycelium, licensed under the GNU Affero General Public License v3.0 or
later. See the LICENSE file in the repository root.
-->

> **Use policy.** The code is AGPL-3.0-or-later with **no field-of-use restriction** (LICENSE) — an AGPL grant cannot be narrowed to "civil use only". The prohibited-use list (military operations, covert surveillance, illegal activity) attaches to the shared **name, marks and trust roots**: use them for that and you lose the right to the marks, not the right to the code. See ACCEPTABLE-USE.md and TRADEMARKS.md.

# AGENTS.md — Operating doctrine for AI agents in this repository

This file is the operating doctrine for AI coding agents working in this repository.
Mycelium is safety-critical connectivity infrastructure. Treat every design choice as a trade-off
between reachability, resilience, privacy, operator safety, and future decentralization.

## 0. Project identity

Mycelium is a **persistent, self-adapting private network (PPN)**: server-side software that grows
from a single multi-protocol node into a decentralized, self-healing mesh.

It is **not**:

- a generic P2P VPN;
- a network of centrally managed VPN servers with a nicer UI;
- a company-shaped service with all users around a permanent center;
- a new cryptographic protocol;
- a bespoke end-user client project.

It is a private connectivity fabric whose useful behavior should emerge from local measurement,
limited trust, route diversity, stress memory, reversible adaptation, and carrier-agnostic bridging.

The biological model is operational, not decorative:

- **hyphae explore**: nodes spend a bounded budget on cheap route/transport/peer probes;
- **anastomoses connect**: independent local paths can fuse under scope and trust constraints;
- **cords carry**: repeatedly useful paths can become temporary high-capacity corridors;
- **gradients guide**: growth is biased toward demand, scarcity, trust, and stress;
- **stress leaves memory**: failures change future routing without exposing users;
- **dead paths decay**: unused or unverified topology expires;
- **spores germinate**: small signed artifacts can survive disconnection and restart reachability;
- **local signals create global structure**: no node needs the full map to improve the network.

## 1. Non-negotiable constraints

Every change must preserve these constraints unless an ADR explicitly changes them.

1. **No custom cryptography or custom transports.** Use audited, standard, widely deployed
   primitives and existing projects. Innovation belongs in adaptation, orchestration, measurement,
   safe decentralization, and carrier adaptation.
2. **Server-side scope.** Nodes expose standard endpoints consumed by off-the-shelf clients. A
   bespoke end-user client is out of scope for the current roadmap.
3. **Indistinguishability over decorative obfuscation.** The target is statistical resemblance to
   ordinary HTTPS/QUIC and fast shape-change, not a tunnel that merely looks unusual in a new way.
4. **No permanent central brain.** Temporary coordination is allowed. Dependency on one coordinator,
   registry, endpoint list, dataset, model, bootstrap server, or bridge class is tracked technical debt.
5. **No full-map visibility by default.** A node should know local neighbors, scoped route options,
   local health, and current obligations — not the whole network.
6. **No raw user telemetry.** Do not collect traffic content, user identities, complete peer lists,
   full topology maps, private content, or persistent behavioral profiles for learning.
7. **User/operator safety beats cleverness.** If a feature improves reachability while increasing
   legal, operational, or de-anonymization risk, it does not ship without an explicit safety review.
8. **Measure before adapting.** No auto-rotation, scoring, quarantine, pruning, or promotion without
   observable signals and a false-positive story.
9. **Degrade, do not fail.** Every layer needs a reduced mode: local mesh, delayed delivery, cached
   content, manual operator recovery, or out-of-band bootstrap.
10. **Mutualism before economics.** Do not introduce tokenomics, bandwidth markets, global node
    rankings, stake-weighted routing, or extractive incentives in early architecture. Contribution may
    increase scoped trust and resilience; it must not turn the network into a farmable market.
11. **Any carrier can be a bridge.** IP links, LTE/5G, satellite, Wi-Fi Direct, Bluetooth, LoRa-style
    meshes, local Ethernet, removable media, QR/file hand-off, and future radios are lower carriers,
    not separate Mycelium protocols.

## 2. Phase discipline

Always identify the phase and layer before proposing code or documentation changes.

### Phase 0–2: single node / multi-protocol / adaptation layer

Allowed:

- clean abstractions for transport health, endpoint bundles, detector signals, and rotation policy;
- metrics that make adaptation testable;
- interfaces that can later be backed by gossip, DHT, or local trust;
- data models that can later carry signed spore artifacts across non-IP carriers;
- deployment reproducibility and safe defaults.

Forbidden even if it seems convenient:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mycelium0/mycelium](https://github.com/mycelium0/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
