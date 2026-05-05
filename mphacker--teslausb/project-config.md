---
trigger: always_on
description: Focused tips to make safe changes quickly. This is a Raspberry Pi USB gadget project (dual-LUN mass storage) with strict mount/namespace rules and YAML-based configuration.
---

# TeslaUSB AI Coding Guide

Focused tips to make safe changes quickly. This is a Raspberry Pi USB gadget project (dual-LUN mass storage) with strict mount/namespace rules and YAML-based configuration.

These devices run in a vehicle; power can drop at any time. Prioritize atomic writes, fsyncs, and recovery paths to avoid corruption.

## Configuration System
- **Single source of truth**: `config.yaml` at repository root contains ALL configuration (paths, credentials, network settings, limits).
- **Bash scripts**: Read YAML via `yq` using `scripts/config.sh` wrapper (auto-sources config.yaml).
  - **Optimized loading**: Single yq call with eval statement (properly quoted for security) - saves ~1.2s per invocation.
  - **Security**: All values double-quoted in eval to prevent command injection from special characters.
- **Python scripts**: Read YAML via `PyYAML` using `scripts/web/config.py` wrapper (auto-loads config.yaml).
- **Never hardcode values**: Always read from config via the wrappers. Both `config.sh` and `config.py` are thin wrappers around `config.yaml`.
- After editing `config.yaml`, restart affected services: `gadget_web.service` (web/Python changes), `wifi-monitor.service` (AP changes).

## Architecture & Modes
- Two disk images: `usb_cam.img` (part1 TeslaCam drive) and `usb_lightshow.img` (part2 LightShow/Chimes drive).
- Modes: **present** (USB gadget active, RO mounts at `/mnt/gadget/part*-ro`, Samba off) vs **edit** (gadget off, RW mounts at `/mnt/gadget/part*`, Samba on). `state.txt` holds the token; `mode_service.current_mode()` falls back to detection.
- **Boot-time fsck**: When `disk_images.boot_fsck_enabled: true` (default), `present_usb.sh` runs `fsck -p` on both drives before presenting to Tesla (~1 second total). Auto-repairs minor filesystem issues.
- Always resolve paths via `partition_service.get_mount_path/iter_all_partitions` instead of hardcoding.

## Template System
- Source templates live in `scripts/` and `templates/` with placeholders `__GADGET_DIR__`, `__MNT_DIR__`, `__TARGET_USER__`, `__IMG_NAME__`, `__SECRET_KEY__`.
- After changing any template/script under those dirs, run `sudo ./setup_usb.sh` to substitute and deploy, then restart relevant services (e.g., `sudo systemctl restart gadget_web.service`). Never hardcode installed paths.

## Mount / Gadget Safety
- All mount/umount/mountpoint commands must be run in the PID 1 mount namespace: `sudo nsenter --mount=/proc/1/ns/mnt ...` (see `present_usb.sh`, `edit_usb.sh`).
- Switching to edit: unbind UDC first, remove gadget config, then unmount and detach loop devices; sync before and after.
- `partition_mount_service.quick_edit_part2` temporarily remounts part2 RW while in present mode; it uses `.quick_edit_part2.lock` (120s stale). Keep operations short and restore RO mount/LUN on all code paths.

## Loop Devices & USB Gadget LUNs
- **USB gadget serves image FILES directly**, not loop devices. The LUN backing file is the `.img` file path, not a `/dev/loopN` device.
- **Loop devices are for LOCAL mounting only** - they allow the Pi to mount and access the image file contents while the gadget serves the same file to the vehicle.
- **Multiple loop devices are normal**: The kernel may create 2-3 loop devices for the same image (one for local mount, others for internal gadget management). This is harmless.
- **Read-only loop devices cannot be mounted RW**: If a loop device is created with `-r` flag (read-only), you CANNOT mount it with `rw` options - the filesystem will still be read-only. Must detach and recreate without `-r`.
- **Cannot detach loop devices used by gadget**: If the gadget's LUN is backed by an image, any loop device for that image may be locked by the kernel. To safely edit, must temporarily clear the LUN backing file first.
- **quick_edit_part2 sequence**: Clear LUN1 backing → unmount RO → detach old loops → create RW loop → mount RW → do work → sync → unmount → detach → create RO loop → remount RO → restore LUN1 backing. Any shortcuts risk read-only filesystems or kernel locks.

## Web App Patterns
- Flask app under `scripts/web/`; blueprints in `scripts/web/blueprints/`; services in `scripts/web/services/` encapsulate logic (mount handling, chimes, thumbnails, Samba, mode).
- Mode-aware file ops: lock chimes/light shows/videos must go through services that choose RO/RW paths; avoid direct filesystem writes in view code.
- Samba cache: after edits in edit mode, call `close_samba_share()` and `restart_samba_services()` (see lock chime routes).
- **Web service runs on port 80** (not 5000) to enable captive portal functionality. The service runs as root (via systemd) to bind to privileged port 80.

## Feature Availability (Image-Gated UI)
- **Dynamic gating**: Nav items and routes are hidden/blocked when their required `.img` file doesn't exist. Uses per-request `os.path.isfile()` checks (negligible overhead, no restart needed).
- **Image path constants**: `IMG_CAM_PATH`, `IMG_LIGHTSHOW_PATH`, `IMG_MUSIC_PATH` in `config.py` — computed from `GADGET_DIR` + image name.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mphacker/TeslaUSB](https://github.com/mphacker/TeslaUSB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
