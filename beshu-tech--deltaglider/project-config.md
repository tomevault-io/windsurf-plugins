---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeltaGlider is a drop-in S3 replacement that achieves 99.9% compression for versioned artifacts through intelligent binary delta compression using xdelta3. It's designed to store 4TB of similar files in 5GB by storing only the differences between versions.

## Essential Commands

### Development Setup
```bash
# Install with development dependencies using uv (preferred)
uv pip install -e ".[dev]"

# Or using pip
pip install -e ".[dev]"
```

### Testing
```bash
# Run all tests
uv run pytest

# Run unit tests only
uv run pytest tests/unit

# Run integration tests only
uv run pytest tests/integration

# Run a specific test file
uv run pytest tests/integration/test_full_workflow.py

# Run a specific test
uv run pytest tests/integration/test_full_workflow.py::test_full_put_get_workflow

# Run with verbose output
uv run pytest -v

# Run with coverage
uv run pytest --cov=deltaglider
```

### Code Quality
```bash
# Run linter (ruff)
uv run ruff check src/

# Fix linting issues automatically
uv run ruff check --fix src/

# Format code
uv run ruff format src/

# Type checking with mypy
uv run mypy src/

# Run all checks (linting + type checking)
uv run ruff check src/ && uv run mypy src/
```

### Local Testing with MinIO
```bash
# Start MinIO for local S3 testing
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  minio/minio server /data --console-address ":9001"

# Test with local MinIO
export AWS_ENDPOINT_URL=http://localhost:9000
export AWS_ACCESS_KEY_ID=minioadmin
export AWS_SECRET_ACCESS_KEY=minioadmin

# Now you can use deltaglider commands
deltaglider cp test.zip s3://test-bucket/
deltaglider stats test-bucket                  # Get bucket statistics
```

### Available CLI Commands
```bash
cp               # Copy files to/from S3 (AWS S3 compatible)
ls               # List S3 buckets or objects (AWS S3 compatible)
rm               # Remove S3 objects (AWS S3 compatible)
sync             # Synchronize directories with S3 (AWS S3 compatible)
stats            # Get bucket statistics and compression metrics
verify           # Verify integrity of delta file
put-bucket-acl   # Set bucket ACL (s3api compatible passthrough)
get-bucket-acl   # Get bucket ACL (s3api compatible passthrough)
```

## Architecture

### Hexagonal Architecture Pattern

The codebase follows a clean hexagonal (ports and adapters) architecture:

```
src/deltaglider/
├── core/           # Domain logic (pure Python, no external dependencies)
│   ├── service.py  # Main DeltaService orchestration
│   ├── models.py   # Data models (DeltaSpace, ObjectKey, PutSummary, etc.)
│   └── errors.py   # Domain-specific exceptions
├── ports/          # Abstract interfaces (protocols)
│   ├── storage.py  # StoragePort protocol for S3-like operations
│   ├── diff.py     # DiffPort protocol for delta operations
│   ├── hash.py     # HashPort protocol for integrity checks
│   ├── cache.py    # CachePort protocol for local references
│   ├── clock.py    # ClockPort protocol for time operations
│   ├── logger.py   # LoggerPort protocol for logging
│   └── metrics.py  # MetricsPort protocol for observability
├── adapters/       # Concrete implementations
│   ├── storage_s3.py      # S3StorageAdapter using boto3
│   ├── diff_xdelta.py     # XdeltaAdapter using xdelta3 binary
│   ├── hash_sha256.py     # Sha256Adapter for checksums
│   ├── cache_cas.py       # ContentAddressedCache (SHA256-based storage)
│   ├── cache_encrypted.py # EncryptedCache (Fernet encryption wrapper)
│   ├── cache_memory.py    # MemoryCache (LRU in-memory cache)
│   ├── clock_utc.py       # UtcClockAdapter for UTC timestamps
│   ├── logger_std.py      # StdLoggerAdapter for console output
│   └── metrics_noop.py    # NoopMetricsAdapter (placeholder)
└── app/
    └── cli/        # Click-based CLI application
        ├── main.py          # Main CLI entry point with AWS S3 commands
        ├── aws_compat.py    # AWS S3 compatibility helpers
        └── sync.py          # Sync command implementation
```

### Core Concepts

1. **DeltaSpace**: A prefix in S3 where related files are stored for delta compression. Contains a `reference.bin` file that serves as the base for delta compression.

2. **Delta Compression Flow**:
   - First file uploaded to a DeltaSpace becomes the reference (stored as `reference.bin`)
   - Subsequent files are compared against the reference using xdelta3
   - Only the differences (delta) are stored with `.delta` suffix
   - Metadata in S3 tags preserves original file info and delta relationships

3. **File Type Intelligence**:
   - Archive files (`.zip`, `.tar`, `.gz`, `.jar`, etc.) use delta compression
   - Text files, small files, and already-compressed unique files bypass delta
   - Decision made by `should_use_delta()` in `core/service.py`

4. **AWS S3 CLI Compatibility**:
   - Commands (`cp`, `ls`, `rm`, `sync`) mirror AWS CLI syntax exactly
   - Located in `app/cli/main.py` with helpers in `aws_compat.py`

### Key Algorithms

1. **Delta Ratio Check** (`core/service.py`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beshu-tech/deltaglider](https://github.com/beshu-tech/deltaglider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
