---
trigger: always_on
description: Four firmware targets, two variants (`usb`, `ps2`) x two chips (`esp`, `rp`). Shared code lives in [firmware/com/](firmware/com/) and is `#include`d by all of them, so a change there rebuilds everything.
---

# okhi

Four firmware targets, two variants (`usb`, `ps2`) x two chips (`esp`, `rp`). Shared code lives in [firmware/com/](firmware/com/) and is `#include`d by all of them, so a change there rebuilds everything.

```
firmware/usb/esp   ESP32-C2, ESP-IDF        firmware/ps2/esp   ESP32-C2, ESP-IDF
firmware/usb/rp    RP2040,   Pico SDK       firmware/ps2/rp    RP2040,   Pico SDK
```

A fifth RP2040 project, [firmware/uart_bridge/](firmware/uart_bridge/), builds the USB-UART bridge both variants ship. It is not one of the four targets and shares only [firmware/com/com_rp_pins.h](firmware/com/com_rp_pins.h) with them, the board pin map: pure macros, no Pico SDK includes, so a project can pull it in without the rest of the okhi hardware layer. `com_rp.h` and `com_rp_hw.h` include it too, so a GPIO is written down once.

## Git

**Never run `git commit` or `git push`.** Only the user commits. Never rewrite the working tree either: no `git stash`, no `git reset --hard`, no `git checkout -- <file>` on files the user changed. To read an older revision use `git show HEAD:path` or `git diff`.

The user runs auto-commit tooling that creates commits titled `prev7` and periodically squashes with `git reset HEAD~N`. Expect the working tree to get committed out from under you mid-session; that is theirs, not a problem to fix.

## Toolchain locations on this machine

**ESP-IDF v6.0.2, installed via EIM (not the classic installer).** `export.ps1` does **not** work: it looks for the venv under `C:\Users\regue\.espressif\python_env\...`, which does not exist, and fails with `ESP-IDF Python virtual environment not found`. The activation script is:

```
C:\Espressif\tools\Microsoft.v6.0.2.PowerShell_profile.ps1
```

It sets `IDF_PATH=C:\Users\regue\esp\.espressif\v6.0.2\esp-idf`, `IDF_TOOLS_PATH=C:\Espressif\tools`, `IDF_PYTHON_ENV_PATH=C:\Espressif\tools\python\v6.0.2\venv`, and defines `idf.py` as an alias for the function `Invoke-idfpy`. The install is described in `C:\Espressif\tools\eim_idf.json`.

**Pico SDK 2.3.0**, under `C:\Users\regue\.pico-sdk\`:

| | |
|---|---|
| SDK | `C:\Users\regue\.pico-sdk\sdk\2.3.0` |
| toolchain | `C:\Users\regue\.pico-sdk\toolchain\15_2_Rel1` (arm-none-eabi gcc 15.2.1) |
| cmake | `C:\Users\regue\.pico-sdk\cmake\v4.3.4\bin\cmake.exe` |
| ninja | `C:\Users\regue\.pico-sdk\ninja\v1.13.2\ninja.exe` |
| picotool | `C:\Users\regue\.pico-sdk\picotool\2.3.0` |

None of these are on `PATH`; call them by absolute path. The `CMakeLists.txt` pulls `${USERPROFILE}/.pico-sdk/cmake/pico-vscode.cmake`, which resolves SDK, toolchain and picotool by itself, so **no `-DPICO_SDK_PATH` is needed**.

## How the user builds

The user does **not** use the command line. They build with the **official ESP-IDF VS Code extension** and the **official Raspberry Pi Pico VS Code extension**. The CLI commands below exist so this agent can verify builds; they must stay compatible with the extensions, never replace them.

They line up today: the Pico extension's `.vscode/settings.json` pins cmake `v4.3.4`, ninja `v1.13.2`, SDK `2.3.0` and toolchain `15_2_Rel1`, which is exactly what the CLI commands here use, so both produce the same `build/`.

Two things to respect when cleaning:

- **`build/.cmake/api/v1/query/client-vscode/query.json`** is written by the Pico extension to ask cmake for the file API (IntelliSense, target list). A CLI `cmake` configure does not create it, so wiping `build/` loses it. Recreate it and reconfigure so the `reply/` is regenerated:
  ```powershell
  $q = '{"requests":[{"kind":"cache","version":2},{"kind":"codemodel","version":2},{"kind":"toolchains","version":1},{"kind":"cmakeFiles","version":1}]}'
  $d = "<project>\build\.cmake\api\v1\query\client-vscode"
  New-Item -ItemType Directory -Force -Path $d | Out-Null
  [System.IO.File]::WriteAllText("$d\query.json", $q)
  ```
- **Never touch `.vscode/`.** That is the extensions' configuration, not build output. Nothing here needs to modify it.

**Known problem, not yet fixed:** the `.vscode/settings.json` of *both* ESP projects points at an ESP-IDF install that no longer exists. `idf.espIdfPathWin` is `C:\Users\regue\esp\v5.5\esp-idf` (the real one is v6.0.2 under `C:\Users\regue\esp\.espressif\v6.0.2\esp-idf`), `idf.toolsPathWin` and `idf.pythonInstallPath` point under `C:\Users\regue\.espressif`, which does not exist at all, and `clangd.arguments` has `--query-driver` aimed at an **xtensa** gcc even though the ESP32-C2 is riscv32, plus `--compile-commands-dir=c:\Users\regue\Desktop\okhi\build`, a path that does not exist either. Ask the user before changing any of it.

## Building

### ESP (both variants)

In PowerShell, dot-source the profile first. Shell state does not persist between tool calls, so activation and build must go in the **same** command:

```powershell
. "C:\Espressif\tools\Microsoft.v6.0.2.PowerShell_profile.ps1" | Out-Null
Invoke-idfpy -C "c:\Users\regue\Desktop\okhi\firmware\usb\esp" build
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [therealdreg/okhi](https://github.com/therealdreg/okhi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
