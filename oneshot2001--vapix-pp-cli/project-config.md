---
trigger: always_on
description: Every VAPIX call you'd reach for, plus a local SQLite history, fleet sweep, and an MCP server no other Axis wrapper has. Trigger phrases: `fingerprint axis camera`, `discover axis cameras on subnet`, `drive PTZ camera to preset`, `list ACAP applications`, `what changed on this camera`, `use vapix`, `run vapix`.
---


# VAPIX — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `vapix-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install vapix --cli-only
   ```
2. Verify: `vapix-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails before this CLI has a public-library category, install Node or use the category-specific Go fallback after publish.

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

A single Go binary for fleet operations against Axis IP cameras. It absorbs every operation in the open VAPIX wrapper landscape (param.cgi, PTZ, applications, analytics, events, MQTT bridge), keeps a per-camera SQLite history of parameters and presets, and exposes an MCP server so an LLM agent can sweep a subnet, drive a PTZ camera to a preset, or diff parameter trees from a conversation. Built around the quirks that bite real Axis fleets: the Q6325-LE basicdeviceinfo digest loop, the AXIS OS 12 ONVIF/VAPIX user-database split, and the AOA + analytics-MQTT surfaces that landed in OS 12.2.

## When to Use This CLI

Reach for vapix-pp-cli when you have a fleet of Axis cameras and a question whose answer requires hitting more than one of them, or when an LLM agent needs to script camera operations. Don't reach for it for live RTSP video pull (use VLC/ffmpeg) or for ONVIF-only Profile S/T flows (use swift-onvif or the ONVIF Device Manager).

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Fleet operations
- **`discover`** — Sweep a CIDR range and fingerprint every Axis camera it finds in one parallel shot.

  _When an installer drops you on a network with 'somewhere between 4 and 40 Axis cameras', this turns 'go figure out the inventory' into one command._

  ```bash
  vapix-pp-cli discover 192.168.1.0/24 --json
  ```
- **`sweep param`** — Run the same param.cgi query against every camera in the local store and return one table.

  _'What hostname is each camera reporting?' takes one command instead of an ad-hoc loop you re-write every audit._

  ```bash
  vapix-pp-cli sweep param --group root.Network.HostName --json
  ```

### Local state that compounds
- **`param diff`** — Diff the current param.cgi tree of a camera against the previous snapshot stored in the local SQLite cache.

  _Answers 'did anything change on this camera in the last week?' without trusting whoever last touched the on-camera config._

  ```bash
  vapix-pp-cli param diff --host 192.168.1.33 --since 7d --json
  ```
- **`preset copy`** — Read presets from one camera and write them to another using their stored coordinates.

  _Replacing or duplicating a PTZ rig stops being a 30-minute click-through and becomes one line._

  ```bash
  vapix-pp-cli preset copy --from Q6358 --to Q6325
  ```
- **`applications drift`** — Diff the installed ACAP application set per camera against last sync; flag installs, uninstalls, version bumps.

  _Catches 'who installed an unsigned ACAP on the front-door camera' without manually walking each device._

  ```bash
  vapix-pp-cli applications drift --json
  ```

### Compound operator workflows
- **`ptz tour`** — Walk a sequence of named presets with a configurable dwell at each.

  _Manual rounds for an integrator becomes scriptable; useful for demos and for late-night perimeter sweeps._

  ```bash
  vapix-pp-cli ptz tour --host 192.168.1.33 --presets Gate,Lobby,Lot --dwell 8
  ```
- **`events watch`** — Tail the MQTT bridge for a topic and shell out to a snapshot command when it fires.

  _Captures evidence frames the second a camera detects motion — without standing up a separate event broker._

  ```bash
  vapix-pp-cli events watch --host 192.168.1.33 --topic tns1:VideoSource/MotionAlarm --on-fire 'snapshot.sh {topic} {ts}'
  ```
- **`doctor`** — Composite health check — param reachability, PTZ whoami, ACAP app list, system clock skew, current firmware.

  _Pre-flight before a demo or an installation handoff in one command._

  ```bash
  vapix-pp-cli doctor --host 192.168.1.33 --json
  ```

### Reachability mitigation
- **`device info`** — Probes param.cgi first (works on Q6325-LE where basicdeviceinfo.cgi loops on digest 401) then falls back to richer endpoints for newer OS.

  _Avoids the silent 401-loop failure mode that has bitten every team that wrote 'a quick Python script' against a Q6325-LE._

  ```bash
  vapix-pp-cli device info --host 192.168.1.32
  ```

## Command Reference

**aoa** — AXIS Object Analytics (AOA) and analytics-metadata configuration

- `vapix-pp-cli aoa aoa-config` — Get AXIS Object Analytics configuration (scenarios, triggers, classes)
- `vapix-pp-cli aoa aoa-supported` — List supported scenario types and capabilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oneshot2001/vapix-pp-cli](https://github.com/oneshot2001/vapix-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
