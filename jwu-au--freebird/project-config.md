---
trigger: always_on
description: Repo-level instructions for AI coding agents. Read this **before** touching any code.
---

# AGENTS.md — FreeBird

Repo-level instructions for AI coding agents. Read this **before** touching any code.

---

## 1. What FreeBird is

A CLI that decrypts NetEase Cloud Music files into playable MP3 / FLAC / M4A, then produces proper filenames + audio tags and verifies audio integrity. Two input families are supported:
- **`.uc` / `.uc!` cache files** (streamed): single-byte XOR; metadata fetched from the NetEase API by musicId parsed from the filename.
- **`.ncm` files** (downloaded): AES/RC4 container with metadata and cover art embedded inside; decoded fully offline (no API call).

Two commands: `fb scan` (one-shot) and `fb watch` (continuous). Plus a Windows-only helper `fb install-flac`.

User-facing docs live in `README.md`. Release history lives in `CHANGELOG.md`. **Do not duplicate either of those into this file.**

---

## 2. Project at a glance

| Item | Value |
|---|---|
| Language / runtime | C# / .NET 10 |
| Solution file | `FreeBird.sln` |
| Projects | `FreeBird.Core`, `FreeBird.Cli`, `FreeBird.Core.Tests`, `FreeBird.Cli.Tests` (all under `src/`) |
| CI | GitHub Actions: `ci.yml` (PR + push, 3 OS) + `release.yml` (tag-triggered) |
| Supported OS | macOS (arm64), Linux (x64), Windows (x64) |
| Latest release | `git tag --sort=-creatordate | head -1` |

---

## 3. Architecture — settled decisions

These were debated and chosen for specific reasons. If you think one is wrong, raise it explicitly before changing.

- **Decryption (`.uc`)**: XOR every byte with `0xA3` (NetEase's actual obfuscation; verified against real cache files). Streamed, not buffered. Lives in `XorDecoder` + `FileProcessor`.
- **Decryption (`.ncm`)**: full NetEase download container (magic `CTENFDAM`) — AES-ECB-decrypt the RC4 key (core key `hzHRAmso5kInbaxW`) to build a 256-byte key box, RC4-XOR the audio body; metadata (AES-ECB + Base64, modify key `#14ljk_!]&0U<'(`) and cover art are embedded. Verified byte-exact against a real `.ncm` (`flac -t` clean). Lives in `NcmDecoder` (transient, BCL-only) + `NcmFileProcessor`. **The `.uc` `FileProcessor` is deliberately untouched** — extension routing is done by `IFileProcessorRouter` (picks `NcmFileProcessor` for `*.ncm`, else `FileProcessor`), injected into both orchestrators. `*.ncm` is added to the scan/watch globs and stripped in `StemBasedFileNamer.GetStem`.
- **Format sniffing**: First 12 bytes, magic-byte matching (MP3 ID3 / MP3 sync / FLAC `fLaC` / M4A `ftyp`). Unknown → quarantine. The sniffer is authoritative for both paths; the `.ncm` embedded `format` field is only an advisory hint.
- **Integrity check**: 4 levels — `off`, `l1` (TagLibSharp structural), `l3` (FLAC `flac -t` PCM-MD5 subprocess), `auto` (probe `flac` at startup, use l3 if available else l1).
- **Atomic writes**: All output goes through `<output>/.freebird-staging/<guid>.<ext>` then `File.Move(..., overwrite: true)`. Never write the final filename directly.
- **Failures**: Quarantine to `<output>/.freebird-failed/<stem>.<ext>` plus a `.txt` sidecar (key=value format) with `timestamp`, `source`, `source_size`, `source_mtime`, `format`, `integrity`, `reason`, `error_class`.
  - **The sidecar contract is a stability promise** — the watch loop reads it to skip permanently-failed files. Don't rename or drop fields without a migration plan.
- **DI**: Autofac with `IDependency` marker-interface convention (auto-registration via reflection scan in `CoreModule.cs`). Stateful singletons (rate limiters, mutex pools, supervisors) need explicit `SingleInstance()` carve-outs.
- **Service contracts (v3.5, Cli-side)**: `IServiceController`, `IElevationChecker`, `IEventLogWriter`, `ILogPathResolver` are registered in `CliServiceModule` (src/FreeBird.Cli/DependencyInjection) with OS-appropriate impls selected via `OperatingSystem.IsWindows()` (Windows* vs NotSupported*/NonWindows*); all `InstancePerLifetimeScope` (NOT singletons — they hold no shared state). `IConfigLoader` (FreeBird.Core.Service) is the one service contract that does NOT extend `IDependency` and is constructed explicitly (`new JsonConfigLoader(logger)`), not auto-scanned.
- **Logging**: Serilog. Console sink always; rolling file sink for watch mode (daily rotation, 14-day retention).
- **Naming**: Files are named `<artist> - <title>.<ext>`. For `.uc` the metadata comes from the NetEase API and falls back to `<musicId>.<ext>` when the API fails; for `.ncm` it comes from the file's **embedded** metadata and falls back to the original `.ncm` basename. Multi-artist joined with ` & ` in filename.
- **Tag writing**: `metaflac` subprocess for FLAC (preserves PCM-MD5); TagLibSharp for MP3/M4A. On by default; opt-out via `--no-write-tags`. Preserves existing unrelated tags (GENRE, DATE, etc).
- **Cover art (`.ncm` only)**: the embedded album cover is written via a separate `ICoverWriter` (`metaflac --import-picture-from` for FLAC, TagLibSharp `Picture` for MP3/M4A) — kept off `ITagWriter` so the shared `.uc` tag path stays untouched. Suppressed by `--no-write-tags`. The `.uc` path writes no cover.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwu-au/FreeBird](https://github.com/jwu-au/FreeBird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
