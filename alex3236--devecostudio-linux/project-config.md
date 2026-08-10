---
trigger: always_on
description: This file documents **why** the PKGBUILD does what it does — every piece of
---

# DevEco Studio Linux Package — Implementation Details

This file documents **why** the PKGBUILD does what it does — every piece of
"magic", the upstream quirk it works around, and the details that matter
when maintaining or forking this package. The README is the user-facing
view; this is the developer-facing view.

## Architecture: three sources, one package

| Source | What it provides | Why |
|---|---|---|
| **Mac DMG** (`devecostudio-mac.zip`) | `lib/*.jar`, `plugins/`, `modules/`, `license/`, `build.txt`, `bin/devecostudio.svg`, `bin/idea.properties`, `bin/devecostudio.vmoptions`, `Resources/product-info.json`, `tools/UxTestService` | Huawei ships DevEco Studio for Windows, macOS, and Linux. The Windows installer is an `.exe` that is painful to extract and lags in version; the Mac DMG extracts trivially with `7z x` and all of these files are platform-independent (Java bytecode, resources, templates). |
| **JetBrains IDEA tarball** (`idea-${_ideaver}.tar.gz`) | `jbr/`, `bin/idea` launcher, `bin/fsnotifier`, `lib/native/linux-x86_64/`, `lib/pty4j/linux/`, `lib/jna/amd64/`, `lib/skiko-awt-runtime-all/` | The macOS-specific bits (JBR, launcher, native `.so`s) are replaced with Linux ones. DevEco's build number is pinned to a specific IDEA baseline — see "IDEA version matching" below. |
| **Command Line Tools for Linux** (`commandline-tools-linux-x64.zip`) | `sdk/`, `tool/node/`, `hvigor/`, `ohpm/`, `hstack/`, `codelinter/`, `emulator/`, `bin/` wrappers | The CLI zip already contains Linux-native versions of every tool, and its SDK is the one the IDE needs. |

Everything from the Mac DMG that is not on the list above is either
platform-native (and unusable on Linux) or duplicated by the CLI: `jbr`,
`sdk/default`, `tools/emulator`, `tools/llvm`, `tools/profiler`,
`tools/node`, `tools/dumpParser` are all excluded during DMG extraction
(`prepare()`).

The two Huawei zips are **user-supplied** (Huawei's download links are
signed and expire), renamed to version-independent filenames
(`devecostudio-mac.zip`, `commandline-tools-linux-x64.zip`). Only the IDEA
tarball is auto-downloaded. Checksums live in `sha256sums`; users changing
versions update `pkgver` + the two checksums (or use `SKIP`).

## The magic, by area

### Emulator (`Emulator.exe` symlink)

Huawei's code (`LocalDeviceConnection.getEmulatorPathName`) only
distinguishes **Mac vs non-Mac**; the non-Mac branch hardcodes
`Emulator.exe`. On Linux the binary is named `Emulator`, so without a
symlink the Device Manager fails every operation ("get emulator status
failed") and debugging fails ("The emulator file ... is missing"). The
package ships `tools/emulator/Emulator.exe -> Emulator`.

Two consequences of the symlink:
- The `.exe` cleanup pass must keep it: `find "$_pkg" -name '*.exe' -not -name 'Emulator.exe' -delete`. (A past bug deleted it; it only existed locally because it had been created by hand.)
- The emulator binary itself comes from the **CLI** zip (Linux ELF), not the Mac DMG (Mach-O).

### Emulator system images

The IDE's "install emulator" wizard only appears when the emulator binary
is missing, and downloads binary **and** system image together. Because we
bundle the binary, the wizard never triggers — system images are the only
missing piece and must be fetched manually:
`Emulator -install -deviceType phone -osVersion "<version>"` (anonymous),
or copied from another platform's install into `~/.Huawei/Sdk/system-image/`.

### Emulator paths (`~/Library/Huawei/Sdk`)

The emulator binary hardcodes the macOS-style user path
`~/Library/Huawei/Sdk` for system images. The launcher wrapper bridges it:
`ln -sfn "$HOME/.Huawei/Sdk" "$HOME/Library/Huawei/Sdk"` on every start.
The same bridge is added to the `Emulator` CLI wrapper (in
`tools/bin/Emulator`) so CLI-only users who never launch the IDE still get
working image paths.
The emulator also needs `QT_QPA_PLATFORM=xcb` (it ships only the xcb Qt
platform plugin — no Wayland build), set by the wrapper.

### Emulator software agreements (auto-accept)

The IDE launches the emulator **binary directly**, bypassing the CLI
wrapper. If the HarmonyOS software agreements were never accepted, the
emulator silently waits for a `y` on stdin and the IDE appears hung — the
classic "starts only after you've run the CLI once" trap.

The agreement state lives in `~/Library/Caches/Huawei/Emulator26.0/.emu_config`
(written by `Emulator -license accept`). The `Emulator` wrapper therefore
does:

```bash
_emu_config="$HOME/Library/Caches/Huawei/Emulator26.0/.emu_config"
if [[ ! -f "$_emu_config" ]]; then
    "$all_tool_dir/emulator/Emulator" -license accept
    exit 0   # original command not forwarded; re-run it
fi
```

Only **existence** is checked, not content — so users can opt out of the
auto-accept by truncating the file (`> .emu_config`), which makes the
wrapper pass through and let the emulator handle agreements itself. Note
`-license` itself always prompts regardless of state (it is not a status
query), and `-license accept` prints every agreement before finishing, so
it takes a while.

### Previewer: unavailable on Linux (how we know)

The previewer (on-device preview) is the one major feature that cannot
work on Linux. The reasons were found by disassembling the CLI's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex3236/devecostudio-linux](https://github.com/alex3236/devecostudio-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
