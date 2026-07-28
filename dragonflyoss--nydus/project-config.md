---
trigger: always_on
description: Nydus is a high-performance container image service that implements a content-addressable file system on the RAFS format. It enhances the OCI image specification by enabling on-demand loading, chunk-level deduplication, and improved container startup performance.
---

# GitHub Copilot Instructions for Nydus

## Project Overview

Nydus is a high-performance container image service that implements a content-addressable file system on the RAFS format. It enhances the OCI image specification by enabling on-demand loading, chunk-level deduplication, and improved container startup performance.

### Key Components

- **nydusd**: User-space daemon that processes FUSE/fscache/virtiofs messages and serves Nydus images
- **nydus-image**: CLI tool to convert OCI image layers to Nydus format
- **nydusify**: Tool to convert entire OCI images to Nydus format with registry integration
- **nydusctl**: CLI client for managing and querying nydusd daemon
- **nydus-service**: Library crate for integrating Nydus services into other projects

## Architecture Guidelines

### Crate Structure
```
- api/          # Nydus Image Service APIs and data structures
- builder/      # Image building and conversion logic
- rafs/         # RAFS filesystem implementation
- service/      # Daemon and service management framework
- storage/      # Core storage subsystem with backends and caching
- utils/        # Common utilities and helper functions
- src/bin/      # Binary executables (nydusd, nydus-image, nydusctl)
```

### Key Technologies
- **Language**: Rust with memory safety focus
- **Filesystems**: FUSE, virtiofs, EROFS, fscache
- **Storage Backends**: Registry, OSS, S3, LocalFS, HTTP proxy
- **Compression**: LZ4, Gzip, Zstd
- **Async Runtime**: Tokio (current thread for io-uring compatibility)

## Code Style and Patterns

### Rust Conventions
- Use `#![deny(warnings)]` in all binary crates
- Follow standard Rust naming conventions (snake_case, PascalCase)
- Prefer `anyhow::Result` for error handling in applications
- Use custom error types with `thiserror` for libraries
- Apply `#[macro_use]` for frequently used external crates like `log`
- Always format the code with `cargo fmt`
- Use `clippy` for linting and follow its suggestions

### Error Handling
```rust
// Prefer anyhow for applications
use anyhow::{bail, Context, Result};

// Use custom error types for libraries
use thiserror::Error;

#[derive(Error, Debug)]
pub enum NydusError {
    #[error("Invalid arguments: {0}")]
    InvalidArguments(String),
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
}
```

### Logging Patterns
- Use structured logging with appropriate levels (trace, debug, info, warn, error)
- Include context in error messages: `.with_context(|| "description")`
- Use `info!`, `warn!`, `error!` macros consistently

### Configuration Management
- Use `serde` for JSON configuration serialization/deserialization
- Support both file-based and environment variable configuration
- Validate configurations at startup with clear error messages
- Follow the `ConfigV2` pattern for versioned configurations

## Development Guidelines

### Storage Backend Development
- When implementing new storage backends:
- - Implement the `BlobBackend` trait
- - Support timeout, retry, and connection management
- - Add configuration in the backend config structure
- - Consider proxy support for high availability
- - Implement proper error handling and logging  

### Daemon Service Development
- Use the `NydusDaemon` trait for service implementations
- Support save/restore for hot upgrade functionality
- Implement proper state machine transitions
- Use `DaemonController` for lifecycle management

### RAFS Filesystem Features
- Support both RAFS v5 and v6 formats
- Implement chunk-level deduplication
- Handle prefetch optimization for container startup
- Support overlay filesystem operations
- Maintain POSIX compatibility

### API Development
- Use versioned APIs (v1, v2) with backward compatibility
- Implement HTTP endpoints with proper error handling
- Support both Unix socket and TCP communication
- Follow OpenAPI specification patterns

## Testing Patterns

### Unit Tests
- Test individual functions and modules in isolation
- Use `#[cfg(test)]` modules within source files
- Mock external dependencies when necessary
- Focus on error conditions and edge cases

### Integration Tests
- Place integration tests in `tests/` directory
- Test complete workflows and component interactions
- Use temporary directories for filesystem operations
- Clean up resources properly in test teardown

### Smoke Tests
- Located in `smoke/` directory using Go
- Test real-world scenarios with actual images
- Verify performance and functionality
- Use Bats framework for shell-based testing

## Performance Considerations

### I/O Optimization
- Use async I/O patterns with Tokio
- Implement prefetching for predictable access patterns
- Optimize chunk size (default 1MB) for workload characteristics
- Consider io-uring for high-performance scenarios

### Memory Management
- Use `Arc<T>` for shared ownership of large objects
- Implement lazy loading for metadata structures
- Consider memory mapping for large files
- Profile memory usage in performance-critical paths

### Caching Strategy
- Implement blob caching with configurable backends

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dragonflyoss/nydus](https://github.com/dragonflyoss/nydus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
