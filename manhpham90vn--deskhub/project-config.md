---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Rules

### 1. Do not write comments — anywhere

Write no comments in any file — no explanatory comments, no section banners, no
docstrings, no `TODO`/`FIXME` notes. This covers C++, `make/*.mk`, `CMakeLists.txt`,
`CMakePresets.json`, `scripts/*`, `.github/workflows/*` and config files alike.

Make the content self-explanatory instead: descriptive names, small functions, early
returns, named constants instead of magic numbers. If a block seems to need a comment,
extract it into a well-named function. Knowledge that must not be lost goes into the
error message of the path that fails without it, or into `docs/ARCHITECTURE.md`
§"Decisions worth remembering" (mirrored in every translation) — never into a comment.

The single exception is the root `Makefile`: its header block is the one place where
every target is documented. Keep it in sync when you add or rename a target, and
mirror the change in `make/help.txt` (what a bare `make` prints).

Not comments, so they stay: shebang lines, preprocessor directives and include
guards, license headers, and the `# vX.Y.Z` tag after a pinned action SHA in the
workflows. Markdown documents are prose, not code.

Comments still present in older files are legacy: when you edit next to one, move
anything load-bearing into the documentation above and delete it.

### 2. Prefer `core/` and `platform/` — reuse before you add

The whole point of this layout is that logic is written once and shared by all five
clients. Before writing anything in `client/*`, check whether it belongs in a lower
layer.

```
core/       platform-agnostic logic, pure C++20, no OS headers, unit-tested
platform/   thin OS abstractions with one shared API (depends on core)
client/     per-OS apps: android, ios, linux, macos, windows (depend on platform + core)
client/apple/  Swift shared by the macOS and iOS apps — not an app of its own
```

Decision order when adding code:

1. **Does `core/` or `platform/` already do this?** Search first — `core/include/deskhub/`
   and `platform/include/deskhubp/` are the public surfaces. Reuse it.
2. **Is it platform-agnostic?** Protocol, packetization, FEC, session state, input
   mapping, bitrate control, diagnostics → `core/`. Add tests in `core/tests/`.
3. **Does it need the OS, but with the same API everywhere?** Sockets, clock, logging,
   randomness, source enumeration → `platform/`, behind one header in
   `platform/include/deskhubp/`, with per-OS `.cpp` files selected in
   `platform/CMakeLists.txt` (see `UdpSocketPosix.cpp` / `UdpSocketWin.cpp`).
4. **Only genuinely OS-specific?** Capture, encode, decode, render, windowing, UI →
   `client/<os>/`. These conform to the contracts in
   `core/include/deskhub/media/VideoContract.h`.

Never duplicate logic across `client/*`. If you find yourself writing the same thing for
a second platform, stop and lift it into `core/` or `platform/`.

Swift that both Apple apps need is the one exception to "lift it into a lower layer": it
goes in `client/apple/swift/`, which both `client/macos` and `client/ios` reference as a
folder in their Xcode projects. Check it before adding Swift to either app.

Hard constraints:

- `core/` must not include any OS or third-party header, and must not depend on
  `platform/`. It stays unit-testable offline with no network and no GPU.
- `platform/` may include OS headers, but its public headers must expose one identical
  API on every OS.
- Use the shared helpers rather than raw OS calls: `LOGI`/`LOGW`/`LOGE` from
  `deskhubp/diag/Log.h`, plus `deskhubp/system/Clock.h`, `deskhubp/system/Random.h`,
  `deskhubp/net/UdpSocket.h`, `deskhubp/client/SourceQuery.h`.
- Platform code is split by role: `deskhubp/auth` (the one handshake),
  `deskhubp/client` (`HostLink`, `ScreenViewer`, `TerminalViewer`,
  `FileTransferClient`), `deskhubp/host` (`HostEngine`, `SharingHost`,
  `TerminalHost`, `FileHost`). Core session machines mirror it:
  `deskhub/session/client` and `deskhub/session/host`, with shared types beside
  them in `deskhub/session`. Put new code on the right side, or beside them if
  both sides genuinely share it.

## Commands

```sh
make                 # print the target list — builds nothing
make bootstrap       # install toolchain + deps (run once)
make test            # build and run core_tests offline — the fast feedback loop
make test-all        # core + platform + integration suites
make test-ctest      # same tests through CTest, as CI runs them
make coverage        # core coverage report (clang + llvm-cov)
make debug           # configure + build the debug preset of the shared CMake tree
make format          # format C++ / Kotlin / Swift
make lint            # check formatting without writing, then lint-dead (what CI enforces)
make lint-dead       # dead code as an error: cppcheck + FFI/string-id/Kotlin checks + detekt
make lint-dead-swift # Periphery over both Apple apps (macOS + Xcode)
make lint-tidy       # clang-tidy over core/src + platform/src, the same gate CI runs
```

Per-platform: `make build-<os>`, `run-<os>`, `release-<os>` where `<os>` is one of
`linux`, `windows`, `macos`, `ios`, `android`. No platform is the default — a bare

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manhpham90vn/Deskhub](https://github.com/manhpham90vn/Deskhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
