---
trigger: always_on
description: A Qt6 / C++ Debian GUI for the XGecu **T48 / T56 / TL866II+** universal device programmers, built on top of the open-source [minipro](https://gitlab.com/DavidGriffith/minipro) library.
---

# CLAUDE.md — xgecu-gui

A Qt6 / C++ Debian GUI for the XGecu **T48 / T56 / TL866II+** universal device programmers, built on top of the open-source [minipro](https://gitlab.com/DavidGriffith/minipro) library.

Released tag: **`v0.6.0`**. Verified end-to-end against a real T48 + Microchip-fab AT27C256 (UV-EPROM): read → save → load → verify → write → auto-verify round-trip works, and against an ATmega328P (fuse read + a live CKDIV8 write round-trip). Feature history: v0.2.0 fuse/config editor + copy-address; v0.3.0 Preferences (themes/fonts); v0.4.0 programmer-aware ZIF + write-dialog voltages; v0.5.0 HEX/S-record I/O + offset-merge + move + split/combine + serial numbers; v0.5.1 menu reorg; v0.6.0 mass-production mode + serial preview + chip-category filter + NAND/eMMC/VGA Windows-only.

License: **GPL-3.0-or-later** (forced by static linkage against minipro). Add `// SPDX-License-Identifier: GPL-3.0-or-later` to every new source file.

Window title: `XGecu T-48/T-56/TL866II+ by Xecaz`. About box credits Xecaz + Claude Code, 2026.

## Layout

- `src/core/` — non-Qt-Widgets logic: `Programmer` + `ProgrammerWorker` (on a dedicated `QThread`), `ChipDatabase`, `BufferModel` (with dirty tracking + `dirtyChanged` signal), `FileFormat` (HEX/S-record/binary decode-encode + split/combine transforms), `SerializationConfig` (serial-number production).
- `src/ui/` — Qt widgets: `MainWindow`, `ChipSelectDialog`, `ZifSocketView`, `HexView`, `FuseEditorWidget`, `PreferencesDialog`, `SerializationDialog`, plus `Theme`/`ThemeManager` (appearance).
- `third_party/minipro/` — **git submodule** of `https://gitlab.com/DavidGriffith/minipro.git`, pinned. Clone with `--recurse-submodules`.
- `scripts/merge_chip_lists.py` — combines `third_party/minipro/infoic.xml` + `../T48_List.txt` into `data/chips_merged.json`.
- `data/chips_merged.json` — generated chip catalog (~9 MB raw; gets Qt-resource-compressed in the binary). Committed for now so building doesn't require running the merge first.
- `tests/` — Qt Test unit tests (`test_chip_database`, `test_buffer_model`, `test_file_format` — HEX/S-record round-trips incl. >64K ELA, split/combine transforms, serial render/patch/persist).
- `tests/live/test_live_programmer.cpp` — live-hardware smoke tests, each method `QSKIP`s unless `XGECU_LIVE_TESTS=1`. The default `ctest` run stays green without hardware.
- `debian/` — packaging (`control`, `rules`, `changelog`, `copyright`, `source/format`, `postinst`, `postrm`). `dpkg-buildpackage -b -us -uc` builds the `.deb` into the parent dir. `debian/rules` uses `--buildsystem=cmake+ninja` (configure AND build via Ninja — don't reintroduce a bare `-GNinja` under the plain `cmake` buildsystem, which makes debhelper run `make` against a Ninja build dir and fail). Runtime deps are auto-filled by `dh_shlibdeps`; the dbgsym package is a normal side-product.
- `packaging/xgecu-gui.desktop` — application launcher.
- `packaging/xgecu-gui.svg` — stylised DIP-package icon.

## Build

Out-of-source build, Ninja generator:

```bash
git clone --recurse-submodules <repo> xgecu-gui && cd xgecu-gui
python3 -m venv .venv
.venv/bin/pip install -r scripts/requirements.txt
cmake -S . -B build -G Ninja
cmake --build build
./build/xgecu-gui
ctest --test-dir build --output-on-failure              # unit + skipped live
XGECU_LIVE_TESTS=1 ./build/tests/test_live_programmer   # active live run
```

The minipro static library is built via its own Makefile, invoked from CMake (`add_custom_command` → `make -C third_party/minipro library` via the resolved absolute GNU `make` path, not `${CMAKE_MAKE_PROGRAM}` since Ninja won't drive minipro's Makefile). Linker pulls in `libusb-1.0` and `zlib` via `pkg-config`.

`cmake --install build --prefix /usr` (or `DESTDIR=…` for staging) lays the binary down under `/usr/bin/`, the bundled `infoic.xml` / `logicic.xml` under `/usr/share/xgecu-gui/`, the `.desktop` + `.svg` icon in the standard XDG paths, and minipro's udev rules under `/usr/lib/udev/rules.d/`.

## Hardware

`lsusb` shows `a466:0a53 "TL866II Plus Device Programmer [MiniPRO]"` — that VID/PID is **shared between TL866II+, T48, and T56**. The actual model is determined post-handshake by minipro and lives in `minipro_handle_t::version` (`MP_T48 = 7`). The dev machine has a real **T48** plugged in (verified with `minipro -k`).

USB access on Debian 13: user must be in `plugdev` AND the minipro udev rules must be installed (from `third_party/minipro/udev/`; or installed system-wide by the `.deb`). The current dev machine has them in place — `/dev/bus/usb/.../<dev>` carries a `user:xecaz:rw-` ACL via `61-minipro-uaccess.rules`.

## Threading

**All `minipro_*` calls are blocking and must NEVER run on the GUI thread.** A dedicated `QThread` owns `ProgrammerWorker`, which holds the `minipro_handle_t*` for the lifetime of the connection. Communication is exclusively via queued signals/slots.

The worker API is **`MemArea`-parametric** (Code/Data):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xecaz/xgecu-t48.debian.gui](https://github.com/xecaz/xgecu-t48.debian.gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
