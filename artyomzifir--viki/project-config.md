---
trigger: always_on
description: Guidance for coding agents (and humans) working in this repository.
---

# AGENTS.md

Guidance for coding agents (and humans) working in this repository.

It has two parts:

1. **[Using the project](#part-1--using-the-project)** — how to run ViKi and get a
   dataset out of it.
2. **[Developing the project](#part-2--developing-the-project)** — how the code is
   laid out and the rules that keep it that way.

The user-facing overview lives in [`README.md`](README.md); hardware bring-up in
[`SETUP_GUIDE.md`](SETUP_GUIDE.md). Don't duplicate either here.

---

## What this is

ViKi turns multi-view RGB-D video of a human doing a manipulation task into a
robot-ready demonstration dataset. One directory per arrow of the pipeline; each
stage writes a durable artifact into an **episode directory**:

```
record     cameras/      live RGB-D  ->  episodes/<id>/raw/
extract    perception/   raw/        ->  rec.npz     per-camera hand landmark trajectories
prepare    prepare/      rec.npz     ->  cln.npz     fused + smoothed + palm pose + gripper
retarget   retarget/     cln.npz     ->  plan.h5     targets -> adapter -> TCP -> robot joints
replay     replay/       plan.h5     ->  replay.h5   proprioception on hardware        [stub]
label      labeling.py   ->  meta.json["labels"]     task / phase segments / outcome
export     export/       episodes/*  ->  datasets/<name>/
```

**There is no live pipeline.** Record scenes first, then extract/prepare/retarget
offline. `data/` and `models/` are gitignored — no recordings, calibration, URDFs
or weights are in the repository.

---

# Part 1 — Using the project

## Everything runs in Docker

The app talks to hardware SDKs (`pyrealsense2`, `libk4a.so`) installed in the
image, not on the host. Run and test through Docker Compose.

```bash
sudo ./scripts/host_setup.sh              # once per host: Docker, udev rules, groups
docker compose up --build                 # web UI + API on :8000 (then just: docker compose up)
docker compose run --rm terminal          # debug shell inside the container
docker compose run --rm test              # full test suite
docker compose run --rm cli <verb> ...    # one pipeline stage (viki <verb> ...)
```

One `docker-compose.yml`, one image (Dockerfile `test` target). Services: `web`
(default, `up`), and `test` / `cli` / `terminal` behind the `tools` profile,
meant for `run`. `test` and `cli` append their args to `pytest` / `viki`.

The server serves UI + API at `http://localhost:8000` (`network_mode: host`).
`viki/` is bind-mounted, so code edits apply on container restart — no rebuild
unless `pyproject.toml` changes.

Kinect has host-level prerequisites (GRUB `usbcore.usbfs_memory_mb=1000`,
`xhost +local:`, a separate 10 Gbps USB hub per Kinect, sync cable). Read
`SETUP_GUIDE.md` before touching camera bring-up.

## The CLI

```bash
viki record   ...                 # capture a synced RGB-D scene into a new episode
viki extract  <episode>           # raw/ -> rec.npz
viki prepare  <episode>           # rec.npz -> cln.npz
viki retarget <episode>           # cln.npz -> plan.h5
viki replay   <episode>           # plan.h5 -> replay.h5            [stub]
viki label    <episode> ...       # get/set episode labels
viki export   <episode>... --out <dir> [--format trajectory|lerobot]
viki run      <episode>           # extract -> prepare -> retarget -> replay
viki cloud    <episode>           # raw/ -> cloud/ (viewer artifact only)
viki hand-fit <episode>           # batch capsule-hand fit, appends hand_fit_* to cln.npz
viki viz      <episode>           # headless 3-D figure (rec|cln)
```

`export` defaults to `--format trajectory`: a self-contained `.npz` bundle plus a
JSON manifest, **numpy only**. `--format lerobot` needs `viki[export]` (lerobot,
torch) and stricter eligibility. See [`viki/export/README.md`](viki/export/README.md).

## The web UI

Tabs: Cameras, Calibration, Record, Extract, Viewer, Retarget, Export. The Extract
tab drives perception over one / several / a whole dataset of episodes through a
background FIFO job queue (`viki/server/jobs.py`, one worker).

## Configuration

All tunables live in `data/user_configuration.json`, copied from
`data/default_configuration.json` on first run. `viki/config.py` reads that file
**once at import** and injects every key into its module globals, so code does
`from viki.config import RETARGET_IK_SOLVER`.

Changing config at runtime means editing the JSON and restarting — the
`/api/config` routes plus `/api/restart` do exactly that.

## What you must supply yourself

Robot and gripper URDFs (fetched by `robot_descriptions` into `models/` on the
first retarget), hand-pose weights (MediaPipe/RTMPose auto-download; the
mmpose-heatmap ONNX files are user-converted), camera calibration, recordings,
and the config file. The README's
[What you must supply yourself](README.md#what-you-must-supply-yourself) table is
the authoritative list.

---

# Part 2 — Developing the project

## Package map

Nine packages under `viki/`, **each with its own README — read those**, they carry
hardware quirks and API tables not repeated here.

| package | role |
|---|---|
| `cameras/` | the only package that touches camera SDKs |
| `calibration/` | intrinsics + extrinsics (chessboard / ChArUco), a side input to perception |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artyomzifir/ViKi](https://github.com/artyomzifir/ViKi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
