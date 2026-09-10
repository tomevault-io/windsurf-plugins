---
trigger: always_on
description: WHAT: the C++ client/server networking library built on netcode + reliable + serialize,
---

<!-- HOT:BEGIN -->
## HOT — read before reasoning about this repo

WHAT: the C++ client/server networking library built on netcode + reliable + serialize,
all three VENDORED in-tree (netcode/, reliable/, serialize/) plus a pruned libsodium subset
in sodium/.

**THE INTERFACE DOES NOT CHANGE.** The library is over ten years old and deliberately
stable. Do not propose interface changes.

**VENDORED CODE IS NOT EDITED HERE.** sodium/ is byte-identical to netcode's copy and
`.github/workflows/sodium-parity.yml` fails if it drifts; see sodium/NOTES.md. Patch
upstream, re-vendor, let it flow down.

DECISIONS THAT READ AS BUGS (they are not — do not "fix" them)
- **Debug asserts are deliberately kept OFF the untrusted network read path**, so a hostile
  peer cannot crash a debug server. This is the single most important invariant to preserve
  when adding validation: the full-trust rule for programmer inputs must never be confused
  with the zero-trust rule for wire data.
- **Config invariants assert in constructors** (power-of-two buffer sizes, channel counts)
  so every debug run validates config at startup; usage contracts assert where exercised;
  API misuse ALSO degrades gracefully in release. That layering is intentional.
- **Malformed wire input degrades to a channel error and a disconnect — never an assert.**
  Validation happens before the memcpy (see the over-long-final-fragment check in
  `ReliableOrderedChannel::ProcessPacketFragment`).
- **Allocation failure is handled on essentially every path** deliberately, because this is
  a game server.

SECURITY: yojimbo 1.6.3 and earlier vendor a netcode missing the AEAD nonce-reuse fix
(netcode 1.4.0); 1.7.0 was the first with it. See netcode's SECURITY.md.
1.12.1 vendors netcode 1.4.5, which spends a connect token on the connection it admits and
refuses tokens that predate the server's start. See SECURITY.md.

THE WRITE/READ RULE — read this BEFORE reporting any assert as a missing bounds check
Glenn, 2026-07-26: "intention is on write, user is responsible to not crash or do undefined
behavior. asserts are there to help. callers responsibility. on read, obviously, we must
check." Plus Postel: "be conservative in what you send, permissive in what you receive."
  WRITE / caller-supplied -> the CALLER validates. Assert-only is the DESIGN; -DNDEBUG
    removing it is correct. Do NOT add runtime checks here.
  READ / off the wire     -> the library checks at runtime, for safety.
yojimbo states this in its own code: yojimbo_config.cpp:34-36 explains that YOJIMBO_CONFIG_CHECK
logs-then-asserts and the whole Validate() family is compiled to nothing in release
(yojimbo_config.cpp:163-175). That is deliberate, not an oversight.
DELIBERATELY ASSERT-ONLY, do NOT "fix" -- a future audit that finds these has found the
contract: oversized block-message send (reliable_ordered_channel.cpp:562/611 -- SendMessage
exempts block messages at :168 on purpose); BaseServer::Start's maxClients
(base_server.cpp:54-55); Client::InsecureConnect's numServerAddresses (client.cpp:50-52); the
whole clientIndex/channelIndex public surface. This rule is about CALLER-SUPPLIED values only:
allocation and factory failure inside Start / CreateInternal is not a caller mistake, and those
are checked in every build and reported through the bool those functions now return (YJ-01).
The WIRE path is properly checked and was verified clean by two independent audits: wire
channelIndex (channel.cpp:446), numChannelEntries (connection.cpp:58), numFragments/fragmentId
(reliable_ordered_channel.cpp:712-724) and the receive memcpy bounds test at :733-738.
yojimbo also satisfies serialize's bytes%8 write contract deliberately at connection.cpp:248
(maxPacketBytes &= ~7) -- do not "simplify" that line.
FIXED 2026-07-26 (#320): Address::IsMulticast/IsLinkLocal/IsSiteLocal compared the first
16-bit group for EXACT equality instead of masking the prefix, and IsGlobalUnicast is their
negation, so ff02::1 (all nodes) reported as GLOBAL UNICAST. Now masked: /8 for multicast,
/10 for link and site local. There had been ZERO test coverage of these predicates, which is
how they drifted -- test_address_classification now covers the range boundaries.
<!-- HOT:END -->

# CLAUDE.md — Audit of yojimbo

*An honest code audit written by Claude (July 2026), covering the yojimbo library proper
(`include/`, `source/`), its vendored dependencies (`netcode/`, `reliable/`, `serialize/`,
`sodium/`, `tlsf/`), tests, fuzzing, CI, and documentation.*

## Orientation

yojimbo is a C++ client/server network library for real-time multiplayer games. It layers
cleanly: **yojimbo** (messages, channels, connection, allocators) sits on **reliable**
(acks, packet fragmentation), **netcode** (encrypted UDP, connect-token authentication),
and a pruned **libsodium** (crypto), with **serialize** (bitpacker) and **tlsf**
(per-client heaps) alongside. Each dependency is vendored as an amalgamated one- or
two-file library maintained by the same author. The yojimbo layer itself is ~9,300 lines
of source plus well-documented headers — small enough to hold in your head, which is a
feature.

## Design contract (per the author)

These are deliberate design decisions, not defects — reviews and changes should work

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mas-bandwidth/yojimbo](https://github.com/mas-bandwidth/yojimbo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
