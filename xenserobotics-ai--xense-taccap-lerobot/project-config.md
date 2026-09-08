---
trigger: always_on
description: Fork of HuggingFace `lerobot` (tracks upstream v5.1), slimmed to the
---

# lerobot-xense — Claude working notes

Fork of HuggingFace `lerobot` (tracks upstream v5.1), slimmed to the
**TacCap-Gripper** (**TacCap** = _Tactile Capture_ Gripper) — a handheld **UMI**
leader gripper for tactile data collection (single + bimanual) — and the
**Pico4** teleoperator/tracker, with Xense tactile cameras and an optional
Pico headset camera. See `src/lerobot/robots/taccap_gripper/README.md` for usage.

## TacCap serial / topology rules (device auto-discovery)

Devices are auto-discovered and assigned to `left`/`right` by serial + USB
topology rules — **no serials are hand-listed**. Source of truth:
`src/lerobot/robots/taccap_gripper/serial_discovery.py`.

### Serial grammar

- Gripper : `TCGU01<batch><line><seq><m|s>` — e.g. `TCGU01A24Z0002m`
- Tactile : `GSPS01<batch><line><seq>` — e.g. `GSPS01A25Z0011`
- Camera : `XC<batch><line><seq><m|s>` — e.g. `XCA24Z0007m`
- `<seq>` is 4 digits; patch `m` → leader, `s` → follower.

### Side rule — 单左双右 (`side_of_sequence`)

The **last digit** of the 4-digit sequence: **odd → left, even → right**.
Applies to gripper / camera side and to tactile _finger_ (below).

### Tactile left/right → `{side}_tactile_{left,right}`

Combines USB topology with the side rule (`discover_tactiles_by_hub`):

- **side** (which gripper a sensor pair belongs to): the two GSPS sensors
  sharing a gripper's **USB hub** are that gripper's pair. The gripper's side is
  read from its **firmware SN** over the wire (`scan_grippers()` → `ep.side`,
  i.e. `Cmd::GetSn`) — **NOT** the CH343 `mcu_serial`. So: hub → gripper → side.
- **finger** (left/right sensor on that gripper): the GSPS serial's **last
  digit** (单左双右).
- Hubs are matched via `/dev/v4l/by-path` (tactiles) ↔ `/dev/serial/by-path`
  (gripper `mcu_device`); a device's hub = its USB port path minus its own port.
- Needs the gripper SDK scan, so tactile discovery runs at **construction**
  (grippers must be powered then), keeping the obs schema ready before
  `connect()`.

### Pico4 motion tracker — different serial system

Tracker serials (e.g. `PC2310MLL3200496G`) are **not** Xense serials. Side is
the **second-to-last digit**: odd → left, even → right (`pico_tracker_side`),
e.g. `…496G` → `6` → right. Trackers enumerate from the XenseVR PC service at
connect (pin with `--robot.{left_,right_,}tracker_serial=<SN>` to bypass).

### Dataset provenance — `meta/hardware.json`, not `robot.id`

`--robot.id` is the station label (`taccap_0`, `taccap_1`; one per rig) — it
reaches the logger prefix, the calibration filename and `str(robot)`, and is
**not a dataset column** (`LeRobotDataset.create` takes only `robot_type`).
Unlike upstream's optional `RobotConfig.id` it is **required**: both TacCap
configs run `validate_robot_id()` in `__post_init__`, so a missing/blank id
fails at CLI-parse time instead of a rig recording anonymously. Enforced there
rather than by changing the base dataclass, which is upstream's.
Identity travels in `meta/hardware.json`, written by `lerobot-record` right
after `connect()` from `robot.hardware_manifest`: per unit, the gripper's
**firmware** SN (`Cmd::GetSn`, not `mcu_serial`) plus its tactile serials, each
tagged with `side` (which gripper), `finger` (which sensor on it) and the
`observation_key` it feeds. Keep it a separate file — `meta/info.json` is
upstream's schema and a fork-local key there collides on the next v5.x sync.
Helpers live in `taccap_gripper/common.py`.

The file is a list of **`epochs`**, not one flat `units`: each carries
`from_episode` / `to_episode` (half-open, matching `dataset_from_index` /
`dataset_to_index`) and `recorded_at`, so a rig swapped **mid-dataset** closes
the open epoch at the current episode count and opens the next one. It used to
warn and keep the original file; that warning went to the log and never reached
the dataset, so afterwards nothing on disk said the rig had changed while the
manifest quietly misattributed every episode recorded after the swap. Only a
`robot_type` mismatch is still keep-and-warn — single vs bimanual changes the
observation keys, so it is not the same dataset and epochs do not model it.
A pre-epoch file reads back as one open epoch (`manifest_epochs`), but an open
single epoch means _"nothing here says the rig changed"_, **not** _"it didn't"_.

**`robot_id` is the exception: a wall, not an epoch boundary.** One dataset is
one station, so `--resume` with a `--robot.id` the manifest disagrees with
raises (`check_dataset_station`) — identical `units` do not make it one rig,
because the label names the seat, not the hardware in it. Checked twice: in
`lerobot-record` **before `connect()`** (the id comes from the config, so no
device has to spin up to be turned away) and again inside
`write_hardware_manifest`, the choke point every writer goes through. Being a
dataset-level invariant it is written at the top level beside `robot_type` and
repeated per epoch so readers of older files keep working; `manifest_robot_ids`
reads both. An unlabelled dataset (recorded before `--robot.id` was required) is
not a mismatch — same reading as the open epoch above.

Tactile **runtime bundles are no longer recorded** (they used to sit at
`meta/runtimes/<serial>-<time>.bin` with a `runtime` key per sensor). The only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XenseRobotics-AI/xense-taccap-lerobot](https://github.com/XenseRobotics-AI/xense-taccap-lerobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
