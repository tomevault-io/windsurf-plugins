---
trigger: always_on
description: Home Assistant custom integration for Onesti/Nimly smart locks via ZHA. It
---

# Onesti Lock: agent guidelines

Home Assistant custom integration for Onesti/Nimly smart locks via ZHA. It
tells **who** unlocked the door and **how**. ZHA's stock quirk decodes the same attribute
only into raw numbers.

## Critical rules

1. The domain is `onesti_lock`, and our own classes and types follow it (`Onesti*`). `Nimly` is kept only where it means the vendor's brand, such as model strings and the app docs. `tests/test_domain_contract.py` holds every path and file name to the domain.
2. Credentials, API keys and secrets do NOT go in git. They belong in `secrets.md` (gitignored). Docs hold API URLs and technical references only, never secrets.
3. The lock is a battery-powered Zigbee EndDevice that sleeps. Every ZCL command goes through `ZhaLockTransport.send()` in `zha.py`, which handles the timeout and the auto-wake, and `send()` calls `cluster.command()` on the zigpy cluster directly. It must never go through ZHA's `issue_zigbee_cluster_command` service: Home Assistant records every `call_service` event with its data, so the service writes each PIN to the recorder in clear text. `tests_ha/test_pin_canary.py` checks for that. The direct call is why the transport looks more involved than the official API; do not "simplify" it back.
4. The Nimly response quirk (`IndexError` when the lock's answer is read) is expected on HA 2025.x. The command reaches the lock despite the error, so `send()` counts it as delivered. Do not "fix" it. The source is zha 0.0.x reading `response[1]` from a one-field Set PIN Code Response, fixed in zha 2.x; see "Nimly response quirk" in `docs/technical.md`.
5. The repo is public and written in English: code, comments, docs and commit messages. Work is tracked outside the repo, so no tracker ids or tracker names in files or commits. GitHub issue numbers (#6) are fine.
6. Vendor manuals are the source for slot rules and lock behaviour. Run `python3 scripts/fetch_manuals.py` once, then read the `.txt` extracts in `docs/manuals/`. The same script fetches the vendor's 2021 Zigbee spec, two 2021 Zigbee sniffs, a Z2M log and a ZHA diagnostics dump plus debug log from a Code Pro nobody here owns; `docs/zigbee-protocol/zigbee-captures.md` says what each of them answers. The files are gitignored, and `docs/manuals/README.md` lists what exists and where it came from. `pdftotext` drops footnotes from some of the PDFs, so render the page before concluding that a sentence is not there.
   - The same README's "Living sources" table is the archive of what other people have written: every forum thread, every GitHub issue and PR in the four upstream projects, the converter and quirk sources, and the vendor's pages and app listings. `python3 scripts/fetch_manuals.py --living` refetches them all, says which ones grew since last time, and writes the new date and size into the table, so `git diff` on that file is the answer to "what is new". `--check` reports without writing. Read the archive before searching the web again, and say in `docs/community-reports.md` what the reading found.

## Architecture

Three layers, each described in full in `docs/technical.md`; the file names below are the map.

- **Entry lifecycle and ZHA access**: `__init__.py` (setup, migration, services, ZHA watch, repair issue), `zha.py` (`ZhaLockTransport`: cluster lookup, `send()` and its `SendOutcome`, `wake()`, wake echo, capability read), `entity.py` (the device and unique ids, keyed on the entry id). Sections "Reaching the lock through ZHA", "Sending commands" and "Auto-wake mechanism".
- **Coordinator, events and sensors**: `coordinator.py` (one `OnestiCoordinator` per lock on `entry.runtime_data`, slot storage in `entry.options`, PIN operations, capabilities), `events.py` (attrid 0x0100 decoding, the two zigpy listener hooks, the system-lock rule), `sensor.py` (slot row, restored activity sensor, three capability sensors off by default). Sections "How user identification works", "Listening for reports" and "Coordinator pattern".
- **BLE library**: `ble/` (protocol, crypto, client; never run against a lock) and `bluetooth.py`, which nothing imports yet. Layers and the rules the code keeps are in `docs/nimly-ble-app/ble-library.md`; the Home Assistant side is "Reaching the lock over Bluetooth" in `docs/technical.md`.

`SOURCE_MAP` in `events.py` is the canonical decoder of the source byte in attrid 0x0100; `docs/zigbee-protocol/zigbee-captures.md` has the table with what verified each value. Session notes and old plans contain earlier wrong guesses. The code is authoritative.

## Key files

| File                                           | Purpose                                                                                                          |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `custom_components/onesti_lock/__init__.py`    | Services registered in async_setup, migration, entry setup/unload, repair issue, ZHA watch, update listener      |
| `custom_components/onesti_lock/coordinator.py` | Slot storage, PIN operations, reserved-slot guard, lock capabilities                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fredrik-lindseth/onesti-lock](https://github.com/fredrik-lindseth/onesti-lock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
