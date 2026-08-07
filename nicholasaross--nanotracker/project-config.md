---
trigger: always_on
description: > **ARCHIVED — Superseded by [StreetTracker](https://github.com/nicholasaross/StreetTracker) as of 2026-07-07; read-only.**
---

> **ARCHIVED — Superseded by [StreetTracker](https://github.com/nicholasaross/StreetTracker) as of 2026-07-07; read-only.**

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

NanoTracker is the Jetson Nano (original, JetPack 4.6.1) deployment of the
VehicleTracker pipeline.  Same goal (detect, track, classify moving
vehicles from a video source; emit a sortable HTML + JSON summary),
different runtime constraints:

  - Python 3.6.9 (no dataclasses, no f-string `=`, no `list[X]`, no walrus)
  - PyTorch / Ultralytics are NOT installed -- inference runs through
    TensorRT 8.2 via pycuda
  - Input is a live RTSP stream (Reolink, H.264 sub by default) decoded
    by OpenCV's bundled FFmpeg backend, **not** GStreamer.  MP4 file
    inputs use GStreamer + NVDEC.  See "GStreamer / OpenCV / NVDEC pitfalls" below.
  - Output is metadata-only (JSON + HTML + thumbnails); no annotated video

## Critical compatibility rules

  - **Python 3.6 syntax only.**  `from typing import List, Dict, Optional`
    everywhere; never use `list[int]` / `dict[str, X]` / `Optional[X] | None`.
    `dataclasses` is unavailable -- use `NamedTuple` or `__slots__` classes.
  - **Never import torch / torchvision / ultralytics.**  They won't install
    on the Nano.  Inference is pure TRT + numpy.
  - **TRT engines are not portable** across GPU architectures.  Always
    build engines ON the Nano via `scripts/build_engine.sh`.  The ONNX
    file produced by `scripts/export_onnx.py` IS portable.

## Architecture

```
nano_tracker.py              -- entry; tracker + attributes + output + HTTP dashboard
  ├── trt_engine.py:TRTYolo  -- engine load, infer, YOLOv8 decode, NMS
  └── gst_source.py
      ├── GstRtspSource  -- live RTSP via cv2.VideoCapture(url, CAP_FFMPEG)
      │                    (class name is historical; no GStreamer involved)
      └── GstFileSource  -- MP4 via GStreamer NVDEC (nvv4l2decoder)
```

The runtime loop is single-threaded by design -- on a 4GB Nano, async
producer/consumer queues tend to cause OOM more often than they help.
If decode-latency hides inference time, add an explicit thread later.

## Dev environment & deployment

You'll typically be editing on the **Windows dev box** (`D:\Projects\NanoTracker\...`)
and SSHing to the **Nano** to test.  The Nano is a separate machine -- none of the
runtime (TRT, pycuda, aarch64 OpenCV, GStreamer NVDEC) works on Windows, so do
not try to `python nano_tracker.py` on the dev box.

  - **Nano host:** `nano` (resolved via mDNS / hosts file -- works regardless
    of DHCP).  Reolink camera is at `192.168.1.72`.  Do NOT hardcode the
    Nano's LAN IP anywhere: it's DHCP-assigned and has moved at least once
    (was `.181`, now `.119`); the hostname is the stable handle.  To get
    the current LAN IPv4 when you need one (e.g. for the dashboard URL):
    `ssh -i ~/.ssh/nanotracker_claude claude@nano "hostname -I | tr ' ' '\n' | grep '^192.168'"`.
  - **SSH:** `ssh -i ~/.ssh/nanotracker_claude claude@nano`.  The working
    account is `claude`; the private key is named exactly `nanotracker_claude`
    (no extension) in the dev box's `~/.ssh/`.  No `Host nano` alias exists in
    `~/.ssh/config` by default -- pass `-i` explicitly or add one.  When the
    Nano's IP changes you may get `Host key verification failed` if you SSH
    by the new IP (the hostname entry in `known_hosts` is fine; new IPs are
    new entries) -- use `-o StrictHostKeyChecking=accept-new` for the first
    connection by IP.
  - **Working dir on the Nano:** `/home/claude/NanoTracker/`.  This is **NOT a
    git repository** -- files were copied
    in directly.  Deploy changes via `scp` of individual files; do not attempt
    `git pull` on the Nano.  (Future cleanup: `git init` + add origin so this
    becomes a normal pull workflow.)
  - **Camera password lives in `camera_config.json`** on the Nano (mode 600).
    `nano_tracker.py` reads it directly -- no `REOLINK_PASSWORD` env var needed
    when launching from the Nano.  Never copy `camera_config.json` from dev to
    Nano (it's gitignored on dev and only the `.example` template is checked
    in -- the dev copy has no real password).
  - **Dashboard:** `http://<nano-lan-ip>:8080/` while a session is running
    (see above for getting the current IP).  The tracker prints a dashboard
    URL on startup but it's often `127.0.1.1` from `gethostbyname` on the
    Nano -- that's loopback, useless from outside; use the LAN IP.  Index
    page auto-redirects to the latest summary HTML.

### Sync code changes from dev box to Nano

```bash
# from the dev box, in your worktree
scp -i ~/.ssh/nanotracker_claude nano_tracker.py claude@nano:/home/claude/NanoTracker/
ssh -i ~/.ssh/nanotracker_claude claude@nano "wc -c ~/NanoTracker/nano_tracker.py"
```

Gitignored on the dev side and Nano-only: `camera_config.json`, `*.engine`,
`*.onnx`, `output/`.  TRT engines are not portable across GPU architectures --
always build on the Nano.

### Pull a capture batch from the Nano to the dev box

```bash
# from the dev box, in your worktree
python scripts/pull_session.py                  # latest session -> ./output/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicholasaross/NanoTracker](https://github.com/nicholasaross/NanoTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
