---
trigger: always_on
description: Full rationale, gate layouts, and failed approaches live in
---

# Agent Workspace Guidelines — Chrome Manifest V2 Patcher

Full rationale, gate layouts, and failed approaches live in
[`mv2-reversing.md`](../mv2-reversing.md); the derivation workflow lives in
[`scripts/README.md`](../scripts/README.md). Read `mv2-reversing.md` before
changing any patching logic. This file is the short operating contract.

## Repository shape

Three self-contained patchers share one signature model; there is **no compiled
patcher or build step** (do not reintroduce the removed Go app / `build.bat`):

- `chrome-mv2.ps1` — Windows PE: `pe` (x64), `pe32` (x86), `pe-arm64` (Win-on-ARM).
- `chrome-mv2.sh` — Unix, dispatched by file magic: Linux `elf`/`elf-arm64` and the
  macOS `macho-arm64` framework slice (then ad-hoc re-signs the app). Must stay
  bash-3.2 clean; it fetches `signatures.json` from the URL and tokenizes it with
  `python3`, so `python3` is now required on the default path too. Intel macOS was
  dropped in v1.10.0.
- `chrome-mv2.py` — stdlib-only universal port (all containers).
- `scripts/*.py` — derivation/verification only; they never patch an install.
- `mv2-mem-patch-{win,mac}` — in-process launchers reusing the same tables.

The patch re-enables MV2 by flipping the `IsExtensionAffected` branch at each
inlined site, using per-milestone, container-tagged tables, applying only a
complete unambiguous match by default. Supported: Chrome **152 / 153 / 154 / 155**.

## Cardinal rule

Only flip the direction of an existing branch to its existing target:

- short `jg`: `7F`→`EB` (disp8 kept); a site may pin any Jcc via `stockOpcode` (the
  InstallVerifier gate flips a `75` `jne`). near `jg`: `0F 8F`→`90 E9` (disp32 kept).
- arm64 `bcond`: rewrite only the condition nibble GT(0xC)→AL(0xE); opcode `0x54`,
  bit4, and `imm19` preserved.
- arm64 `cbz`/`tbz`: a `cbz`/`cbnz` (imm19) or test-bit `tbz`/`tbnz` (`0x36`/`0x37`
  family, imm14) → unconditional `B` to the **same resolved target** (imm26 recomputed
  from the sign-extended imm19/imm14). `tbz` is the arm64 InstallVerifier flip; `cbz`
  is no longer used by any shipped milestone (Gate B removed).

Never delete/blank a `call`, edit the compared manifest-version value, or invent
control flow — structurally valid but semantically wrong edits have crashed Chrome
or hidden extensions. See `mv2-reversing.md` §6 before changing the byte strategy.

## Runtime ownership & safety contract

Keep platform behavior in its owning script (`ps1` = PE strip-Authenticode +
checksum; `sh` = ELF atomic-replace / Mach-O fat parse + inside-out ad-hoc
`codesign`). All engines implement the same contract:

- Validate signature data and image bounds; probe the recorded RVA, then relocate
  with a masked `.text` scan.
- Mask only volatile fields: the jump opcode + displacement; the `bcond`/`cbz`/`tbz`
  word's variable field (`bcond` condition + `imm19`; `cbz` `imm19` + Rt; `tbz`
  `imm14`, family/op/bit-position/Rt pinned); and, for `cbz`, any embedded `BL`/`B`
  (opcode class still checked — its `imm26` is build-specific).
- Require each site's exact `expectedMatches`; select the **best full match** (most
  sites) and **decline** equal-rank ties or partial/unknown layouts.
- Treat stock and already-patched opcodes as valid (idempotent reruns); preserve a
  validated, build-specific backup outside any signed bundle; report candidates
  without modifying an unknown layout.

Never weaken a decline, backup, identity, bounds, or post-write check to make a new
build pass — a declined unknown layout is the safe, expected result.

## Signature sources

`signatures.json` is canonical and the **only** copy. All three scripts fetch it from
the GitHub raw URL
(`https://github.com/Sketchystan1/chrome-mv2-patch/raw/master/signatures.json`) at
runtime; precedence is an explicit `--signatures`/`-Signatures` path →
`signatures.json` beside the script → the URL. There are **no embedded tables** (the
`$EmbeddedSignatures`/`EMBEDDED_SIGNATURES` blobs and `scripts/sync_embedded.py` were
removed), so editing `signatures.json` is the whole job — but **push it** so the raw
URL serves the change. Consequences: the default path now needs network, and
`chrome-mv2.sh` now needs `python3` on the default path (to tokenize the fetched
JSON). Keep older milestones so the scripts keep covering supported versions.

## Deriving / porting a milestone

Use the scripts, not hand analysis (`scripts/README.md` has the full workflow):
`fetch_chrome_binary.py` → `fetch_symbols.py` → **`port_milestone.py`** (the driver:
learns field offsets, folds shared bodies, picks the shortest signature free of
build-specific PC-relative immediates, carries site names from `--prev`) →
`audit_signatures.py --binary` → `run_tests.py` → patch a **scratch copy** and
GUI/runtime-test (with a real MV2 extension installed off-store via the non-policy
external-extensions provider — the reason-string trap in §5/§6 is invisible on a
fresh profile).

Two table-level failures are invisible from a single binary — never skip the audit:

- **Equal-rank tie.** Two milestones in one container with identical signature sets
  rank equally and the runtime declines. When a new version's gates are unchanged,
  reuse/extend the existing table instead of adding a duplicate (e.g. Chrome 154 mac

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sketchystan1/chrome-mv2-patch](https://github.com/Sketchystan1/chrome-mv2-patch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
