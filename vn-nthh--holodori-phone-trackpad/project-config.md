---
trigger: always_on
description: This repository is an unofficial companion for **hololive Dreams (holodori)**.
---

# Holodori Phone Trackpad: product and engineering contract

This repository is an unofficial companion for **hololive Dreams (holodori)**.
Treat it as a latency-sensitive rhythm-game input device, not as a generic
trackpad, remote-control demo, or lossy event stream.

## Why this app exists

QualiArts and hololive describe holodori as a Rhythm & RPG whose rhythm game is
the core experience. The official Steam page presents a full rhythm game with
hard charts and player-created charts, alongside the park and mini-games.

The mobile charts use spatial touch gestures. PC community feedback consistently
describes the Steam keyboard adaptation as much flatter: wide key zones let a
small number of keys cover much of the board, flicks behave like taps, and a
moving hold can often be satisfied by holding one stationary key. This project
exists so a player can use an Android phone as the missing multi-touch surface
while running the PC version.

The default keyboard bridge maps the phone surface to `S D F J K L`. Its job is
not merely to emit the key under the latest coordinate. It must preserve the
physical play vocabulary:

- a tap is a timed down/up;
- a hold is continuous asserted state for as long as the finger remains down;
- a slide is an ordered path through every crossed lane;
- a chord is simultaneous state with independent finger ownership;
- a lift or cancellation must release the corresponding Windows state.

## Current usage baseline

**Keyboard mode is the current real-world usage path.** Treat gameplay bug
reports and validation as keyboard-mode issues unless the user explicitly says
otherwise; do not repeatedly ask which input mode they use. Windows touch
injection is a proof of concept, not the current gameplay baseline.

For rapid taps and slide-to-tap transitions, distinguish ordered OS submission
from game-observed input. Passing host event-order tests alone does not prove
that the game recognizes closely spaced releases and presses. Reports can
involve either the same finger or different fingers; do not assume overlapping
ownership of one lane explains every missed tap.

## Non-negotiable gameplay invariants

1. **A stationary hold is active input, not idleness.** Do not use lack of
   `MOVE` callbacks as a disconnect signal. Full-state heartbeats sustain it.
2. **A slide is history, not a latest-value update.** Never coalesce gameplay
   frames to the newest coordinate. Copy Android `MotionEvent` history in time
   order and apply every sequence in order.
3. **Crossed lanes cannot disappear.** If one reported coordinate jumps over
   lanes, walk every intermediate lane. Press the next lane before releasing
   the previous one so a slide has no no-key gap.
4. **Multi-touch ownership is reference-counted.** One finger moving or lifting
   cannot release a lane still held by another finger.
5. **An ACK means durable input progress.** Windows may acknowledge a sequence
   only after the chosen OS sink accepted it. Parsing or buffering is not
   success.
6. **Liveness means committed sequence progress.** Duplicate ACKs, discovery
   packets, malformed frames, and valid future frames behind an ordering hole
   prove traffic, not progress. They must not keep stale input alive forever.
7. **Failure is a clean boundary.** Release all injected Windows input, reject
   delayed gameplay from the failed session, require a fresh session-start
   `CANCEL`, and then reconstruct any still-held phone contacts from the latest
   complete snapshot.
8. **Never replay seconds-old gameplay.** Reliability repairs short packet loss;
   it must not turn an outage into late notes after reconnection.
9. **V5 input is authenticated before it is actionable.** Discovery and an
   incomplete pairing handshake can never reach an OS input sink. Do not
   silently downgrade, and never accept v4 gameplay on a Wi-Fi listener. A
   remote `PAIR_CONFIRM` never replaces the required local host approval.
10. **An AEAD nonce belongs to one datagram.** In v5, every immediate copy and
    repair gets a new packet number and nonce even when it carries the same
    logical frame. Never patch timestamps and re-encrypt under a reused nonce.
11. **Transport selection is explicit.** Pair and Start operate only on the USB
    or local-network interface chosen by the user. An interface change is a
    clean authenticated-session boundary.
12. **Thumb mode cannot erase touch history.** Transform every historical and
    current Android sample before framing. A captured contact crossing the
    center gap stays captured and moves monotonically from lane 3 to lane 4.

## Latency contract

The live target is one 120 Hz frame: **8.333 ms**. This is a design budget, not
a universal hardware guarantee. Phone touch scan rate, Android scheduling, the
USB controller, RNDIS, Windows scheduling, and the game all contribute.

Changes must preserve these properties:

- no intentional batching, debounce, frame-age wait, or polling bridge;
- no UI rendering, logging, report formatting, sorting, or file I/O on the hot
  path;
- no JSON or per-frame process boundary in the native path;
- first send is immediate, its redundant copy is immediate, and repair begins
  after 2 ms;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vn-nthh/holodori-phone-trackpad](https://github.com/vn-nthh/holodori-phone-trackpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
