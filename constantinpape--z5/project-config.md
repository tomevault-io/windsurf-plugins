---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

z5 is a library for reading and writing zarr and n5 files. It is written in C++ (header-only, requires **C++20**) and represents multi-dimensional arrays with a lightweight, non-owning strided view (`ArrayView`/`ConstArrayView`, numpy-compatible). It provides Python bindings via nanobind that pass numpy arrays directly. The Python library is called z5py. The build system is CMake.

## Common Commands

### Build (Python bindings — the usual case)
Dependencies come from a conda env (`.github/environment.yaml`, or `environments/unix/z5-dev.yaml`). CMake auto-detects the active conda env as `CMAKE_PREFIX_PATH`; if that fails, pass `-DCMAKE_PREFIX_PATH=/path/to/conda-env`.

```bash
rm -rf bld && mkdir bld && cd bld
cmake .. -DWITH_BLOSC=ON -DWITH_ZLIB=ON -DWITH_BZIP2=ON -DWITH_XZ=ON -DWITH_LZ4=ON -DWITH_ZSTD=ON
make -j 4
```

`make` compiles the `_z5py` nanobind extension and copies the pure-Python `z5py` package + tests into `bld/python/`. `make install` additionally installs headers and the Python package into the conda env. nanobind is located via `python -m nanobind --cmake_dir` (or `CMAKE_PREFIX_PATH`); the active conda env must provide `nanobind` and `numpy`.

Key CMake options (see `CMakeLists.txt`): compression codecs `WITH_BLOSC` / `WITH_BZIP2` / `WITH_XZ` / `WITH_LZ4` / `WITH_ZSTD` are **OFF by default** (`WITH_ZLIB` is ON); a codec must be compiled in to read/write chunks using it. Backends: `WITH_S3`, `WITH_GCS` (both incomplete). Also `BUILD_TESTS` (C++ tests, OFF), `BUILD_Z5PY` (ON).

### Python tests
```bash
cd bld/python
python -m unittest discover -s test -v          # all tests
python -m unittest test.test_dataset -v          # one module
python -m unittest test.test_dataset.TestDataset.test_ds_open -v   # one test
```
CI instead runs `make install` then `python -m unittest discover -s src/python/test -v` against the installed package.

### Python coding standards

Python code should be written according to PEP8. 

The following linter commands should pass for each committed python file:
```bashbash
pyflakes <path/to/file.py>
flake8 <path/to/file.py> --max-line-length=120
```
Doc-strings should be written following the google convention.


### C++ tests
Configure with `-DBUILD_TESTS=ON` (uses the bundled googletest submodule — clone with `--recursive` or run `git submodule update --init`), build, then run the gtest binaries under `bld/src/test/`.

There is no configured linter/formatter in this repo.

## Code Architecture

### Layers

- **Abstract base classes** in `include/z5/`: `Dataset` (`dataset.hxx`), and the handle/metadata/attribute interfaces (`handle.hxx`, `metadata.hxx`, `attributes.hxx`). `Dataset` declares pure-virtual chunk-level IO (`writeChunk`/`readChunk`/`readRawChunk`), compression, and path operations.
- **Backend-generic implementations** in `include/z5/generic/`: the `Dataset`/`ShardedDataset` class templates parameterized on a per-backend `ChunkStore` policy (byte-level read/write/erase of one chunk or shard object; interface checked by the `ChunkStorePolicy` concept in `generic/store.hxx`), the attribute and metadata-write logic parameterized on a per-operation `JsonIO` object, and the factory dtype dispatch. All format logic (codecs, shard read-modify-write, zarr v2/v3 dispatch) lives here — backends supply only byte IO.
- **Backend implementations** of those bases live in `include/z5/filesystem/` and `include/z5/s3/` (both complete; s3 clients are cached process-wide per configuration — `makeClient()` returns a `shared_ptr` to the shared client) and `include/z5/gcs/` (an unimplemented scaffold whose operations all throw; a future implementation needs only a `ChunkStore` + `JsonIO` over GCS byte IO plus the handles). Each provides its own `handle`, `store` (the `ChunkStore` policy), `dataset`/`sharded_dataset` (alias templates onto `z5::generic`), `attributes` (`JsonIO` + thin wrappers), `metadata` (write side generic, read side per-backend), and `factory`.
- **`factory.hxx`** is the entry point (`createFile`, `createDataset`, `openDataset`, etc.). It dispatches to a backend **at runtime** by inspecting the passed handle (`root.isS3()` / `root.isGcs()` / `root.isZarr()`), guarded by `WITH_S3` / `WITH_GCS` compile flags. To use a backend, call the factory with that backend's handle type (e.g. `z5::filesystem::handle::File`).
- **`include/z5/compression/`**: one compressor per codec, all deriving from `CompressorBase` (`compress`/`decompress`/`type`). Each is gated behind its `WITH_*` define.
- **`include/z5/multiarray/`**: in-memory IO. `array_view.hxx` defines the non-owning strided `ArrayView`/`ConstArrayView` (element strides) plus `cOrderStrides`/`subview`/`makeView`; `array_util.hxx` provides the generic strided `copyView`/`fillView`; `array_access.hxx` implements `readSubarray`/`writeSubarray` (the main user-facing IO functions) on those views; `broadcast.hxx` implements `writeScalar`. To support another multiarray type, wrap its buffer in an `ArrayView` (data pointer + shape + element strides).
- **`include/z5/util/`**: threadpool, chunk blocking/iteration, file modes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [constantinpape/z5](https://github.com/constantinpape/z5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
