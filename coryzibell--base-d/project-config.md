---
trigger: always_on
description: **base-d** is a universal multi-dictionary encoding library and CLI tool for Rust. It encodes binary data using 35+ built-in dictionaries including RFC standards (base64, base32), ancient scripts (hieroglyphs, cuneiform), emoji, playing cards, and more.
---

# Copilot Instructions for base-d

## Project Overview

**base-d** is a universal multi-dictionary encoding library and CLI tool for Rust. It encodes binary data using 35+ built-in dictionaries including RFC standards (base64, base32), ancient scripts (hieroglyphs, cuneiform), emoji, playing cards, and more.

**Key Details:**
- Version: 0.1.18
- Language: Rust (edition 2021)
- License: MIT OR Apache-2.0
- Repository: https://github.com/coryzibell/base-d

## Core Architecture

### Three Encoding Modes

The project supports three distinct encoding strategies:

1. **BaseConversion** (Mathematical) - Treats binary data as a single large number and converts to target base. Works with any dictionary size. Variable output length.

2. **Chunked** (RFC 4648) - Processes data in fixed-size bit groups for RFC 4648 compatibility. Requires power-of-two dictionary sizes (2, 4, 8, 16, 32, 64, 128, 256). Supports padding character.

3. **ByteRange** - Direct 1:1 byte-to-character mapping using Unicode codepoint ranges. Always 256 characters, zero encoding overhead.

### Module Organization

```
src/
├── lib.rs              # Public API: encode(), decode(), hash()
├── main.rs             # CLI with clap parser
├── core/
│   ├── dictionary.rs   # Dictionary struct with fast ASCII lookup tables
│   └── config.rs       # TOML config, EncodingMode enum
├── encoders/
│   ├── encoding.rs     # Mathematical BigUint base conversion
│   ├── chunked.rs      # RFC 4648 bit-chunking (base64/32/16)
│   ├── byte_range.rs   # Direct byte-to-codepoint mapping
│   └── streaming.rs    # Memory-efficient large file processing
├── compression.rs      # 6 compression algorithms (gzip, zstd, brotli, lz4, snappy, lzma)
├── hashing.rs          # 24 hash algorithms (cryptographic, CRC, xxHash)
└── detection.rs        # Dictionary auto-detection from encoded input
```

### Key Data Types

- **Dictionary**: Main encoding dictionary containing character set, HashMap index, optional O(1) lookup table for ASCII, mode, and padding
- **DictionariesConfig**: Loads and manages dictionary definitions from TOML files
- **EncodingMode**: Enum determining which encoding algorithm to use
- **StreamingEncoder/Decoder**: Process large files with constant 4KB memory usage
- **CompressionAlgorithm**: Enum for 6 compression algorithms (pure Rust)
- **HashAlgorithm**: Enum for 24 hash algorithms (16 crypto + 4 CRC + 4 xxHash)

## Configuration System

Dictionaries are loaded with layered overrides:
1. Built-in from embedded `dictionaries.toml`
2. User-level: `~/.config/base-d/dictionaries.toml`
3. Project-level: `./dictionaries.toml`

Load via `DictionariesConfig::load_with_overrides()`.

## Performance Characteristics

Recent optimizations (see OPTIMIZATION_SUMMARY.md):
- Fast lookup tables: O(1) array-based ASCII lookup (5x improvement over HashMap)
- Pre-allocated buffers: Eliminate reallocation overhead during encoding
- Cache-friendly chunking: 64-byte chunks align with L1 cache lines
- Combined operations: Use `div_rem()` instead of separate `%` and `/`
- Benchmarks: Criterion.rs suite achieving ~370 MiB/s for base64 encoding

Performance targets:
- **Encoding**: Base64 ~370 MiB/s, base256_matrix 1:1 mapping (no overhead)
- **Compression**: Snappy ~600 MB/s, LZ4 ~500 MB/s, Gzip ~80 MB/s
- **Hashing**: xxHash3 ~30 GB/s, BLAKE3 ~1 GB/s, SHA-256 ~300 MB/s
- **Streaming**: Constant 4KB memory regardless of file size

## Recent Features (2024)

### Compression Support
- 6 algorithms: gzip, zstd, brotli, lz4, snappy, lzma
- Pure Rust implementations (no OpenSSL)
- CLI: `--compress <algo>`, `--decompress <algo>`, `--level <N>`
- Libraries: flate2, zstd, brotli, lz4, snap, xz2

### Hashing Support
- 24 algorithms total:
  - 16 cryptographic: MD5, SHA-2 family, SHA-3 family, Keccak, BLAKE2/3
  - 4 CRC checksums: CRC16, CRC32, CRC32C, CRC64
  - 4 xxHash: xxHash32, xxHash64, xxHash3-64, xxHash3-128
- Pure Rust implementations (RustCrypto, crc, twox-hash)
- CLI: `--hash <algo>` with optional encoding via `-e`
- Default hex output, or encode hash with any dictionary

### Dictionary Auto-Detection
- Automatically identify encoding dictionary from input
- CLI: `--detect` with optional `--show-candidates <N>`
- Supports base64, base32, hex, and pattern-based detection

### Known Limitations
- **Streaming mode**: Does NOT yet support compression or hashing
  - Line 150-152 in main.rs: Explicit error if compression + streaming
  - No hash integration with streaming mode
  - TODO: Add streaming compression and streaming hashing support

## Code Style & Conventions

### When Making Changes

- **Minimal modifications**: Only change what's necessary to achieve the goal
- **Preserve optimizations**: Fast lookup tables, pre-allocation, chunking patterns
- **Match existing patterns**: Follow established encoder structure in `encoders/` directory
- **No breaking changes**: Keep public API stable (encode, decode, hash, compress functions)
- **Test coverage**: Run `cargo test` (73 tests as of 2024-11) before committing

### Dictionary-Related Code

When working with dictionaries:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coryzibell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
