---
trigger: always_on
description: A single, dependency-free Python tool (`machomorph.py`) that converts a Mach-O
---

# macos-to-ios

A single, dependency-free Python tool (`machomorph.py`) that converts a Mach-O
binary from one Apple platform to another (typically macOS -> iOS) so it can be
run on a jailbroken device.

It replaces this multi-tool dance:

```sh
lipo -thin arm64e /usr/sbin/ioreg -output /tmp/ioreg_thin
./cbv /tmp/ioreg_thin to ios 27.0
otool -L /tmp/ioreg_thin
install_name_tool -change .../Versions/A/CoreFoundation .../CoreFoundation /tmp/ioreg_thin
ldid -e /tmp/ioreg_thin > ents.xml
codesign --entitlements ents.xml -f -s - /tmp/ioreg_thin
```

with one invocation:

```sh
./machomorph.py /usr/sbin/ioreg -o ioreg_ios --platform ios --version 27.0
```

## Design decisions

* **Pure stdlib Python 3.** No `lief`, no `macholib`. The Mach-O edits needed here
  are small and well understood; a hand-rolled `struct`-based reader/writer keeps
  the tool a single file with zero install story. (`lief` was considered and
  rejected: heavy dependency, and it rewrites/normalises more of the binary than
  we want.)
* **Binary rewriting only.** Every library this project ships is Apple's own
  binary, lifted out of the dyld shared cache and rewritten. Compiling a
  library from source and hand-writing a stub were both tried, both worked, and
  both have been **abandoned and deleted** (`ios-libs/`, `libxcselect-stub/`).
  Lifting is strictly more general: it works for `libdtrace` and `libxcselect`,
  which have no open-source upstream and exist as a file nowhere, and it gives
  Apple's actual implementation rather than a look-alike. Do not add a source
  build or a stub back; lift instead. The measurements from both routes are
  kept below because they are the evidence for the diagnosis, not because they
  describe what is shipped.
* **Only `codesign` is shelled out to.** Re-implementing CMS/ad-hoc code signing
  is out of scope; the user explicitly allowed assuming macOS + Xcode tools.
  `ldid` is *not* required — entitlement extraction is done by parsing the
  embedded `CS_SuperBlob` ourselves.
* **Load commands are rebuilt, not patched in place.** This handles dylib paths
  that grow as well as shrink. We check available header padding before writing
  and refuse rather than clobber `__text`.
* **Nothing else in the file moves.** All file offsets stay identical, so no
  fixups, symbol tables or LINKEDIT data need rewriting.
* **One binary at a time is the PRIMARY use, and a conversion includes the
  libraries.** `machomorph.py <one binary> -o <path>` produces a binary that
  runs, which means it also produces every library the target does not have,
  lifted out of the shared cache and staged beside it. The batch is the same
  mechanism pointed at a whole system, not a different tool. See "The
  architecture: one tool, and what is left outside it".

## What we reimplemented, and how it was verified

| External tool | Replacement | Verification |
|---|---|---|
| `lipo -thin` | `FatSlice` extraction in `machomorph.py` | byte-identical to `lipo -thin arm64e` |
| `cbv` | `set_platform()` + cpusubtype fix | byte-diffed cbv output vs ours (see below) |
| `install_name_tool -change` | `rewrite_paths()` | `otool -L` comparison |
| `ldid -e` | `extract_entitlements()` (CS_SuperBlob parser) | compared to `ldid -e` output |
| `ldid -S` / `codesign` | shells out to `/usr/bin/codesign` | n/a (intentional) |

### cbv reverse-engineered behaviour (verified by byte diff, 2026-08-30)

Running `cbv ioreg_thin to ios 27.0` changes exactly three things in the Mach-O:

1. `LC_BUILD_VERSION.platform` -> target platform id.
2. `LC_BUILD_VERSION.minos` -> requested `maj.min.micro`.
3. `LC_BUILD_VERSION.sdk` -> `maj << 16` (major only, minor/micro zeroed).

Plus, **for macOS targets only** on arm64e, `mach_header.cpusubtype` is forced to
`0x81000002` (PTRAUTH ABI version 1) — with version 0 XNU on macOS kills the
process. For iOS targets cbv leaves the subtype alone; macOS system arm64e
binaries already carry `0x80000002`, which is what iOS wants. We additionally
clear any stale ptrauth-version bits when targeting non-macOS, which cbv does not.

Everything else that differs between `cbv` input and output (the `__LINKEDIT`
segment `filesize`, `LC_CODE_SIGNATURE.datasize`, and the signature blob itself)
is the work of the `codesign` calls cbv makes, not cbv itself.

## Task list

- [x] Reverse-engineer exactly what `cbv` mutates (byte diff against real `ioreg`)
- [x] Write CLAUDE.md
- [x] Fat/universal parsing + slice extraction (replaces `lipo -thin`)
- [x] Mach-O header + load command parser
- [x] `LC_BUILD_VERSION` / `LC_VERSION_MIN_*` platform+version rewriting (replaces `cbv`)
- [x] arm64e cpusubtype handling
- [x] Dylib/rpath path rewriting incl. automatic `Versions/A/` stripping (replaces `install_name_tool -change`)
- [x] Load-command area rebuild with header-padding check
- [x] Entitlement extraction from `CS_SuperBlob` (replaces `ldid -e`)
- [x] Entitlement injection (`research.com.apple.license-to-operate`)
- [x] Ad-hoc re-signing via `codesign`
- [x] `--info` mode (replaces `otool -hv`, `otool -L`, `ldid -e`)
- [x] End-to-end verification against the real toolchain
- [x] README
- [x] `--license-to-operate` made opt-out: added by default when the binary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mowisec/macos-to-ios](https://github.com/mowisec/macos-to-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
