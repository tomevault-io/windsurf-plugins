---
trigger: always_on
description: A Home Assistant add-on that impersonates the PetKit cloud: a litter box, feeder or fountain
---

# petkit-local

A Home Assistant add-on that impersonates the PetKit cloud: a litter box, feeder or fountain
is pointed at it instead of PetKit's servers (the Pura Air purifiers are BLE-only and reach it
through a parent device), and petkit-local answers as the official API, stores events and media
locally, and publishes entities via MQTT discovery.

A device speaks HTTP **or** MQTT, not both: it starts on HTTP and stops polling the heartbeat once
it reaches the broker. Neither transport is dedicated to a kind of message — see `ARCHITECTURE.md`.

It is an add-on **repository** — `repository.yaml` at the root, the add-on in `addon/`, the package
in `addon/petkit_local/`. It also runs as a plain container or bare process (`docker-compose.yml`
at the root, or `--no-ha`); HA Container and HA Core have no add-on system, so that path is
supported, not a fallback.

**Stack.** Python 3.11+, one asyncio loop, one container. aiohttp (device API, bucket, panel), amqtt
(embedded device-facing broker), aiomqtt (client for HA's broker), SQLAlchemy 2.0 async + aiosqlite
(`{data_dir}/petkit.db`), Jinja2, ffmpeg. Device identity and settings persist as atomic JSON
(`devices.json`, `ble_devices.json`). `README.md` credits the projects the payloads came from.

## Orientation

`ARCHITECTURE.md` is the map: what each package owns, and how a device request travels through
them. Read it first — this file assumes it.

Two rules about placement that the map does not make obvious:

- `devices` depends only on `events` and `utils`. "Which entities does this device publish" is an
  HA question, answered in `ha/categories.py`. Do not answer it from `devices/`.
- `events/codes.py` and `devices/payloads.py` carry almost all the reverse-engineered protocol and
  are deliberately large. Everything below is about not breaking them.

## Invariants — do not break these

- **All protocol knowledge lives in `events/codes.py`** — seven namespaces, each row graded
  (`confirmed`/`inferred`/`unverified`/`conflicted`) and carrying the firmware function behind it.
  Add a code there, not in a private set somewhere: the nine parallel collections this replaced are
  exactly why seven real codes were classified by none of them. `events/decode.py` renders values;
  `events/normalize.py` only does transport.
- **HTTP `event_type` codes are PER DEVICE CATEGORY, not global.** Code `2` is `err_over` on a
  litter box and `feed_over` on a D4H feeder. Always pass `device_type` to `codes.lookup` /
  `classify_event_kind` / `event_label`; omitting it assumes a litter box.
- **Two event-code namespaces, never merged.** Our `dev_event_report` codes are authoritative here;
  the cloud-record API's `LitterRecord.subContent[].eventType` ints overlap on 5/8/10 with
  *different* meanings, so only namespace-independent sub-field decoding (`result`/`start_reason`/
  `err`) is borrowed. NS2 stays quarantined in `codes.CLOUD_RECORD_TYPES`, read by nothing.
- **The capture wins on SHAPE, the firmware RE wins on MEANING.** Where they disagree the row is
  graded `conflicted` and its `note` says how — never pick silently. The repo's original labels
  were neither source: code `8` read "Cleaning started" but is the deodorizing cycle's completion.
- **Days are cut at LOCAL midnight** (`utils/timeutil.local_day_bounds`), and the end of a day is
  midnight-next-day re-localized, never `start + 86400` — DST makes days 23 and 25 hours long.
- **`event_id` is a SESSION key** shared by several distinct reports of one visit; it becomes
  `related_event`, and the dedup key is `event_id + event_type` — deduping on `event_id` alone
  loses every report of a session but the last.
- **`petEvent` alone is not a toilet visit** — only `toiletEvent` means the box was used. Code `20`
  is a pet-appeared episode (`petEvent=1`, `toiletEvent=0`, no weight, no cleaning cycle) that the
  app counts under "Pet", never "Toileting".
- **A state report is a FIXED 29-key dump plus 3 optional keys, and absence is the signal.**
  Measured over 1254 real snapshots across both transports: 29 keys in every single one, and only
  `workState` (166), `lightState` (166) and `refreshState` (32) come and go. Those three are sent
  ONLY while the thing they describe is happening and carry no "off" value — so they must be turned
  into a real 0/1 (`state_parsers.PRESENCE_FLAGS`), because `device.state` is merged into and never
  pruned and a key that stops arriving keeps its last value forever. `refreshState` is also an
  OBJECT, and a non-empty dict is truthy, so a binary sensor bound straight to it latches on at the
  first spray and never returns. Gate any such derivation on `SNAPSHOT_MARKER`: reading absence as
  "off" is exactly as wrong as reading presence as "on" if the payload was never going to carry the
  key. Same rule gives `workState` — a default of 0 there is `WORK_MODES[0] == "cleaning"`, which
  had an idle box reporting itself as cleaning 79% of the time.
- **Every MQTT `params` carries the transport envelope with the telemetry** — `XDevice`, `event_id`,
  `timestamp`, `content`, `state` (`ingest.MQTT_ENVELOPE_KEYS`). `XDevice` is the signed request
  credential. Strip it with `telemetry_only()` before anything merges `params` into `device.state`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex-so-3/petkit-local](https://github.com/alex-so-3/petkit-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
