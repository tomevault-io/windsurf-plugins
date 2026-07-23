---
trigger: always_on
description: A bug in this code can brick production devices. Extra caution required.
---

# Components/RealSenseID/Host/src/FwUpdate/ — bricking risk

A bug in this code can brick production devices. Extra caution required.

## File map

| File | Role |
|------|------|
| `FwUpdaterApi.cc` | Entry point; dispatches to F45x or F50x impl via `FwUpdater(DeviceType)` |
| `F45x/FwUpdaterF45x.cc` | `UpdateModules`, `ExtractFwInformation`, `IsSkuCompatible` for F450 |
| `F45x/FwUpdateEngineF45x.cc` | Partition burn logic: `BurnModules` → `BurnModule` (SCRAP/OPDW swap) |
| `F45x/FwUpdaterCommF45x.cc` | Raw serial I/O; `WriteCmd`, `WriteBinary`, `WaitForStr` |
| `F45x/Utilities.cc` | UFIF parsing: `ParseUfifToModules`, `ParseUfifToOtpEncryption` |
| `F45x/Cmds.cc` | Serial command strings: `dlinit`, `dl`, `dlact`, `dlinfo`, `dlspd`, etc. |
| `F50x/FwUpdaterF50x.cc` | Same API surface for F460/F500 |
| `F50x/FwUpdateEngineF50x.cc` | File-based burn logic; enforces BOOT module must be last |
| `F50x/FwUpdaterCommF50x.cc` | Raw serial I/O (identical structure to F45x comm) |
| `F50x/Utilities.cc` | UFIF parsing for F50x; stricter size bounds and entry count checks |
| `Common/Common.cc` | `CalculateCRC` (CRC-32 table), `LoadFileToBuffer` |

## F450 vs F460/F500 update paths

**F450 (`FwUpdateEngineF45x`)** — partition-based (SCRAP/OPDW swap):
1. `BurnModules`: sends `dlspd` to negotiate baud rate, calls `ModulesFromDevice` (`dlver`), `CleanObsoleteModules` (`dlsize sz=0`), `InitNewModules` (`dlnew`), then `BurnSelectModules`.
2. `BurnModule`: sends `dlinit` with session flag on first module; after each module (non-last) sends `dlact` and waits for `"validation ok"`; final module's `dlact session reboot` triggers reboot.
3. Partial-update logic: skips blocks whose CRC already matches device (`GetBlockUpdateList` reads `dlinfo` response); resumes partial updates only when module state is `active-updating`. Any other state forces all blocks.
4. Post-burn CRC re-check via a second `dlinfo`; throws `std::runtime_error("Update failed")` on any mismatch.
5. On error: dumps session to `fw-update.log`, resets `_comm`, rethrows — no automatic rollback.

**F460/F500 (`FwUpdateEngineF50x`)** — file-based:
1. `BurnModules`: sends `dlclean` before burn to reclaim space; no `CleanObsoleteModules`/`InitNewModules` step.
2. `BurnModule`: no session concept; each module uses `dlinit <filename> sz=<size>` (filename includes version, e.g. `SBC.7.1.2.0.SBIN`).
3. `dl` command includes the filename: `dl <filename> <blockNo>` (F45x sends only block number).
4. BOOT module must be last — `BurnModules` throws if it isn't. BOOT module contains `BOOT.INI` config, not firmware binary.
5. After all modules, sends `PacketManager::Commands::reset` to reboot.
6. Same post-burn CRC re-check; same `fw-update.log` dump on failure.

## UISP_PACK_VER / UFIF header checks

`ParseUfifToModules` (both F45x and F50x) throws `std::runtime_error` immediately in these cases:
- UFIF magic sig `!= 0x46484655` or major version byte mismatch
- F50x only: `entryN` outside `[1, 32]`; module size outside `[1024, 32MB]` (BOOT: `[8, 2048]`)
- Digest header major version mismatch (`hdr.ver >> 16 != DIGEST_HEADER_VERSION >> 16`, where `DIGEST_HEADER_VERSION = 0x00000004`)
- F45x: `hdr.id` not null-terminated within 7 bytes
- CRC32 of module data doesn't match `entry.crc32`
- Module name not in the `AllowedModules` set

None of these are soft failures — all propagate as `std::runtime_error` caught in `UpdateModules`, which returns `Status::Error`. There is no separate UISP_PACK_VER field checked at runtime on the host side; the version lives in `UfifFile.ver` and only its major byte is compared.

## SKU / locked-unit handling

`IsSkuCompatible` must be called by the caller before `UpdateModules` — `UpdateModules` does not call it internally.

- **F450**: reads `UfifFile.otpEncryptVersion` from the .bin file (via `ParseUfifToOtpEncryption`). Queries the device via `QueryOtpVersion`; falls back to serial-number regex if the device FW is too old to support that command. Mismatch returns `false` (no exception). If SN query also fails, returns `true` (assumes compatible).
- **F460/F500**: same field in the UFIF header. Device SKU determined by querying `bspver` and checking for the string `"(secure)"` between the two delimiter lines. Mismatch returns `false`.

The UFIF `otpEncryptVersion` field is the sole host-side indicator of locked vs unlocked. Dummy signatures in `icatch/tools/` are for unlocked units — these pass the same code path.

## Signature validation

There is no cryptographic signature verification in this host-side code. The `DigestHeader` struct (F45x `Utilities.cc`) contains ECDSA fields (`Qx`, `Qy`, `signature[64]`), but they are read only to extract `hdr.id` (module name) and `hdr.binVer` (version string). Actual signature verification happens on the device after each block is received — the device reports `dl ret=<N>` where `N != 0` signals rejection. `ParseDlBlockResult` treats non-zero return as failure and throws.

## Failure modes and edge cases

- **128 KB read buffer overflow**: `FwUpdaterCommF45x`/`FwUpdaterCommF50x` both use a fixed 128 KB ring buffer. If the device sends more than 128 KB during a session, `_read_index` wraps (with an `assert(false)`) — this will corrupt the session silently in release builds.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realsenseai/RealSenseID](https://github.com/realsenseai/RealSenseID) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
