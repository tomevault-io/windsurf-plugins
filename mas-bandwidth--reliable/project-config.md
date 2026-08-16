---
trigger: always_on
description: WHAT: the C reference for reliable — packet fragmentation, reassembly and acks over an
---

<!-- HOT:BEGIN -->
## HOT — read before reasoning about this repo

WHAT: the C reference for reliable — packet fragmentation, reassembly and acks over an
unreliable transport. NOT reliable.rs / reliable.go (the ports).

DECISIONS THAT READ AS BUGS (they are not — do not "fix" them)
- **Release builds trust the caller. This is the design contract**, confirmed with the
  maintainer 2026-07: correct configuration is the programmer's responsibility in release.
  Debug asserts catch bad config during development; release deliberately carries no
  validation overhead. Do NOT add release-mode config checks or defensive branches — that
  includes the tempting ones like `max_fragments > 256` (which would overflow
  `fragment_received[256]`). Adding them is fighting the library, not hardening it.
- **No authentication, no anti-spoofing, and that is out of scope.** reliable assumes an
  authenticated encrypted transport beneath it — that is netcode's job. Forged packets,
  window-warping via a fake far-future sequence, spoofed acks: all netcode's to prevent.
  Do not add defenses here.
- **Fragmentation amplifies loss on purpose.** One lost fragment loses the whole packet,
  and in-progress reassemblies evicted by newer traffic are the same trade. Latency comes
  first. Callers needing large reliable blocks build block transfer above this (yojimbo
  does exactly that) rather than sending very large packets.
- **No keepalives, because there are never lulls.** reliable assumes continuous
  bidirectional exchange at ~60Hz; acks piggyback on outgoing packets. One-directional or
  bursty request/response traffic is out of scope, and the rtt/jitter/loss stats are fresh
  only under that same assumption.

THE WRITE/READ RULE — read this BEFORE reporting any assert as a missing bounds check
Glenn, 2026-07-26: "intention is on write, user is responsible to not crash or do undefined
behavior. asserts are there to help. callers responsibility. on read, obviously, we must
check." Plus Postel: "be conservative in what you send, permissive in what you receive."
  WRITE / caller-supplied -> the CALLER validates. Assert-only is the DESIGN; -DNDEBUG
    removing it is correct. Do NOT add runtime checks here.
  READ / off the wire     -> the library checks at runtime, for safety.
DELIBERATELY ASSERT-ONLY, do NOT "fix" -- a future audit that finds these has found the
contract, not a bug:
  - reliable_copy_string( dest, source, 0 ) (reliable.c:1729). dest_size is size_t, so
    `i < dest_size - 1` wraps to SIZE_MAX and writes until it finds a NUL. I proved this
    under ASan (heap-buffer-overflow WRITE at reliable.c:1739). It is still caller
    responsibility.
  - reliable_endpoint_send_packet( ep, data, negative ) (reliable.c:762). The
    > max_packet_size test at :768 is false for negatives, so it reaches the memcpy at
    :802 and the int becomes a huge size_t. Proved under ASan (negative-size-param).
    Note netcode's equivalent DOES range-check; that difference is not a defect here.
  - the config asserts in reliable_endpoint_create (reliable.c:561-572), including
    max_fragments <= 256 with no runtime companion.
THE RECEIVE PATH IS CLEAN AND UNUSUALLY WELL HARDENED -- two independent audits (Opus 5 and
Fable 5) agree. read_fragment_header checks minimum length (:963), num_fragments >
max_fragments (:982), fragment_id >= num_fragments (:988), fragment_bytes (:1038, :1044);
reliable_store_fragment_data has an explicit pre-memcpy bounds test at :1089 with a comment
naming the attacker case.
WHY fragment_received[256] (reliable.c:472) CANNOT be overrun even though max_fragments is
assert-only: fragment_id is read as a uint8_t at :979, so it is <= 255 by the WIRE TYPE
regardless of any check or config value. Verified empirically, not just by reading -- a
hostile-input harness under ASan+UBSan with -DNDEBUG hit that indexing site 24,384 times at
a highest index of exactly 255.
<!-- HOT:END -->

# CLAUDE.md

## What this is

**reliable** is a single-file C library (`reliable.c` / `reliable.h`, ~2,600 lines
including embedded tests) implementing packet acknowledgement, fragmentation/reassembly,
and RTT/jitter/packet-loss/bandwidth estimation over UDP. It is transport-agnostic: the
caller supplies `transmit_packet_function` / `process_packet_function` callbacks. The
other files (`test.cpp`, `example.c`, `soak.c`, `stats.c`, `fuzz.c`) are thin harnesses
around the library.

Build and test (CMake, 3.15+):

```
cmake -B build -DCMAKE_BUILD_TYPE=Debug        # or Release; on Windows: cmake -B build -A x64
cmake --build build                            # on Windows add: --config Debug
ctest --test-dir build --output-on-failure     # runs the test suite + bounded fuzz and soak runs
```

Binaries land in `build/bin` (`build/bin/<Config>` on Windows). Add
`-DRELIABLE_SANITIZE=ON` for ASan+UBSan. CI (`.github/workflows/ci.yml`) runs
Debug+Release on Windows x64, macOS arm64, and Ubuntu LTS, plus a sanitizer job, plus
a weekly 2M-iteration fresh-seed fuzz job under ASan/UBSan (manually triggerable via
workflow_dispatch).

Tests live at the bottom of `reliable.c` behind `RELIABLE_ENABLE_TESTS`, driven by
`test.cpp`. Debug/release is selected by `RELIABLE_DEBUG` / `RELIABLE_RELEASE`; asserts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mas-bandwidth/reliable](https://github.com/mas-bandwidth/reliable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
