---
trigger: always_on
description: Self-contained build that takes the Sega "Dragon" Windows CE 2.12 SDK and adds a real networking
---

# CLAUDE.md — wince-dc (Windows CE on the Sega Dreamcast)

Self-contained build that takes the Sega "Dragon" Windows CE 2.12 SDK and adds a real networking
stack (Ethernet + modem + dead-game-server revival), a windowed desktop shell with apps (browser,
media player, file explorer, network diagnostics), and SPI transports (W5500, SD card) — then
bakes a bootable Dreamcast disc with the vendored CE image tools + a real SH-4 PE compiler.
Everything builds from this repo via **CMake**; nothing external is needed.

> The user is a low-level Dreamcast/OS-porting expert — give expert-level, register-specific
> answers, no beginner framing. Sibling projects on their machine (ReactOS `sh4pe-toolchain`,
> `DreamShell`, `img4dc`) are SEPARATE — not part of this repo.

## Current state (resume here)
- ✅ **Self-contained CMake build.** The SH-4 compiler (`vendor/sh-toolchain`) and the whole
  CE 2.12 SDK (`vendor/wcesdk`: headers, libs, image tools, OS modules, patched kernels +
  `.map`/`.pdb`) are vendored. `CMakeLists.txt` (`project(NONE)`) drives the vendored
  `cl.exe`/`shasm.exe`/`link.exe` to build our modules, then `makeimg` → `wrap-image.ps1` →
  `make-gdi.ps1` for the bootable disc. Builds `retail` (silent) or `debug` (SCIF-console)
  images. See `toolchain/README.md`.
- ✅ **Networking — full TCP/IP over the STOCK CE stack.** The SDK's `mppp.dll` (dial-up PPP)
  is replaced by a universal link shim (`net/netif/`) so stock `microstk.exe` + `winsock.dll`
  run over Ethernet — no lwIP. **BBA path verified END-TO-END: DHCP → DNS → TCP → HTTP**, and a
  retail game (4x4 Evolution) dials + reaches its master server on real hardware. DNS chains
  DHCP option-6 → DC system-flash ISP config (`flashrom.c`) → public resolver. **Modem dial-up
  works** too: with no ethernet the shim delegates to the vendored original PPP driver
  (`mpppdial.dll`) and `dcwnet` RasDials to bring it up. **Revival mode** redirects dead game
  master servers to revival hosts — a revival DNS as PRIMARY (default `dns.flyca.st` = DCNet's
  `178.156.255.64`) + hardcoded-IP DNAT for the no-DNS games (Alien Front Online / Internet Game
  Pack); config in `HKLM\Comm\Netif`. Link-ABI + MTU/byte-order/DNS-registry gotchas are in the
  `net/netif/` sources.
- ✅ **DCWin desktop shell + apps.** `shell/` is a windowed desktop + PVR2/Direct3D compositor
  (move/resize/min/max, clipping, analog-stick pointer; the compositor quad buffers grow/shrink/
  free dynamically). Client apps run in their own processes: `dcwcalc`/`dcwclock`/`dcwexp`/
  `dcwtask`/`dcwmem`, `dcwnet` (Network Diagnostics: dial + DNS/TCP/HTTP test + download-to-RAM
  benchmark with progress/speed), `dcwlog` (in-app System Log viewer), `dcwplay` (music player).
  `dcshell` is the default autorun; `dcwboot.exe` (a D3D boot logo + live checklist that then
  launches `dcshell`) is an optional alternate. SDK-correct DirectInput (DC controller by Maple
  HID usage).
- ✅ **Music player** (`shell/dcwplay.c`) streams MP3 (minimp3) / WAV from disc into a 1-second
  AICA DirectSound ring. DC gotchas baked in: 32-byte buffer + Lock alignment (else
  `DSERR_NOT32BYTEALIGNED`), decoded-PCM carry (no clicks), and always-44100 output via a linear
  resampler (the AICA is pitch-exact only at its native 44100).
- ✅ **SD card over SPI** (`drivers/sdcard/`): a WDM FAT FSD mounts a FAT16 card on the SCIF SPI
  bus as `\External Storage` (FatFs); Explorer browses + launches off it.
- 🔄 **Internet Explorer** (`shell/iexplore.cpp` + `iehost`/`ieproto`) hosts the stock Trident
  WebBrowser control, baked behind **`-DHTML=on`** (the IMGHTML CoreOS_HTML split). Renders via a
  dcgfx page-layer; fetches http over a winsock `IInternetProtocol` (CE WinInet is dead on this
  image). Boots + `CoCreate` OK on Flycast; UNTESTED end-to-end on hardware.
- 🔄 **W5500/MACRAW backend over SPI** (`drivers/dcspi/` SCI hardware-SPI + SCIF bit-bang;
  `net/netif/w5500.c`). Detected over SPI (VERSIONR) and SMALL TCP transfers work (DNS, an HTTP
  308, the 4x4 master), but a SUSTAINED receive (a ~1 MB download) STALLS: the inbound side
  freezes (the netif `rx=` counter stops) ~6–7 KB in. A torn-read guard (re-read the MACRAW
  length header before discarding the ring) + RX-backlog/desync `SysLog` are in to localize it
  on the bit-bang SCIF bus; user suspects the Flycast W5500 emulation for the recent stalls.

## Setup on a fresh PC
1. `git clone <this repo>` — **fully self-contained.** Both the SH-4 compiler
   (`vendor/sh-toolchain`) and the CE 2.12 SDK (`vendor/wcesdk`: headers, libs, image tools,
   OS modules, patched kernels + their `.map`/`.pdb`) are vendored. No external SDK.
2. Install **CMake ≥ 3.20 + Ninja** (the VS-bundled pair works — see `toolchain/README.md`).
   Nothing else to configure; the build derives all paths from the repo.

## Build / commands (CMake — see `toolchain/README.md`)
```sh
cmake -G Ninja -DCMAKE_MAKE_PROGRAM=<ninja> -S . -B build
cmake --build build                  # all SH-4 modules -> build/modules/ (dcspi/mppp/dcshell/dcw*)
cmake --build build --target image   # makeimg -> NK.bin -> wrap -> build/0winceos.bin
cmake --build build --target gdi      # sparse/truncated multi-track GDI (fast; emulator)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maximqaxd/wince-dc](https://github.com/maximqaxd/wince-dc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
