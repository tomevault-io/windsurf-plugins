---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A polyglot monorepo for **DCF / Punctim** (DeMoD Communication Framework) — a
handshakeless, encryption-free, export-control-compliant mesh protocol. The same
protocol is implemented as bindings/SDKs across C, C++, Rust, Go, Python, Perl,
Java/Kotlin, Swift, Node.js, Haskell, and Common Lisp, each in its own top-level
directory.

The gravitational center of current work is narrow: the **wire quantum** and its
cross-language certification, now extended with a certified **audio** layer.
Read these first — they are normative:
- `Documentation/WIRE_QUANTUM_SPEC.md` — the 17-byte `DeModFrame` wire format.
- `Documentation/DCF_AUDIO_SPEC.md` — collaborative audio as an adapter over it.
- `Documentation/DCF_GAME_SPEC.md` — multiplayer game state/events as an adapter
  over it (same fragmentation scheme as audio, on `DATA` frames).
- `Documentation/DCF_TEXT_SPEC.md` — chat / agent-to-agent UTF-8 text as an adapter
  over it (also on `DATA` frames, but a 10-bit fragment index).
- `Documentation/DCF_SSTV_SPEC.md` — slow-scan television (still images) as an adapter
  over it (also on `DATA` frames, but an 11-bit fragment index), carried over HydraModem.
- `Documentation/DCF_SNAKE_SPEC.md` — a synchronized studio audio snake over cat5e: a star
  of nodes → one "mixer" hub, with a quanta-coded **record plane** (`CTRL` 5:11) and a
  bidirectional low-latency PCM **cue plane** (`CTRL` 9:7), locked to a `BEACON` media clock.
- `Documentation/DCF_MESH_SPEC.md` — self-healing redundancy (peer-health FSM,
  REPORT/ROLE control, election + failover) as a `MsgMesh` control adapter.
- `Documentation/HYDRAPACK_SPEC.md` — universal serialization above DeModFrame + Pipe:
  a declarative schema model, plane-aware emission (quantum vs Pipe), and byte-certified
  bit-packing across C/Rust/Python. Not a new wire format; the library intermediary.
- `Documentation/DCF_PIPE_MULTI_SPEC.md` — DCF-Pipe Multi-Control: up to 3 steady-state
  DCF-Pipe commands packed into one 4-byte DeModFrame payload for concurrent pipe
  steering. Control adapter; pipe_vectors.json untouched.
- `Documentation/DCF_SECURITY_EXPOSURE.md` — the plaintext wire's exposure and the
  WireGuard / external-crypto deployment rule.
- `Documentation/DCF_SPA_SPEC.md` — single-packet port authorization: an
  **authentication-only** (EAR99) side channel that gates mesh data ports, NOT a
  wire format (the certificate is untouched). Implemented in `spa/` (Rust
  `dcf-spa-authorizer` + `python/dcf/spa/knock.py`).
- `Documentation/DCF_CODE_REVIEW.md` — frank, module-by-module status (consult
  before trusting any module's surface area).

## The wire quantum (the one invariant)

One wire format: the 17-byte `DeModFrame` (version nibble = 1). Layout:
`sync(0xD3) | flags[ver|type] | seq | src | dst | payload(4B) | ts24 | crc16`.
Valid iff sync byte + version nibble + CRC-16/CCITT-FALSE over bytes `[0..14]`.
Everything else on the wire is an **adapter** over this quantum. CRC anchors:
`CRC("123456789")=0x29B1`, `CRC(0^15)=0x4EC3`.

Reference codecs (must stay byte-identical):

| Lang | File | Entry points |
|------|------|--------------|
| C | `codec/demod_frame.h` | `dcf_frame_encode/decode`, `dcf_crc16` |
| Rust | `codec/frame.rs`, `codec/src/lib.rs` | `Frame::encode/decode` |
| Python | `python/MCP/wirelab_core.py` | `encode`/`decode`/`crc16_ccitt`/`syndrome` |
| Lua | `GUI/wirelab.lua` | `encode`/`decode`/`crc16` (self-certs on load) |
| Haskell | `haskell/src/DCF/Transport/FrameSpec.hs` | `encodeFrame`/`decodeFrame` |
| Lisp | `lisp/src/punctim.lisp`, `lisp/src/wire.lisp` | `encode-dcf-frame` |

### Certification is the contract

`Documentation/golden_vectors.json` is a 246-vector certificate (109 encode +
137 syndrome). Matching all 246 ≡ agreeing with the reference on the entire
input space. When you touch any codec, regenerate and diff:

```sh
python3 python/MCP/verify_laws.py /tmp/gv.json     # regenerate + verify laws
python3 python/MCP/certify_sdk.py --selftest
cd codec && cargo test --test certify              # Rust
```

CI: `.github/workflows/wire-certify.yml` runs Python/C/Rust certs on push/PR to
`main` and diffs regenerated vs committed vectors.

> Canonical Python lives in `python/MCP/` (the old `GUI/MCP/*` dangling symlinks
> were removed). `C_SDK/tests/test_wire_certify.c` is fixed and certifies against
> the current `dcf_frame_t` / `dcf_crc16` API.

## DCF-Audio (collaborative audio over the wire)

A 20 ms codec block is an **adapter** over `DeModFrame`, serialised into
`1 + ceil(payload_len/4)` ordinary `CTRL` (type 3) frames. The **L2 framing**
is **codec-agnostic and byte-certified across C/Rust/Python**; `codec_id` lives
in the descriptor, so adding codecs never changes the vectors. Precisely: the L2
framing, the PCM-diag codec bytes, and the PM param layout are byte-certified;
**Opus output and PM synthesis audio are NOT byte-certified**.

- `seq = packet_id[15:5] | frag_idx[4:0]`; `frag_idx 0` = `[len, frag_total, codec_id, flags]`; payload ≤ **124 B/block**.
- Codecs: Opus (id 0), PCM-diag (id 1, byte-certified), Faust phase-mod (id 2).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALH477/Punctim](https://github.com/ALH477/Punctim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
