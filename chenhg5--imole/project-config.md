---
trigger: always_on
description: |
---


# iMole — iPhone Storage Cleaner Skill

Help users slim down their iPhone storage. Scan media, back up files to the computer, delete verified files from the device, and guide users through the steps only Apple allows on the phone itself.

Use this skill whenever the user says: "my iPhone is full", "clean up my iPhone", "free up iPhone storage", "back up iPhone photos/videos", "iPhone space is running out", or asks to delete old videos from their phone.

---

## Platform Support

| Feature | macOS | Linux | Windows |
|---------|:-----:|:-----:|:-------:|
| USB scan (auto) | ✅ ImageCaptureCore | ✅ gphoto2 | ➖ use `--source` |
| Scan via `--source PATH` | ✅ | ✅ | ✅ |
| Backup (copy files) | ✅ | ✅ | ✅ |
| Delete from device (USB) | ✅ ImageCaptureCore | ❌ planned | ❌ not supported |
| Delete via mounted path | ✅ | ✅ (ifuse) | ✅ (iTunes mount) |
| Device detection (`doctor`) | ✅ | ✅ | ✅ |

### macOS prerequisites

- iPhone connected via USB, screen unlocked, "Trust This Computer" accepted.
- `swift` available: `xcode-select --install`
- ImageCaptureCore used automatically — no extra install needed.

### Linux prerequisites

```bash
# Device detection and trust pairing
sudo apt install libimobiledevice-utils

# Scan via USB (gphoto2 — auto-detected)
sudo apt install gphoto2

# Mount DCIM as filesystem (required for backup + delete workflow)
sudo apt install ifuse
idevicepair pair          # trust the device (one-time)
mkdir -p ~/iphone
ifuse ~/iphone            # mount iPhone DCIM
```

### Linux full cleanup workflow

```bash
# 1. Mount
ifuse ~/iphone

# 2. Scan what's on the phone
imole scan --source ~/iphone/DCIM

# 3. Backup (verifies every file with SHA-256)
imole backup --source ~/iphone/DCIM --to ~/backup/iphone --only videos --older-than 90d

# 4. Review what will be deleted
imole report --manifest ~/backup/iphone/manifest.json

# 5. Delete verified files directly from the mount (space freed immediately, no Recently Deleted)
imole clean --manifest ~/backup/iphone/manifest.json --source ~/iphone/DCIM

# 6. Unmount
fusermount -u ~/iphone
```

### Windows prerequisites

- Install **iTunes** (provides USB drivers and the Apple Mobile Device service).
- Connect iPhone, unlock it, tap "Trust This Computer".
- Open **Windows Explorer** → This PC → [Your iPhone] → Internal Storage → DCIM.
- Note the path (e.g. `\\Apple\iPhone\Internal Storage\DCIM`) or copy it to a local folder.
- Use `--source` with that path:

```powershell
imole.exe scan --source "\\Apple\iPhone\Internal Storage\DCIM"
imole.exe backup --source "\\Apple\iPhone\Internal Storage\DCIM" --to C:\backup
imole.exe clean --manifest C:\backup\manifest.json --source "\\Apple\iPhone\Internal Storage\DCIM"
```

---

## Installation

### Option A — npm (recommended — works on macOS, Linux, Windows)

```bash
npm install -g @getimole/imole
```

Works on all platforms with Node.js installed. Downloads the pre-built binary automatically. Verify with:

```bash
imole --version
```

### Option B — Install script (macOS / Linux)

```bash
curl -fsSL https://raw.githubusercontent.com/chenhg5/imole/main/install.sh | bash
```

Installs the pre-built binary to `/usr/local/bin/imole`. Verify with:

```bash
imole --version
```

### Option C — Build from source

```bash
git clone https://github.com/chenhg5/imole.git
cd imole
go build -o imole ./cmd/imole
sudo mv imole /usr/local/bin/
```

---

## Full Cleanup Workflow (end-to-end)

Follow these steps in order. Never skip the backup-and-verify step before cleaning.

### Step 1 — Check device and environment

```bash
imole doctor --json
```

Expected: `"device"` block with `"connected": true`. If the device is not connected, ask the user to plug in the iPhone, unlock it, and tap "Trust" on the device screen.

Use device storage pressure to decide how aggressive the cleanup plan should be:

| Free space | Pressure | Agent response |
|---:|---|---|
| `< 5%` | Critical | Target immediate reclaim. Prefer large videos and old videos first. |
| `5-10%` | High | Recommend a concrete backup target large enough to reach at least 15% free. |
| `10-20%` | Moderate | Offer a conservative plan: oldest/large videos first, then app cleanup guidance. |
| `> 20%` | Low | Diagnose only; do not push deletion unless user asks. |

Useful fields:

```bash
imole doctor --json --fields device.name,device.product_type,device.storage.total_data_capacity,device.storage.amount_data_available,device.storage.free_percent
```

### Step 2 — Scan to understand what's taking space

```bash
imole scan --summary --json
```

Key fields in the response:

| Field | Meaning |
|---|---|
| `device.storage.free_percent` | Remaining device storage percentage |
| `media.total_size` | Total media size (photos + videos) |
| `media.video_size` | Videos only — usually the biggest offender |
| `apps.total_size` | App storage estimate from iOS installation_proxy |
| `top_video.size` | Largest video candidate |

To show only the most useful subset:

```bash
imole scan --summary --json --fields device.storage.free_percent,media.total_size,media.video_size,apps.total_size,top_video
```

To focus on videos older than 90 days:

```bash
imole scan --summary --only videos --older-than 90d --json
```

If you already ran a scan recently and don't want to wait ~15 s for USB enumeration:

```bash
imole scan --cache --summary --json
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenhg5/imole](https://github.com/chenhg5/imole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
