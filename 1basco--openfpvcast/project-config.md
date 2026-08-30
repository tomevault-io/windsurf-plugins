---
trigger: always_on
description: Working notes for anyone — human or agent — touching this repo. It carries the protocol knowledge
---

# AGENTS.md

Working notes for anyone — human or agent — touching this repo. It carries the protocol knowledge
the code depends on, so you don't have to re-derive it from the wire.

## What this is

An Android app that casts the video from DJI FPV goggles (N3 / ZV300, O4 air units) to the phone
over USB, decodes it, records it, and controls the air-unit camera. Three pieces:

| path | what |
|---|---|
| `apps/web` | SvelteKit UI (Svelte 5 runes), wrapped by Capacitor |
| `plugins/capacitor-dji-aoa` | the native half: USB transport, protocol codec, H.264 decode, MP4 muxing, WebRTC |
| `android/` | Capacitor Android shell |

Build: `pnpm sync` (web build + `cap sync`), `pnpm android` to run on a device. `pnpm --filter web
check` typechecks.

## Ground rules

**Every protocol constant in this repo was established by observing the hardware.** Byte maps,
command ids, offsets and CRC seeds are empirical findings, not documentation from a vendor. Two
consequences:

1. **Grade your claims, in the comment, where the value lives.** `[HARDWARE]` = confirmed by a live
   SET → GET round-trip on a real unit, with the date and the model. `[INFERRED]` = our reading, no
   confirmation. Never quietly promote an inference to a fact.
2. **Do not attribute findings to any third-party application, its binaries, symbol names or
   addresses.** Describe the studied case — what was sent, what came back, what that implies. A
   value that can only be justified by pointing somewhere else does not belong in the codebase; go
   confirm it on hardware or mark it `[INFERRED]`.

**Comments.** A comment earns its place only when the code cannot say it: why a decision was made,
where a magic value came from, the shape of a dense algorithm, or what breaks if you change it.
Delete anything that restates the code, decorates a section, or narrates edit history. Never delete
tool directives (`@ts-expect-error`, `svelte-ignore`, `eslint-disable`, `#region`), licences, or
`TODO`s. If a comment and the code disagree and you can't tell which is right, leave it and say so.

## Transport

The goggles expose one app-layer protocol (**DUML**) over three USB personalities. Only one streams
video to a phone:

| mode | USB host | device | framing | video |
|---|---|---|---|---|
| **AOA** (Android) | **the goggles** | phone re-enumerates as `18D1:2D00` | LogicLink | port `0x574A` |
| MFI (iPhone) | goggles | `2CA3:1002` | LogicLink | port `0x574A` |
| PC native | the PC | `2CA3:0020` | raw DUML, no LogicLink | gated shut |

The inversion is the thing to internalise: **in AOA the goggles are the USB host and the phone is
the accessory** (`UsbManager.openAccessory`). The firmware's video gate only opens for a phone that
authenticated over AOA/MFI, which is why a PC-side probe gets a STALL and why an ESP32 bridge that
completes the AOA negotiation still receives zero bulk IN tokens — the block is in the goggles' AOA
host state machine, below the app layer. Sending correct bytes cannot help when the host never
reads your endpoint.

### LogicLink envelope

```
off  size  field
0    2     magic 0x55 0xCC
2    2     port (u16 LE)
4    2     payload length (u16 LE)
6    2     reserved 0x00 0x00
8    N     payload
```

Three ports matter: **`0x5749`** session/registration and video control · **`0x574A`** H.264
Annex-B video · **`0x7530`** DUML telemetry and commands. Several envelopes can share one USB
transfer; the parser resyncs on `55 CC`.

The `0x574A` payload is fed **directly** to the decoder — pure H.264 Annex-B, no wrapper, no SEI
stripping.

### DUML packet

```
off    size  field
0      1     magic 0x55
1      2     length (bits 0-9) | version (bits 10-15)   length = total bytes incl. trailing CRC-16
3      1     CRC-8 of bytes 0..2
4      1     src address
5      1     dst address
6      2     sequence (u16 BE)
8      1     cmdType
9      1     cmdSet
10     1     cmdId
11..   N     payload
end-2  2     CRC-16 of bytes 0..end-3 (LE)
```

- **CRC-8**: init `0x77`, poly `0x8C` (reflected), over bytes 0..2.
- **CRC-16**: init `0x3692`, poly `0x8408` (reflected CCITT), over bytes 0..end-3.
- **cmdType**: bit 7 (`0x80`) = REPLY · bit 6 (`0x40`) = REQUEST · bits 5-6 also carry the ACK
  policy · bits 0-2 encryption. **Outgoing camera requests use exactly `0x40`.** Setting
  ACK-required makes the camera drop the packet silently — it surfaces as an ACK timeout on every
  apply, which is a miserable thing to debug. `[HARDWARE]`
- **cmdSets** in play: `0x00` General · `0x02` Camera · `0x03` Flight Control · `0x09` HD Link ·
  `0x19` an O4-specific set, undocumented.

### Stream-start handshake

The N3 pushes video only after the mobile side registers, and it needs the registration **resent as
a ~1 Hz keepalive** — not sent once. Two LogicLink frames on port `0x5749`:

- **camcap_common** — DUML src `0x02` → dst `0x28`, cmdType `0x40`, cmdSet `0x00`, cmdId `0x99`
- **APP register** — DUML src `0x02` → dst `0x3C`, cmdType `0x40`, cmdSet `0x00`, cmdId `0x88`

Both are stored as verbatim byte strings in `UsbAoaService.kt` with their CRCs already baked in;
write them raw, do not re-encode. Without the keepalive only telemetry flows on `0x7530` and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1Basco/openfpvcast](https://github.com/1Basco/openfpvcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
