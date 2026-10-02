---
trigger: always_on
description: - `runtime/`: Mach-O loading, guest memory, native execution (Developer service arena, Local signing page container `SignedImage`, or External JIT arena `DebuggerArena`), carried libraries (`GuestLink`), runtime stubs (`GuestStubs`), interpreter diagnostics.
---

# Repository Guidelines

## Project Structure & Module Organization

- `runtime/`: Mach-O loading, guest memory, native execution (Developer service arena, Local signing page container `SignedImage`, or External JIT arena `DebuggerArena`), carried libraries (`GuestLink`), runtime stubs (`GuestStubs`), interpreter diagnostics.
- `translation/`: macOS API adapters (AppKit input/windowing, Metal shader handling, audio, Security).
- `authorization/`: Developer service: bundled pairing, transport, tunnel extension and memory-preparation protocol.
- `launcher/`: the iPad app. `profiles/`: data-only descriptions of tested applications.
- `tests/`: C, Objective-C, Swift and Python tests; `testguest/`: our synthetic guest fixture.
- `tools/`: builds, diagnostics and regression scripts. `docs/`: building guide and architecture.
- `build/` and `logs/` are ignored outputs. Third-party applications and assets are never committed.

## Build, Test, and Development Commands

Requires Xcode, XcodeGen and Python 3. Personal settings (team, bundle ID, device) live in ignored `local.env`; see `local.env.example` and `docs/BUILDING.md`.

```sh
tools/test_emulation.sh     # sanitizer regression suite on the Mac
tools/test_translation_sim.sh   # the UIKit-backed adapters' tests, run in the simulator
tools/run.sh sim            # loader diagnostics in the simulator (TolkaraDiagnostics scheme)
TOLKARA_MODE=local-signing tools/run.sh sim   # Local signing: first initializer of testguest (or GUEST_EXE) from an ad-hoc page container
tools/install.sh            # build, sign and install the Tolkara app on the configured iPad
tools/probe_guest.sh "/path/to/Executable"   # inspect an unchanged executable
python3 tools/probe_captured_shaders.py       # compile every shader library a game left on the iPad with the iPad's own Metal (no app code runs)
python3 tools/build_signed_container.py       # build and sign the Local signing page container for GUEST_EXE
python3 tools/make_icon.py                    # redraw the app icon at every size and appearance (needs Google Chrome)
NATIVE_GUEST_SHIMS=GENERIC tools/install.sh   # build for no particular application: one adapter per translation/<Framework>/
tools/package_ipa.sh                          # unsigned generic TolkaraDiagnostics .ipa for External JIT (sideloaded, e.g. SideStore)
```

Device installs replace the app under the same bundle ID and keep its data.

## Coding Style & Naming Conventions

Four-space indentation in source files, two spaces in YAML; match surrounding conventions without unrelated reformatting. Preserve module prefixes such as `gm_`, `AK` and `TK`. Tests are named `test_*.{c,m,swift,py}`. Compiler warnings are errors in the regression scripts.

## Testing Guidelines

Standalone native executables and Python `unittest`, with ASan/UBSan where configured. Add focused coverage for memory bounds, malformed modules, protocol failures and adapter behavior. Simulator success does not prove physical Metal or authorization behavior; record device results in `COMPATIBILITY.md`.

## Runtime & Security Constraints

For this branch's Cyberpunk experiment, the user explicitly authorized a rare
exception on 2026-09-26: runtime memory emulation may fetch the faulting
instruction and execute its memory access against software backing. The opt-in
is restricted to Cyberpunk. The general rule below stands for other apps, and
the bans on game-code reverse engineering, binary patches, debugger attachment
after entry, input automation and integrity workarounds still apply. See
`docs/SOFTWARE_MEMORY.md` for the tested scope and current limits.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tolkara/tolkara](https://github.com/tolkara/tolkara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
