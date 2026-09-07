---
trigger: always_on
description: Home Assistant custom integration for **JUNG HOME** (HACS). It talks to a local
---

# Repository guide

Home Assistant custom integration for **JUNG HOME** (HACS). It talks to a local
JUNG HOME Gateway over its REST API and WebSocket.

## Layout

- `custom_components/junghome/` — the integration.
  - `__init__.py` — setup/unload, one-time stable-ID registry migrations,
    stale-device pruner, area auto-assignment, capability-change reload.
  - `coordinator.py` — REST poll (default 60 s, options-configurable) +
    WebSocket push and commands. The WS
    `functions` broadcast (the authoritative device list, sent on connect and
    on change) is adopted exactly like a poll result, so device add/remove is
    push-driven; the poll is the backstop.
  - `config_flow.py` — zeroconf + manual setup (app-approval or network-key
    password), reauth (confirm form first — registration opens the gateway's
    single 180 s approval window the moment it runs), reconfigure, options
    (REST poll interval, inverted covers).
  - `const.py` — `DOMAIN` and the stable-ID helpers (`device_slug`,
    `datapoint_suffix`, `stable_unique_id`, `duplicate_slugs`,
    `scene_unique_id`, `is_presence_quantity`).
  - `light.py`, `switch.py`, `sensor.py`, `binary_sensor.py`, `event.py`,
    `cover.py`, `climate.py`, `scene.py` — platforms; each discovers devices
    added at runtime via a coordinator listener.
- `tools/ws-capture/capture_ws.py` — read-only WS capture + analysis tool.
  Records frames **with timestamps** and walks the user through a scripted
  gesture set (`--script rocker` / `cover`), then `analyze` derives per-gesture
  edge sequences, channel-echo detection and the timing bounds the button
  blueprint's defaults rest on. This is how the two evidence-blocked backlog
  items get unblocked; the old `disk_dump/ws-capture*/` dumps have no timing.
- `blueprints/automation/junghome/button_gestures.yaml` — shipped blueprint
  deriving single/double/hold from raw press/release edges. Imported by URL;
  **not** distributed by HACS (HACS only installs `custom_components/`).
- `docs/` — reverse-engineered gateway reference (see below) plus
  `docs/example-button-automation.md` (user-facing guide).
- `config/`, `docker-compose.yml`, `scripts/` — local test harness.
- `disk_dump/` — gateway microSD dumps + live WS captures, **gitignored**
  (tokens + mesh keys; never commit it). Two dumps of the same card:
  `jung/` (2026-06-13, `sdc*`) and `jung-20260801/` (`sdb*`) — same builds
  byte-for-byte, but the 2026-08-01 extraction is higher fidelity (see its
  `NOTES.md`). **Quote evidence from `jung-20260801/sdb2`** (current
  firmware, v2.1.3 build 2840, API 1.5.0; `jung/sdc2` is the same build);
  `sdb3`/`sdc3` are the older v2.0.0 A/B partition — evidence found *only*
  there is stale (v2.1.3 refactored the middleware into
  `models/device_states/*State.js`). `ws-capture/` and `ws-capture-20260727/`
  are live production WS sessions; the data partition's live mesh DB is
  `sd?4/middleware/res_6/` (the unnumbered `res/` is empty factory state).

## Protocol facts the platforms encode (all firmware-verified)

- Function-type → platform: `OnOff`/`DimmerLight`/`ColorLight` → light;
  `Socket` → switch + sensor; `Measurement` → sensor + binary_sensor;
  `Position`/`PositionAndAngle` → cover; `Thermostat` → climate;
  `RockerSwitch` → event + switch (status LED). Rockers report only raw
  `pressed`/`depressed` edges (`up_request` / `down_request`, one datapoint
  per physical side — **not** alternating channels per press; that earlier
  belief was refuted by a timestamped capture).
- **On current DEVICE firmware, one tap is reported as TWO press/release
  pairs — same channel; a hold as ONE.** Labelled capture (2026-08-02, 16
  taps + 5 holds): tap pulse 0.40–0.53 s (near-constant — the device's
  reporting granularity, not the finger), hold pulse 2.44–3.11 s, intra-burst
  gap 0.11–1.03 s. Single vs double click is **indistinguishable** (both = 2
  identical pairs, overlapping gap ranges); tap vs hold separates perfectly on
  **pulse width** (5× empty band). **This is a regression**: the gateway's own
  archived logs (2026-06-20→07-28, ~450 bursts) show 1.00 presses/burst on
  the same buttons; gateway fw unchanged across the window, JUNG app went
  2.1.0→2.2.0 (app 2.2.x updates device firmware — issue #66). Mechanism
  unestablished — do NOT present it as BT-Mesh retransmission (gaps up to
  1 s refute that); gesture logic must tolerate both one and two pairs per
  tap. A duplicate-suppression window must be **≥ ~1.2 s** (earlier
  0.15–0.25 s guidance came from a mis-segmented unlabelled capture —
  refuted). Evidence + tables in docs/gateway-websocket.md.
- **Every button gang exposes BOTH `up_request` and `down_request`**, even a
  single-action one: the firmware's `JungHome_PushButton` model always creates
  PushedUp + PushedDown + StatusLed states. The gateway knows the difference
  (a `KeyMode` property) but that is category `manufacturer_property`, which
  `getDatapointTypeByState` never turns into an API datapoint — and JUNG's own
  code carries a `// TODO: set visibility here based on mode` for exactly
  this. So a single-action gang unavoidably gets one dead event entity; there
  is nothing on the wire to suppress it with. Multi-gang panels report each

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ernetas/junghome](https://github.com/ernetas/junghome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
