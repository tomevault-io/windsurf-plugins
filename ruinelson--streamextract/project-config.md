---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`StreamExtract` streams the contents of a RAR, ZIP, 7z or tar archive, or an exFAT volume image, to an FTP server without extracting to disk (C++17, CMake). See `README.md` for options and behaviour. `StreamExtract` is a desktop front-end for the same engine (Tauri 2 + vanilla HTML/CSS/JS in `gui/`) that talks to it through the `streamextractcore` shared library.

## Build and test

UnRAR sources are **not in the repo** (license) and must exist in `unrarsrc/` (gitignored) or CMake fails at configure time; the same for 7-Zip's LZMA SDK in `lzmasdk/` (a `.7z`, extracted with bsdtar or 7-Zip; the user wants it fetched by hand like UnRAR, not by CMake). FatFs R0.16 must likewise be downloaded manually from elm-chan.org into `fatfs/` (gitignored); CMake builds a read-only copy with the official patches in the build directory. The exact commands are in `README.md`.

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release   # add -DSTREAMEXTRACT_BUNDLED_CURL=ON to build FTP-only libcurl from source (what CI does)
                                                           # add -DSTREAMEXTRACT_BUILD_LIBRARY=ON for the shared library the GUI links
cmake --build build                                        # produces build/sext (and build/libstreamextractcore.{dylib,so} / streamextractcore.dll)
ctest --test-dir build --output-on-failure                 # unit tests (doctest); with the library also the C API smoke test (tests/test_capi.cpp)
build/tests/streamextract_tests -tc="*pipe*"                      # single unit test case, doctest filter
python3 tests/integration/run.py --sext build/sext --rar /path/to/rar [--7z /path/to/7zz] [-k NAME] [--big] [--lib build/libstreamextractcore.dylib]   # e2e
cd gui && cargo tauri build                                # GUI bundle (macOS: gui/src-tauri/target/release/bundle/macos/StreamExtract.app); `cargo tauri dev` runs it
cargo test --manifest-path gui/src-tauri/Cargo.toml        # Rust tests (memory file, FFI wrapper)
```

- Integration tests need Docker (vsftpd via `delfer/alpine-ftp-server`), RARLAB's `rar`, and Python 3.9+ stdlib only (ZIP fixtures come from `zipfile`; the Zstandard one needs 3.14+). 7-Zip (`7zz`, for 7z, AES/split/Deflate64 ZIP) and Info-ZIP `zip` are optional: tests whose archive could not be made are skipped (`env.require`). `-k` filters tests by name substring. Active-mode tests only work on Linux. `--lib` adds the `lib_*` tests (`-k lib_`), which drive the C API with `ctypes` and check the JSON contract at every poll; they are skipped without it.
- The GUI build needs Rust, `cargo install tauri-cli --version "^2" --locked` and the library already built: `gui/src-tauri/build.rs` looks for it in `build/` (or `STREAMEXTRACT_LIB_DIR`) and fails with the CMake command otherwise. The bundle configs (`tauri.macos.conf.json`, `tauri.linux.conf.json`; Windows has none, it is built with `--no-bundle`) hard-code `../../build/...` (the macOS one is read even without bundling); with another `STREAMEXTRACT_LIB_DIR` override them with `cargo tauri build --config '<json>'` (the CLI exports it to the build script as `TAURI_CONFIG`; setting `TAURI_CONFIG` by hand reaches only the build script, not the bundler).
- Formatting: `.clang-format` (Google style, 115 columns, left pointer alignment). Warnings are strict (`-Wall -Wextra -Wpedantic -Wshadow -Wconversion`; `/W4` on MSVC).
- Dependencies (fmt, CLI11, FTXUI, doctest, optionally curl) are fetched by `FetchContent` with pinned URL + SHA-256 in `cmake/Dependencies.cmake`; bump the hash together with the version. UnRAR is built from `unrarsrc/` by `cmake/UnRAR.cmake` as a static library using the DLL API (`RAR_TEST` mode). The LZMA SDK is built from `lzmasdk/` by `cmake/LzmaSdk.cmake` as the static `lzmasdk` (only the C++ 7z decoding files, `Z7_EXTRACT_ONLY` as a public define since the headers depend on it, with thread support so that 7zAES's global key cache is locked, no assembly); its codecs register from static constructors in files nothing refers to, so `src/sevenzip_codecs.cpp` (compiled into `lzmasdk`) `#include`s them, plus the interface GUIDs (`MyInitGuid.h`), and the 7z reader calls `sevenzip_codecs_linked()` to keep it in the link. libarchive and its dependencies (zlib with prefixed symbols, bzip2 via our `cmake/bzip2/CMakeLists.txt`, xz, zstd, lz4, mbed TLS on Linux only) are `ExternalProject`s in `cmake/LibArchive.cmake` (libarchive's configure links test programs against them, so FetchContent cannot work), always Release, installed into `build/archive-deps`, toolchain settings forwarded through `build/archive-deps-cache.cmake`; the imported target is `streamextract::libarchive`, whose system libs (iconv on macOS, bcrypt on Windows) are listed by hand. Rust crates are pinned by `gui/src-tauri/Cargo.lock`; new ones must be added to `THIRD_PARTY_NOTICES.md`.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuiNelson/StreamExtract](https://github.com/RuiNelson/StreamExtract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
