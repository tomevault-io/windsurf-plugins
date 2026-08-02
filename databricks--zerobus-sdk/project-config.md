---
trigger: always_on
description: C++ wrapper around the Rust core via the C FFI library (`rust/ffi/zerobus.h`).
---

# C++ SDK

C++ wrapper around the Rust core via the C FFI library (`rust/ffi/zerobus.h`).
Like Go, it consumes the C FFI directly; unlike Go there is no GC, so resource
management is RAII and errors are reported by throwing `zerobus::ZerobusException`.

## Structure

```
cpp/
├── include/zerobus/      # Public headers (the API surface)
│   ├── zerobus.hpp       # Umbrella header — include this
│   ├── sdk.hpp           # Sdk + SdkBuilder + TableProperties
│   ├── stream.hpp        # Stream (proto/JSON ingestion)
│   ├── arrow_stream.hpp  # ArrowStream (Arrow Flight, Beta)
│   ├── proto_schema.hpp  # ProtoSchema (UC JSON → descriptor + encoder)
│   ├── headers_provider.hpp  # HeadersProvider interface
│   ├── ack_callback.hpp  # AckCallback interface + AckCallback::from adapter
│   ├── config.hpp        # StreamOptions / ArrowStreamOptions / CallbackWaitPolicy / enums
│   ├── record.hpp        # UnackedRecord
│   ├── error.hpp         # ZerobusException
│   └── version.hpp       # ZEROBUS_CPP_VERSION
├── src/                  # Implementation (the only place that includes zerobus.h)
│   ├── sdk.cpp / stream.cpp / arrow_stream.cpp / proto_schema.cpp
│   ├── ack_callback.cpp  # AckCallback::from lambda adapter
│   ├── headers_callback.cpp  # extern "C" trampoline for HeadersProvider
│   └── detail/           # Internal: ffi_util, config_convert, headers_callback, ack_callback
├── tests/                # Dependency-free unit tests (plain executables)
├── examples/             # Runnable usage examples (JSON, proto, Arrow)
├── cmake/
│   ├── BuildRustFfi.cmake       # Builds libzerobus_ffi from local Rust source
│   └── zerobus-config.cmake.in  # find_package(zerobus) package-config template
├── CMakeLists.txt
└── Makefile              # Convenience wrappers (build/test/lint/fmt)
```

The public headers never include `zerobus.h`; they hold opaque, forward-declared
FFI handles. Only `src/` includes the generated C header, keeping the C FFI an
implementation detail of the SDK.

## Build commands

Run from `cpp/`:

- `make build` — Configure (CMake) and build the SDK, tests, and the examples
  under `examples/` (gated on `ZEROBUS_BUILD_EXAMPLES`, on by default)
- `make build-ffi` — Build only the Rust C FFI static library
- `make test` — Build and run the test suite (`ctest`)
- `make lint` — Formatting check + compiler warnings (`-Wall -Wextra`)
- `make fmt` — Format all sources with clang-format (Google style)
- `make clean` — Remove the build directory
- `make test SANITIZE=address` — Build + run the suite under a sanitizer
  (`address`, `thread`, or `undefined`; CMake option `-DZEROBUS_SANITIZE=`). Off
  by default. Targets the memory/lifetime bugs this FFI wrapper is prone to
  (use-after-free, double-free). `address`/`thread` also instrument the Rust FFI
  (nightly `-Zsanitizer` + `-Zbuild-std`) — a sanitizer only observes code
  compiled with it, so leaving the core uninstrumented would miss the races/UAF
  that live inside the FFI. This needs a nightly toolchain with
  `rust-src` and build the C++ side with clang to match the LLVM sanitizer
  runtime (`CLANG_CC`/`CLANG_CXX`, default `clang`/`clang++`); GCC can't resolve
  the instrumented archive's runtime symbols. `undefined` uses the default
  compiler and a plain FFI build. Sanitizer builds use their own build dir
  (`build-<sanitizer>`), so they don't clash with a normal `build/` (CMake
  caches the compiler per build dir).

CMake builds the FFI library from local Rust source by default
(`cargo build --release` in `rust/ffi`). To link a prebuilt/vendored library
instead, configure with `-DZEROBUS_FFI_LIBRARY=<path-to-.a>` and
`-DZEROBUS_FFI_HEADER_DIR=<dir>`.

## FFI boundary: RAII + static linking

### Memory ownership

- **Opaque handles**: each wrapper class holds a raw pointer to a Rust-allocated
  handle (`CZerobusSdk`, `CZerobusStream`, `CArrowStream`, `CZerobusProtoSchema`).
  Rust owns the memory.
- **RAII cleanup**: destructors call the matching free/close function
  (`zerobus_sdk_free`, `zerobus_stream_close` + `zerobus_stream_free`,
  `zerobus_arrow_stream_close` + `zerobus_arrow_stream_free`, etc.). Wrapper
  objects are **move-only** (no copies) — moving nulls the source handle so it
  is freed exactly once.
- **Error strings**: FFI calls route their `CResult` through a helper in
  `detail/ffi_util.hpp`, which converts failure into a `ZerobusException` and
  **always** frees `CResult.error_message` via `zerobus_free_error_message`.
- **Returned arrays**: `Stream::get_unacked_records` /
  `ArrowStream::get_unacked_batches` copy the payloads out, then free the FFI
  array (`zerobus_free_record_array` / `zerobus_arrow_free_batch_array`).
  `ProtoSchema::encode_json` frees its buffer with `zerobus_free_proto_bytes`;
  `descriptor_bytes()` copies the borrowed bytes.

### Headers provider callback

When using a custom `HeadersProvider`:
- `detail::zerobus_cpp_headers_trampoline` is an `extern "C"` function passed to
  `zerobus_sdk_create_stream_with_headers_provider`. `user_data` is the raw
  `HeadersProvider*`.
- The `Stream` / `ArrowStream` keeps a `std::shared_ptr<HeadersProvider>` alive
  for its whole lifetime; the handle is freed in the destructor **before** the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databricks/zerobus-sdk](https://github.com/databricks/zerobus-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
