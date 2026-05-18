---
trigger: always_on
description: A content-addressable registry for Typst templates with server-side rendering capabilities.
---

# Papermake - Typst Template Registry

A content-addressable registry for Typst templates with server-side rendering capabilities.

## Project Overview

Papermake consists of three main crates:
- **`papermake`**: Core Typst compilation engine with virtual filesystem
- **`papermake-registry`**: Content-addressable template storage and publishing
- **`papermake-server`**: HTTP API and web interface

## Architecture

```
User → Web UI → Server API → Registry Library → Typst Engine
                     ↓              ↓
                Cache Layer    Blob Storage (S3)
```

### Key Design Principles
- **Content-addressable storage** using SHA-256 hashes
- **Merkle tree approach** for efficient deduplication
- **Mutable tags** pointing to immutable content
- **Server-side rendering** with `template.render(data) → PDF`
- **Library-first architecture** with clean separation

## Current Implementation Status

### ✅ Phase 1: Core Typst Engine (`papermake` crate)
- [x] `TypstWorld` with virtual filesystem support
- [x] `TypstFileSystem` trait for async file resolution
- [x] Font caching via `CACHED_FONTS` static
- [x] Data injection through `sys.inputs.data`
- [x] Basic template rendering functionality

### ✅ Phase 2: Registry Foundation (`papermake-registry` crate)
- [x] `BlobStorage` trait with async operations
- [x] `S3Storage` implementation for AWS S3 compatibility
- [x] `ContentAddress` utilities for SHA-256 hashing and key generation
- [x] `TemplateBundle` and `TemplateMetadata` structs with validation
- [x] `Manifest` serialization/deserialization
- [x] Integration tests for storage layer

### 🚧 Phase 3: Registry File System Integration (In Progress)
- [x] `RegistryFileSystem<S: BlobStorage>` implementing `TypstFileSystem`
- [x] `TemplateReference` parsing (`namespace:tag@hash` format)
- [x] `Registry::publish()` method (store files → create manifest → update refs)
- [x] `Registry::resolve()` method (tag → manifest hash lookup)
- [ ] `Registry::render()` method using `RegistryFileSystem` ← **NEXT**

### 📋 Phase 4: Caching Layer (Planned)
- [ ] `Cache` struct with LRU for blobs, manifests, and refs
- [ ] Cache integration in `Registry` methods
- [ ] Cache invalidation API for webhooks
- [ ] Performance testing

### 📋 Phase 5: Server Layer (Planned)
- [ ] HTTP server with `/render/{reference}` endpoint
- [ ] `/publish` endpoint for template uploads
- [ ] Authentication and authorization
- [ ] Version tag immutability enforcement

## Code Structure

```
papermake/
├── papermake/                 # Core Typst engine
│   ├── src/
│   │   ├── lib.rs
│   │   ├── typst_world.rs     # TypstWorld implementation
│   │   └── filesystem.rs      # TypstFileSystem trait
│   └── Cargo.toml
├── papermake-registry/        # Registry core
│   ├── src/
│   │   ├── lib.rs
│   │   ├── storage/           # BlobStorage implementations
│   │   ├── bundle.rs          # TemplateBundle & TemplateMetadata
│   │   ├── manifest.rs        # Manifest format ← IMPLEMENT NEXT
│   │   ├── address.rs         # ContentAddress utilities
│   │   ├── registry.rs        # Registry core logic
│   │   └── reference.rs       # TemplateReference parsing
│   └── Cargo.toml
└── papermake-server/          # HTTP API (future)
    ├── src/lib.rs
    └── Cargo.toml
```

## Reference Format

Templates are referenced using: `[org/user]/name:tag@sha256:hash`

**Examples:**
- `invoice:latest` - Official template
- `john/invoice:latest` - User template
- `acme-corp/letterhead:stable` - Organization template
- `john/invoice:latest@sha256:abc123` - Tag with hash verification

## Storage Layout

```
storage/
├── blobs/sha256/{hash}           # Individual files
├── manifests/sha256/{hash}       # Template manifests
└── refs/
    ├── invoice/latest            # Official: name/tag
    ├── john/invoice/latest       # User: user/name/tag
    └── acme-corp/invoice/stable  # Org: org/name/tag
```

## Manifest Format

```json
{
  "entrypoint": "main.typ",
  "files": {
    "main.typ": "sha256:abc123...",
    "schema.json": "sha256:def456...",
    "components/header.typ": "sha256:ghi789...",
    "assets/logo.png": "sha256:jkl012..."
  },
  "metadata": {
    "name": "invoice-template",
    "author": "john@example.com"
  }
}
```

## Implementation Tasks

### Immediate Next Steps

1. **Implement `Manifest` struct** (`papermake-registry/src/manifest.rs`)
   ```rust
   #[derive(Serialize, Deserialize)]
   pub struct Manifest {
       pub entrypoint: String,
       pub files: HashMap<String, String>, // filename -> hash
       pub metadata: TemplateMetadata,
   }

   pub struct TemplateMetadata {
       pub name: String,
       pub author: String,
   }
   ```

2. **Add manifest serialization tests**
   - JSON roundtrip testing
   - Validate required fields
   - Error handling for malformed manifests

3. **Implement `TemplateReference` parsing** (`papermake-registry/src/reference.rs`)
   ```rust
   pub struct TemplateReference {
       pub namespace: String,
       pub tag: Option<String>,
       pub hash: Option<String>,
   }

   impl std::str::FromStr for TemplateReference { /* ... */ }
   ```

### Current Development Focus

**Working on:** Registry file system integration to resolve Typst imports directly through blob storage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rkstgr/papermake](https://github.com/rkstgr/papermake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
