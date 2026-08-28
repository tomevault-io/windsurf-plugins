---
trigger: always_on
description: Philips 49PUS7503/12 (TPM171E / MT5891) root work. This file is a session-to-session handover note.
---

# CLAUDE.md

Philips 49PUS7503/12 (TPM171E / MT5891) root work. This file is a session-to-session handover note.

## Goal

Install permanent root on the TV with Magisk. **Custom ROM is NOT a goal** — there is no AOSP/LineageOS port for TPM171E and one is not feasible (MT5891's panel driver, DVB tuner, Ambilight controller are closed blobs; no BSP published). The bootloader is not locked, but there is no ROM to install. Realistic ceiling: **root + `/system` write access**.

## Hardware identity (read live from the TV, not guessed)

| Field | Value |
|---|---|
| Model | Philips **49PUS7503/12** (2018, Ambilight) |
| Chassis | `ro.product.model` = **TPM171E** |
| Board | `ro.product.device` = **PH7M_EU_5596** |
| SoC | `ro.board.platform` = **mt5891** |
| Android | 8.0.0 (`OC release-keys`), `os_type: MSAF_2018_O` |
| Firmware | `ro.tpvision.product.swversion` = **TPM171E_R.107.001.143.000** |
| IP | `<TV-IP>` (Mac: `<PC-IP>`) |
| NetTV | 8.0.2 |

🟢 **This identity is critical:** the SoC is the **same chip** as yath's research device (65OLED873); the firmware version is **exactly the same** as the one CRHASH rooted on the 65PUS8602/12 (`107.1.143.0`). So this is not "it worked on a similar device" — it is the same chip + the same firmware.

## Getting a shell on the TV — telnet, NOT ADB

🔴 **ADB does NOT work over the network, do not bother.** Port 5555 is open, `adbd` is up, it sends an AUTH request — but the TV **never shows** the authorization dialog on screen (on Android TV 8 the dialog is only triggered over USB). Five different approaches were tried, all ended `unauthorized`:

- `adb kill-server` + reconnect
- Regenerating the adb keypair from scratch (old key backup: `~/.android/adbkey-backup-20260825-233958/`)
- "Revoke USB debugging authorizations" on the TV + ADB toggle
- Bringing the TV to the home screen, verifying developer options

**Working path:** `com.waxrain.telnetd` installed from the Play Store on the TV, gives a telnet shell on **port 23456**. Raw `nc` is not enough (telnet does IAC negotiation) → `tools/tvsh.py` handles that:

```bash
python3 tools/tvsh.py <TV-IP> 23456 "cmd1" "cmd2" ...
```

Shell uid is `u0_a105` (untrusted_app) — not root, but `/system` is world-readable so reading was sufficient.

Tools available on the TV: `base64`, `dd`, `md5sum`, `toybox` (includes `nc`, `tar`). **No `unzip`** — and `nc` is not on PATH, call it as `toybox nc`.

File pull pattern (verified):
```bash
(nc -l 9999 > file &) ; sleep 1
python3 tools/tvsh.py <TV-IP> 23456 "toybox nc <PC-IP> 9999 < /remote/path"
```

## Done ✅

1. **`recovery-resource.dat` pulled** — `/system/etc/recovery-resource.dat`, 1,827,876 bytes, `md5 b0720616ddb96a6f3cc851bc48ea3df0`. Transfer verified byte-perfect. → `keys/recovery-resource.dat`
2. **`keyfile.txt` extracted** (from the ZIP, 256 bytes) → `keys/keyfile.txt`
3. **`passfile.txt` generated** — `dd if=keyfile.txt of=passfile.txt bs=127 count=1`, 127 bytes → `keys/passfile.txt`
4. **`tools/unpack-firmware`** downloaded (yath's script). The script looks for `passfile.txt` **in its own directory** → `tools/passfile.txt` was a symlink to `../keys/passfile.txt`.
5. **Firmware downloaded** — `firmware/update.zip`, **1,320,163,695 bytes**, contains a single file `autorun.upg`. Source: `https://firmware.nettvservices.com/files/6/65oled903_12/65oled903_12_fus_aen.zip` (toengel.net firmware table, 2018/TPM18.1 row — the model list explicitly includes **7503**).
6. **Firmware decrypted** — `firmware/update/` tree ready. **`boot.img` obtained**: 12,526,888 bytes, `md5 dacc5c4babf49e78e35fefc86bdc6e8a`, valid `ANDROID!` image (kernel 4,708,896 B + ramdisk 7,811,293 B, page 2048, cmdline `buildvariant=user`).
7. **Rescue files downloaded** → `firmware/rescue/` (see below).
8. **Magisk v25.2 patch applied** → `magisk/boot-patched.img`. Details below.

🟢 **The firmware is proven to be the correct one.** The decrypted `system/build.prop` matches the identity read live from the TV exactly:
`ro.product.model=TPM171E`, `ro.product.device=PH7M_EU_5596`, `ro.board.platform=mt5891`, `ro.tpvision.product.swversion=TPM171E_R.107.001.143.000`.
So `firmware/update/boot.img` = the boot image the TV is **currently running**.

🔴 The three files under `keys/` are **secrets specific to this TV**, in `.gitignore`. Do not commit, do not share.

## Firmware download — VPN required

🔴 **toengel.net does not open from Turkey** (TLS handshakes, then the connection stalls). **With an Amsterdam VPN it returns HTTP 200** — geo-block confirmed. Enable the VPN before downloading firmware/loaders.

There are two different pages, both needed:
- [firmware-download](https://toengel.net/philipsblog/firmware-download/) — current firmware table (zip containing `autorun.upg`)
- [firmware-archiv](https://toengel.net/philipsblog/firmware-archiv/) — old versions **+ upgrade_loaders** (Google Drive links)

Large Google Drive downloads require a confirm token and **can be silently truncated** (first attempt got 1.67/2.16 GB, no EOCD). Range requests are supported, `curl -C -` resumes. Verify the size from `content-range`, confirm with `unzip -l`.

## Rescue net ✅ — `firmware/rescue/`

| File | Size | Note |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saidsurucu/philips-tpm171e-root](https://github.com/saidsurucu/philips-tpm171e-root) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
