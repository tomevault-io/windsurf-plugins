---
trigger: always_on
description: - Goal: 1:1 feature parity with MediaInfo CLI
---

# AGENTS

Owner: soup

## Scope
- Goal: 1:1 feature parity with MediaInfo CLI
- Pure Go implementation, no CGO
- Cross-platform

## Workflow
- Don't ask for "continue": run parity + perf loops automatically, then commit/push once verified.
- Before committing: run `gofmt -w` on touched `.go` files.
- For `/mnt/storage/...` scans: be polite to disks (sample files; avoid full-tree scans; use `ionice -c3 nice -n 10` when doing bulk comparisons).

## Status (2026-02-16)
- CI: `gofmt` clean; `go test ./...` green.
- Parity target: official `/usr/bin/mediainfo --Output=JSON --Language=raw --ParseSpeed=0.5` (MediaInfoLib v23.04).
- TS parity controls: `Nickelodeon - Generic Halloween Promo.ts=0`, `Nickelodeon - Saturday Morning Promo.ts=0`, `Disney Channel - Evermoor Behind The Scenes.ts=0`.
- BDAV/M2TS parity spot checks now `0` on user discs:
  - `Static .../BDMV/STREAM/00004.m2ts`
  - `Intruders .../BDMV/STREAM/00061.m2ts`
  - `Sabrina .../BDMV/STREAM/00003.m2ts`
  - `Network UHD .../BDMV/STREAM/00005.m2ts` (HEVC derived `StreamSize`/`BitRate` + General remainder math)
  - `Network UHD .../BDMV/STREAM/00007.m2ts` (AC-3 dynrng_* suppression for mono)
  - `Excalibur UHD .../BDMV/STREAM/00004.m2ts` (bounded tail window selection + long-duration rounding)
- BDAV bounded ParseSpeed behavior aligned closer to MediaInfoLib:
  - PCR head shrink threshold: ~30s for BDAV (vs ~16.8s for TS captures); tail window uses head-sized end window when shrunk, else uses 16 MiB.
  - Long BDAV video Duration (`>= 3600s`): JSON uses millisecond truncation to match official 0.001s rounding on long clips.
  - AVC `Format_Settings_GOP`: emitted for BDAV only when duration window is stable (`>= 120s`), to avoid false-positive GOP on short clips.

## Status (2026-02-09)
- CI: green on `main` (recent runs: MP4/AVI/MPEG-PS fixes).
- Parity: verified 1:1 JSON raw vs `/usr/bin/mediainfo` (MediaInfoLib v23.04) for:
- Matroska samples (incl. 4K sample)
- MP4 sample
- VOB sample (`VTS_01_2.vob`) after `fix(mpeg-ps): improve VOB parity`

## Status (2026-02-11)
- CI: green locally (`go test ./...`, `gofmt -w` clean).
- PTP helper: `scripts/ptp_scan.go` supports `-min-seeders`; latest conservative scan (`4` requests, `35s` min interval) returned `13` candidates at `seeders>=10` to `.cache/ptp-scan/candidates.jsonl`.
- Focused parity samples (official: `mediainfo --Output=JSON --Language=raw --ParseSpeed=0.5`):
- TS `Nickelodeon - Generic Halloween Promo.ts`: diff `12` (down from `14` this pass; GOP JSON parity improved).
- TS `Disney Channel - Evermoor Behind The Scenes.ts`: diff `25` (remaining: AC-3 stats/count windows).
- M2TS `01007.m2ts`: diff `8` (remaining: AC-3 dynrng stats + 1-byte StreamSize split).
- M2TS `01099.m2ts`: diff `4` (remaining: 2-byte General/Video StreamSize split).
- Non-TS spot checks: `mp4/mp3/flac/avi` at `0`; one sampled `mkv` at `1`.
- Extended TS sweep (`/mnt/storage/torrents/keepalive/Halloween 2014 Promos and Bumpers Pack`, 12-file sample):
- Diff range `13..64`; recurring gaps are AC-3 `dynrng_*` count windows, MPEG-2 `Format_Settings_Matrix`/`intra_dc_precision`, caption service mapping (extra/misaligned text tracks), and missing General `Title`/`Movie` on some captures.
- GOP experiment note: forcing `Variable` on interlaced TS improves some Cartoon Network captures but regresses Nick captures; reverted for now.
- `ptp-archive` spot checks (ParseSpeed=0.5): `VTS_01_0.IFO=0`, `VTS_01_1.VOB=28`, `KAUTOKEINO 00011.m2ts: 62->27`, `KAUTOKEINO 00012.m2ts: 66->27` after BDAV MPEG-2 field parity expansion.
- Remaining highest-impact work:
- TS/BDAV AC-3 stats parity (`compr_*`/`dynrng_*` counts + averaging windows).
- BDAV/TS tiny General vs Video `StreamSize` byte-allocation parity (1-3 byte deltas).

## Status (2026-02-12)
- New samples checked:
- `.../David Bowie and the Story of Ziggy Stardust ... .ts` diff `107`.
- `.../Now You See Me ... /BDMV/STREAM/00000.m2ts` diff `400`.
- `.../Now You See Me ... /BDMV/PLAYLIST/00100.mpls` diff `27146`.
- `.../Infernal.../STREAM/00000.m2ts` diff `272`; `.../PLAYLIST/00999.mpls` diff `13468`.
- `.../Intruders.../STREAM/50201.m2ts` diff `507`; `.../PLAYLIST/00020.mpls` diff `6699`.
- `.../Network.../STREAM/00007.m2ts` diff `214`; `.../PLAYLIST/00002.mpls` diff `1390`.
- Additional availability sweep (non-cross-seed canonical paths):
- `A.Beautiful.Mind.../STREAM/00068.m2ts` diff `312`; `.../PLAYLIST/00020.mpls` diff `3091`.
- `Now You See Me.../STREAM/00000.m2ts` diff `406`; `.../PLAYLIST/00101.mpls` diff `27074`.
- `Through.the.Looking.Glass.../STREAM/00003.m2ts` diff `164`; `.../PLAYLIST/00001.mpls` diff `1159`.
- Continuous filename parity:
- `detectContinuousFileSet` now includes sparse numeric sets (not just strict contiguous sequence) and picks highest numbered matching file.
- `detectContinuousFileSet` sparse matching retained for explicit `--File_TestContinuousFileNames=1` flows; default remains off.
- TS caption parser parity win:
- Fixed CEA-708 DTVCC packet payload sizing in `internal/mediainfo/mpeg_ts_mpeg2.go` (`packet_size_code` handling): payload bytes are `127` when code `0`, else `(code*2)-1`.
- Validation (official `mediainfo --Output=JSON --Language=raw --ParseSpeed=0.5`):
- 71-file TS sweep (`keepalive/Halloween...`): `improved=6 same=65 worse=0`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autobrr/go-mediainfo](https://github.com/autobrr/go-mediainfo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
