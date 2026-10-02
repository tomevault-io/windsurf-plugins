---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`rarftp` streams the contents of a RAR, ZIP, 7z or tar archive to an FTP server without extracting to disk (C++17, CMake). See `README.md` for options and behaviour. `rarftp-gui` is a desktop front-end for the same engine (Tauri 2 + vanilla HTML/CSS/JS in `gui/`) that talks to it through the `rarftpcore` shared library.

## Build and test

UnRAR sources are **not in the repo** (license) and must exist in `unrarsrc/` (gitignored) or CMake fails at configure time; the exact `curl` + `tar` commands are in `README.md`.

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release   # add -DRARFTP_BUNDLED_CURL=ON to build FTP-only libcurl from source (what CI does)
                                                           # add -DRARFTP_BUILD_LIBRARY=ON for the shared library the GUI links
cmake --build build                                        # produces build/rarftp (and build/librarftpcore.{dylib,so} / rarftpcore.dll)
ctest --test-dir build --output-on-failure                 # unit tests (doctest); with the library also the C API smoke test (tests/test_capi.cpp)
build/tests/rarftp_tests -tc="*pipe*"                      # single unit test case, doctest filter
python3 tests/integration/run.py --rarftp build/rarftp --rar /path/to/rar [--7z /path/to/7zz] [-k NAME] [--big] [--lib build/librarftpcore.dylib]   # e2e
cd gui && cargo tauri build                                # GUI bundle (macOS: gui/src-tauri/target/release/bundle/macos/rarftp-gui.app); `cargo tauri dev` runs it
cargo test --manifest-path gui/src-tauri/Cargo.toml        # Rust tests (memory file, FFI wrapper)
```

- Integration tests need Docker (vsftpd via `delfer/alpine-ftp-server`), RARLAB's `rar`, and Python 3.9+ stdlib only (ZIP fixtures come from `zipfile`; the Zstandard one needs 3.14+). 7-Zip (`7zz`, for 7z, AES/split/Deflate64 ZIP) and Info-ZIP `zip` are optional: tests whose archive could not be made are skipped (`env.require`). `-k` filters tests by name substring. Active-mode tests only work on Linux. `--lib` adds the `lib_*` tests (`-k lib_`), which drive the C API with `ctypes` and check the JSON contract at every poll; they are skipped without it.
- The GUI build needs Rust, `cargo install tauri-cli --version "^2" --locked` and the library already built: `gui/src-tauri/build.rs` looks for it in `build/` (or `RARFTP_LIB_DIR`) and fails with the CMake command otherwise. The bundle configs (`tauri.macos.conf.json`, `tauri.linux.conf.json`; Windows has none, it is built with `--no-bundle`) hard-code `../../build/...` (the macOS one is read even without bundling); with another `RARFTP_LIB_DIR` override them with `cargo tauri build --config '<json>'` (the CLI exports it to the build script as `TAURI_CONFIG`; setting `TAURI_CONFIG` by hand reaches only the build script, not the bundler).
- Formatting: `.clang-format` (Google style, 115 columns, left pointer alignment). Warnings are strict (`-Wall -Wextra -Wpedantic -Wshadow -Wconversion`; `/W4` on MSVC).
- Dependencies (fmt, CLI11, FTXUI, doctest, optionally curl) are fetched by `FetchContent` with pinned URL + SHA-256 in `cmake/Dependencies.cmake`; bump the hash together with the version. UnRAR is built from `unrarsrc/` by `cmake/UnRAR.cmake` as a static library using the DLL API (`RAR_TEST` mode). libarchive and its dependencies (zlib with prefixed symbols, bzip2 via our `cmake/bzip2/CMakeLists.txt`, xz, zstd, lz4, mbed TLS on Linux only) are `ExternalProject`s in `cmake/LibArchive.cmake` (libarchive's configure links test programs against them, so FetchContent cannot work), always Release, installed into `build/archive-deps`, toolchain settings forwarded through `build/archive-deps-cache.cmake`; the imported target is `rarftp::libarchive`, whose system libs (iconv on macOS, bcrypt on Windows) are listed by hand. Rust crates are pinned by `gui/src-tauri/Cargo.lock`; new ones must be added to `THIRD_PARTY_NOTICES.md`.

## Architecture

`rarftp_core` (static library) is the engine only: archive reading, plan, FTP, transfer pipeline, progress/log, JSON and text helpers. It has no CLI11/FTXUI. It is linked into the `rarftp` executable (`src/main.cpp`, `options.cpp`, `ui_plain.cpp`, `ui_tui.cpp`: the only code using CLI11 and FTXUI), into `tests/`, and into the `rarftpcore` shared library (only with `RARFTP_BUILD_LIBRARY`).

Pipeline (`src/transfer.cpp`), two threads joined by a bounded `Pipe` (`src/pipe.hpp`):

1. **Extractor thread**: an `Archive` (`src/archive.hpp`) decompresses and verifies each entry in memory and feeds `on_data`, which pushes `PipeMessage`s (`EnsureDir`, `FileBegin`, `Data`, `FileEnd`, `End`).
2. **Uploader thread**: pops messages and streams `Data` to a single libcurl FTP `STOR` on one control connection (`src/ftp_client.cpp`).

Key invariants:
- The `Pipe` capacity counts only `Data` bytes; that is what couples decompression speed to upload speed (`--buffer`, default 64 MiB). Oversized messages are accepted when the queue is empty.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuiNelson/rarftp](https://github.com/RuiNelson/rarftp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
