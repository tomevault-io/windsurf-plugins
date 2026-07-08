---
trigger: always_on
description: **This file is the single source of truth for project state. Read it first. Do not generate "handoff documents" — update this file instead.**
---

# CLAUDE.md — Project context for Claude sessions

**This file is the single source of truth for project state. Read it first. Do not generate "handoff documents" — update this file instead.**

---

## What this is

Multi-protocol LoRa mesh bridge on Seeed Xiao ESP32-S3. Bridges Meshtastic, MeshCore, and (stub) Reticulum networks across two radios sharing one SPI bus.

- **Repo:** https://github.com/GrayHatGuy/Xiao-esp32s3-lora-repeater
- **Local path:** `C:\Users\6r4yh\workspace\Platformio\Projects\lora_cam_xiao` (renamed 2026-06-28 from `…repeater – dev_2xiao_4sx1262`; on branch `lora_cam_xiao`)
- **Owner:** GrayHatGuy — `grayhatguyllc@protonmail.com`
- **Contest:** Seeed/Meshtastic Build-Off 2026, issue #2 at `Seeed-Projects/meshtastic-build-off-2026`

## Current state

| Item | Value |
|---|---|
| **Production line** | `main` @ **v10.0 "LoRaCam"** (`8d0d39e`, tag `v10.0`, shipped 2026-07-04) — a **LoRa-commanded camera** on the bridge firmware: XIAO ESP32-S3 **Sense** (OV2640 + microSD) + one hand-wired edge SX1262; encrypted/whitelisted/replay-protected C2 over Custom 0x33, signed photo+video capture to microSD, an always-on **WPA2 web portal** (live MJPEG + snap/record + Photos + config + pairing + messaging), secure first-flash, optional multi-hop C2 relay. **All camera code gated `-DBRIDGE_ROLE_CAMERA` → stock bridge byte-identical (865781 B).** Lineage: … v8.4.1 → v9.0 2xiao_4sx1262 → **v10.0 LoRaCam**. **GitHub release is a DRAFT** (12 bins: loracam cam + commander + master v1.0/v1.1 + coproc v1.0/v1.1, each app+factory) — awaiting owner review + publish. |
| **Active cycle** | **none — v10.0 SHIPPED to main 2026-07-04** (LoRaCam; `lora_cam_xiao` ff-merged → `main`@`8d0d39e`, annotated tag `v10.0` pushed). Draft GitHub release created (`RELEASE_NOTES_v10.0.md`, 12 bins in `dist/v10.0/`) — owner to review + publish (`gh release edit v10.0 --draft=false --latest`). Docs updated: README (woven LoRaCam section), CONFIG-USER-MANUAL §10, platformio.ini commented LoRaCam examples. **Next: OTAA** (own release); optional follow-ups: promote `BRIDGE_CUSTOM_REPEAT` to a portal option; the deferred AP-pass-change + Wave-5 soaks. |
| **Investigation branch** | `lr1121-phase1` |
| **Investigation branch** | `lr1121-phase1` |
| **Branch tip** | Check with `git rev-parse lr1121-phase1` |
| **Snapshot tag (shared with Seeed)** | `lr1121-bringup-2026-05-26` — mutable, force-push acceptable; bump after material commits |
| **Default build flag** | `LR1121_RX_AUDIT_RUN=0` in `platformio.ini` (clean state) |

## ⭐ LoRaCam — camera + LoRa-commanded edge radio (ACTIVE branch `lora_cam_xiao`, NOT merged)

A LoRa-commanded camera built ON the v9.0 bridge firmware: a XIAO ESP32-S3 **Sense** (OV2640 cam + mic +
microSD on the rear B2B 40-pin) + a **perimeter-pin Wio-SX1262** edge radio. Commanded over LoRa — encrypted,
sender-whitelisted, replay-protected — and (later) a SoftAP web portal with live video + config + messaging.
Branch `lora_cam_xiao` off `main`@`24528ad` (v9.0). Design of record: `LORACAM-SPEC.md`; bench guide
`BENCH-CAMC2.md`; HW handoff `C:\Users\6r4yh\XIAO-Sense-Wio-SX1262-Compatibility-Handoff.md`.

**Locked design (LORACAM-SPEC §9):** SoftAP portal · per-sender PSK + allowlist auth · microSD on the shared
SPI bus (spiMutex) · binary C2 on a `PROTO_CUSTOM` radio (sync **0x33**) · encrypt-then-MAC reusing
`LoRaWANCrypto` (AES-CTR + 8-byte CMAC, keys derived from the PSK) · the cam is an **edge node of the repeater
mesh** (a repeater raw-repeats Custom frames → commander→repeater→cam works for free); three roles = edge cam
(responder) / repeater = commander (ships dormant in the standard build, ABP-encoder pattern) / standalone
self-mastered (cam's own portal). Two deployment modes (standalone vs paired) chosen at provisioning.

**HW (handoff, confirmed in-firmware):** the camera occupies the B2B pins R1 uses (GPIO38-42) → a camera build
MUST disable R1 and use only the EDGE radio R2 (V1.0 map NSS=GPIO5/DIO1=GPIO2/RST=GPIO3/BUSY=GPIO4 + SPI
D8/D9/D10). microSD CS=GPIO21 shares the SPI bus. TX capped **+20 dBm**. Mechanical: the Wio baseboard and the
Sense daughterboard both want the XIAO underside → **flying-leads prototype**. **Wiring CONFIRMED on silicon**
(R2 reads its chip ID, `[Radio2-Edge] ready sync 0x33`).

**✅ Phase 1 DONE + PROVEN ON SILICON (2026-06-28) — the C2 security core.**
- New: `src/CamC2.{h,cpp}` (frame `[ver0xC2][type][senderId:4][recipientId:4][seq:4][ct][cmac:8]`,
  encrypt-then-MAC, constant-time tag compare, fail-closed boot self-test `ready()`, responder + commander +
  beacon), `src/CamC2Config.{h,cpp}` (`c2auth` NVS whitelist + persist-on-accept fail-closed anti-replay +
  reboot-safe block-reserved tx seq), `tools/cam-c2.py` (offline gen/verify/selftest, reuses lw-verify crypto).
  Shared-code touch = ONE guarded hook in `ingestAndFanout()` (PROTO_CUSTOM branch, before the LW-ENCODE block)
  + `CamC2::begin(camC2Emit)` in setup + the `camC2Emit` seam (dedup-record + `g_routeQ` push). Everything
  `#if defined(BRIDGE_ROLE_CAMERA)||defined(BRIDGE_CAM_COMMANDER)` → **stock builds byte-identical (do-no-harm

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GrayHatGuy/Xiao-esp32s3-lora-repeater](https://github.com/GrayHatGuy/Xiao-esp32s3-lora-repeater) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
