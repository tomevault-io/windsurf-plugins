---
trigger: always_on
description: This document captures key learnings and decisions made during the development of krust with Claude.
---

# krust Development Notes

This document captures key learnings and decisions made during the development of krust with Claude.

## Project Overview

krust is a container image build tool for Rust applications, inspired by ko.build for Go. It builds static binaries and packages them into minimal OCI container images without requiring Docker.

## Key Design Decisions

### 1. Static Binaries with musl

We chose musl libc over glibc for static linking because:
- **True static linking**: glibc uses dynamic loading internally (NSS) which breaks in static binaries
- **Smaller binaries**: musl static binaries are 5-10x smaller than glibc
- **No runtime surprises**: glibc static binaries often fail with DNS resolution, user lookups, or locale issues
- **Container-optimized**: Perfect for minimal container images

### 2. Default Push Behavior

krust pushes images by default (use `--no-push` to skip) because:
- Aligns with the common workflow of building and immediately using images
- Enables the `docker run $(krust build)` pattern
- Reduces friction for the most common use case

### 3. Output Design

- **stdout**: Only the pushed image reference by digest (e.g., `ttl.sh/user/app@sha256:...`)
- **stderr**: All logging and progress information
- This enables composability with other tools

### 4. Image Naming Strategy

- Uses `KRUST_REPO` environment variable for repository prefix
- Automatically appends project name from Cargo.toml
- Can be overridden with `--image` flag
- Default tag is `latest`

## Technical Learnings

### OCI Image Building

1. **Layer Digest vs Diff ID**:
   - Layer digest: SHA256 of the compressed (gzip) layer
   - Diff ID: SHA256 of the uncompressed tar (goes in image config)
   - Docker validates these match during pull

2. **Image Structure**:
   ```
   Manifest -> Config + Layers
   Config contains: architecture, OS, environment, command, diff_ids
   Layers contain: compressed tar.gz files
   ```

3. **Registry API**:
   - Push blobs (config and layers) first
   - Then push manifest referencing those blobs
   - Manifest URL contains the final digest

### Google Artifact Registry (GAR) Blob Uploads

GAR has special handling for blob uploads that differs from the standard OCI spec:

1. **Location Header Format**:
   - POST to `/v2/.../blobs/uploads/` returns `location: /artifacts-uploads/...`
   - The location is a relative path starting with `/artifacts-uploads/`, NOT `/v2/`
   - Must build absolute URL as `https://{registry}{location}` for ANY path starting with `/`

2. **Upload Flow** (Resumable):
   - **Monolithic upload not supported**: PUT with body to `location?digest=` returns `301 Moved Permanently`
   - Must use resumable upload flow instead:
     1. POST to `/v2/.../blobs/uploads/` → get upload location
     2. PATCH to upload location with blob data → returns `202 Accepted`
     3. PUT to finalize location with `?digest=` and empty body → returns `201 Created`

3. **Critical Implementation Details**:
   - GAR returns `301` redirect on monolithic PUT attempts (not `307`)
   - HTTP spec says `301` means don't resend body, so automatic redirect following fails
   - Must explicitly handle `301` as a signal to switch to resumable upload
   - PATCH response may include a new `location` header for the finalize PUT
   - Use `reqwest` with `redirect::Policy::none()` to handle redirects manually

4. **Why reqwest over hyper**:
   - Initially used raw `hyper` but it doesn't auto-follow redirects with request bodies
   - Switched to `reqwest` for cleaner API and better redirect handling
   - Disabled automatic redirects to manually handle GAR's upload flow

5. **Memory Efficiency**:
   - Blob downloads return `bytes::Bytes` instead of `Vec<u8>` to avoid unnecessary copies
   - Blob uploads require `.to_vec()` due to reqwest's `'static` requirement for request bodies
   - This is acceptable as reqwest streams the data internally

### Cross-Compilation

krust requires `cargo-zigbuild` for cross-compilation. This eliminates the need for
per-target system linkers and `.cargo/config.toml` linker configuration. If zigbuild is not
available, krust fails with install instructions.

Required targets are auto-installed via `rustup target add` when needed.

### Rust Static Linking

- Use `RUSTFLAGS="-C target-feature=+crt-static"` for static linking
- musl targets default to static, but explicit is better
- The resulting binary has no runtime dependencies

## Architecture Decisions

### Module Structure

```
src/
├── main.rs          # CLI entry point and orchestration
├── lib.rs           # Public API exports
├── cli/             # Command-line interface definitions
├── builder/         # Rust compilation logic
├── image/           # OCI image construction
├── registry/        # Registry push operations
└── config/          # Configuration management
```

### Error Handling

- Used `anyhow` for error propagation with context
- Errors include contextual information for debugging
- All errors go to stderr, preserving stdout for output

### Dependencies

Key crates chosen:
- `clap` - CLI parsing with derive macros
- `tokio` - Async runtime for registry operations
- `reqwest` - HTTP client with automatic redirect handling
- `tar` + `flate2` - Layer creation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjasonh/krust](https://github.com/imjasonh/krust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
