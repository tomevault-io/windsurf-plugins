---
trigger: always_on
description: This file contains information to help Claude (and other AI assistants) understand and work effectively with this repository.
---

# CLAUDE.md - Repository Management Guide

This file contains information to help Claude (and other AI assistants) understand and work effectively with this repository.

## Project Overview

**Name**: tigerblock
**Module**: `github.com/firetiger-oss/tigerblock`
**Type**: Go library/toolkit
**Purpose**: Batteries-included toolkit for building applications on top of object storage in Go (storage abstraction, secrets, notifications, OCI/oras integration, CLI)
**License**: Apache 2.0

## Architecture

### Storage Package (`storage/`)

The `storage` package (`github.com/firetiger-oss/tigerblock/storage`) provides a unified interface for cloud object storage providers. The `Bucket` interface (`storage/storage.go`) defines 14 methods for storage operations:

**Metadata Operations:**
- `Location() string` - Returns the bucket URI
- `Access(ctx) error` - Verifies bucket accessibility
- `Create(ctx) error` - Creates a new bucket

**Object Operations:**
- `HeadObject(ctx, key) (ObjectInfo, error)` - Retrieves metadata without body
- `GetObject(ctx, key, options...) (io.ReadCloser, ObjectInfo, error)` - Retrieves object content
- `PutObject(ctx, key, value, options...) (ObjectInfo, error)` - Stores an object
- `DeleteObject(ctx, key) error` - Removes an object
- `DeleteObjects(ctx, objects) iter.Seq2[string, error]` - Batch delete with streaming
- `CopyObject(ctx, from, to, options...) error` - Copies within bucket
- `ListObjects(ctx, options...) iter.Seq2[Object, error]` - Lists objects
- `WatchObjects(ctx, options...) iter.Seq2[Object, error]` - Watches for changes

**Presigned URL Operations:**
- `PresignGetObject(ctx, key, expiration, options...) (string, error)`
- `PresignPutObject(ctx, key, expiration, options...) (string, error)`
- `PresignHeadObject(ctx, key) (string, error)`
- `PresignDeleteObject(ctx, key) (string, error)`

### Key Data Types

```go
// Object - minimal metadata for listings
type Object struct {
    Key          string
    Size         int64
    LastModified time.Time
}

// ObjectInfo - full metadata
type ObjectInfo struct {
    CacheControl    string
    ContentType     string
    ContentEncoding string
    ETag            string
    Size            int64
    LastModified    time.Time
    Metadata        map[string]string
}
```

### Storage Backends

| Backend | URI Format | Description |
|---------|------------|-------------|
| `storage/s3/` | `s3://bucket-name/path` | Amazon S3 |
| `storage/r2/` | `r2://bucket-name/path` | Cloudflare R2 |
| `storage/gs/` | `gs://bucket-name/path` | Google Cloud Storage |
| `storage/file/` | `file:///path` or `/path` | Local file system |
| `storage/memory/` | `:memory:` | In-memory (testing) |
| `storage/http/` | `http://host/path` | HTTP/HTTPS with S3-compatible server |

### Adapters

Adapters wrap buckets to add functionality (all under `storage/`):

| File | Purpose |
|------|---------|
| `storage/cache.go` | In-memory caching with LRU and TTL |
| `storage/prefix.go` | Mount bucket at a key prefix |
| `storage/readonly.go` | Make bucket read-only |
| `storage/instrument.go` | OpenTelemetry tracing |
| `storage/mount.go` | Mount different buckets at different prefixes |
| `storage/merge.go` | Combine multiple buckets into one |
| `storage/empty.go` | Read-only empty bucket |
| `storage/overlay.go` | Layered bucket overlay |

### Key Patterns

- **Registry Pattern**: Backends register with `storage.Register(scheme, registry)`
- **Adapter Pattern**: `storage.AdaptBucket(bucket, adapters...)` wraps buckets
- **Options Pattern**: `GetOptions`, `PutOptions`, `ListOptions` for configuration
- **Iterator Pattern**: Uses Go 1.23+ `iter.Seq2[T, error]` for streaming

### Error Codes

```go
var (
    ErrBucketExist         // Bucket already exists
    ErrBucketNotFound      // Bucket doesn't exist
    ErrBucketReadOnly      // Write on read-only bucket
    ErrObjectNotFound      // Object doesn't exist
    ErrObjectNotMatch      // ETag mismatch (conditional write)
    ErrInvalidObjectKey    // Invalid object key
    ErrInvalidObjectTag    // Invalid metadata tag
    ErrInvalidRange        // Invalid byte range
    ErrPresignNotSupported // Backend doesn't support presigning
    ErrPresignRedirect     // Presign requires redirect
    ErrTooManyRequests     // Rate limited
)
```

## Supporting Packages

### cache/
Caching implementations:
- `Cache[K,V]` - Generic cache with singleflight deduplication
- `SeqCache[K,V]` - Iterator-aware caching for `iter.Seq2`
- `LRU[K,V]` - LRU cache with promise-based async loading
- `TTL[K,V]` - LRU with time-to-live expiration

### backoff/
Retry logic:
- `Exponential()` - Exponential backoff strategy (100ms → 200ms → 400ms...)
- `FullJitter(strategy)` - Adds randomization to prevent thundering herd
- `Watch[T](ctx, fn)` - Polls and yields only when values change

### uri/
- `Split(uri) (scheme, location, path)` - Parses storage URIs
- `Join(scheme, location, path) string` - Constructs URIs
- `Clean(path)` - Normalizes paths

### internal/oteltrace/
OpenTelemetry integration:
- `Start(ctx, name, attrs...)` - Creates spans
- `RecordError(span, err)` - Records errors on spans
- `RecordSeq(seq)` - Wraps iterators with telemetry

### internal/sequtil/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firetiger-oss/tigerblock](https://github.com/firetiger-oss/tigerblock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
