---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## What this is

libmongocrypt is a C library that assists MongoDB Client Libraries in implementing **In-Use Encryption**. The Client Library behavior it helps support is defined by the Client-Side Encryption specification: <https://github.com/mongodb/specifications/blob/master/source/client-side-encryption/client-side-encryption.md>.

### Naming (from the spec)

The public feature is **In-Use Encryption**, consisting of **Client-Side Field Level Encryption (CSFLE)** and **Queryable Encryption (QE)**. Internally, In-Use Encryption is sometimes called **Field Level Encryption (FLE)**; CSFLE is sometimes called Client Side Encryption (as in the spec title); and CSFLE and QE are sometimes called **FLE1** and **FLE2** respectively. So the `mc-fle2-*` payload files under `src/` implement **Queryable Encryption**, not CSFLE.

Separately (and confusingly), **`csfle`** is sometimes used within libmongocrypt to mean the **crypt_shared** library — an older name for it, unrelated to the CSFLE feature above. For example, `crypt->csfle` is the crypt_shared vtable.

## Build System

This project uses CMake.

Use `cmake-build/` as the CMake binary directory unless otherwise specified by the user. When the user specifies a custom binary directory, always use that directory - do not fall back to `cmake-build/`.

> [!IMPORTANT]
> Despite `build/` being a common choice for a CMake binary directory name, that is not recommended in this repository because the `build/` directory is not ignored by Git.

The typical configure and build steps (for release and installation):

```bash
cmake -D CMAKE_BUILD_TYPE=RelWithDebInfo -B cmake-build
cmake --build cmake-build
```

The optional install step:

```bash
cmake --install cmake-build
```

Key CMake configuration options for this project (given `option=(default|alternatives...)`):

- `-D CMAKE_INSTALL_PREFIX:PATH=<install-prefix>`: install directory used by `install()`. Use `cmake-build/install/` when system modification is undesirable or disallowed by the user.
- `-D MONGOCRYPT_CRYPTO:STRING=(OpenSSL|CommonCrypto|CNG|none)`: crypto backend. Defaults to the platform native (OpenSSL on Linux, CommonCrypto on macOS, CNG on Windows).
- `-D DISABLE_NATIVE_CRYPTO:BOOL=(OFF|ON)`: shortcut for `MONGOCRYPT_CRYPTO=none` — build with no crypto backend; a driver then supplies crypto at runtime via `mongocrypt_setopt_crypto_hooks`.
- `-D OPENSSL_ROOT_DIR:PATH=<dir>`: OpenSSL location, if not on a default path.
- `-D USE_SHARED_LIBBSON:BOOL=(OFF|ON)`: link an installed libbson instead of the bundled static one.
- `-D ENABLE_STATIC:BOOL=(ON|OFF)`: build and install static libraries.
- `-D BUILD_TESTING:BOOL=(ON|OFF)`: required to enable test targets including `test-mongocrypt` (see [Running Tests](#running-tests)).
- `-D ENABLE_ONLINE_TESTS:BOOL=(ON|OFF)`: required to enable test targets requiring external servers and the `csfle` utility (requires libmongoc).

> [!NOTE]
> `.evergreen/build_all.sh` is the authoritative reference for CI configure-build-install routines. Consult for platform-specific options and flags.

> [!NOTE]
> For local development and testing, use the Debug configure in the [Running Tests](#running-tests) section instead of the release configure above.

## Dependencies

- **libbson** (BSON support): *fetched* at configure time by `cmake/FetchMongoC.cmake`, which downloads the mongo-c-driver source archive at a pinned tag and builds libbson, statically linked into libmongocrypt by default. See comments in `cmake/FetchMongoC.cmake` to bump the pinned tag. Set `-D USE_SHARED_LIBBSON=ON` to instead link an installed libbson found via `find_package(bson)` (set `CMAKE_PREFIX_PATH`). 
- **Intel DFP** (Decimal128 math; target `mongocrypt::intel_dfp`): built at configure time from a *vendored tarball* under `third-party/`. Enabled by `MONGOCRYPT_ENABLE_DECIMAL128` (default ON). Set `-D MONGOCRYPT_DFP_DIR=USE-SYSTEM` to instead use a system install.
- **kms-message** (KMS request/response): a separate subproject under `kms-message/`, statically compiled into libmongocrypt. The MongoDB C Driver also vendors kms-message.
- **Crypto backend**: OpenSSL / CommonCrypto / CNG, selected by `MONGOCRYPT_CRYPTO` (see above). On Linux the system OpenSSL is used by default; `DISABLE_NATIVE_CRYPTO=ON` removes the backend entirely.

## Running Tests

The typical configure and build steps (for testing and development):

```bash
cmake -D CMAKE_BUILD_TYPE=Debug -B cmake-build
cmake --build cmake-build
```

> [!TIP]
> For development, prefer the Ninja generator (`-G Ninja`) when available: it builds in parallel across all cores by default, for the fastest edit-build-test loop.

Test targets build by default (`BUILD_TESTING=ON`). The suite is a single executable, `test-mongocrypt`, at the top of the binary directory. Run it **from the source root** (it reads `test/data` and `test/example` via relative paths):

```bash
./cmake-build/test-mongocrypt                              # whole suite
./cmake-build/test-mongocrypt _test_setopt_kms_providers   # a single test, by function name
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb/libmongocrypt](https://github.com/mongodb/libmongocrypt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
