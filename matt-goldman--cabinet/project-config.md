---
trigger: always_on
description: Cabinet is a lightweight, encrypted offline data layer for .NET applications, primarily .NET MAUI but useful for other .NET platforms as well. It provides structured file storage with encryption at rest, full-text search capabilities, and cross-platform reliability without requiring native database dependencies like SQLite or Realm.
---

# GitHub Copilot Instructions for Cabinet

## Project Overview

Cabinet is a lightweight, encrypted offline data layer for .NET applications, primarily .NET MAUI but useful for other .NET platforms as well. It provides structured file storage with encryption at rest, full-text search capabilities, and cross-platform reliability without requiring native database dependencies like SQLite or Realm.

**Key characteristics:**
- Pure .NET implementation
- AOT-safe (no JIT or reflection dependencies)
- Cross-platform (.NET MAUI: Android, iOS, Windows, macOS, Catalyst)
- Security-first design with AES-256-GCM encryption
- Simple, developer-friendly API

## Architecture

The project follows a clean, modular architecture:

```
src/
├── Abstractions/       # Core interfaces (IOfflineStore, IEncryptionProvider, IIndexProvider)
├── Core/              # Main implementations (FileOfflineStore, data models)
├── Security/          # Encryption providers (AesGcmEncryptionProvider)
└── Utilities/         # Helper classes and extensions
```

**Data storage structure:**
```
/AppData/
 ├── records/          # Encrypted JSON records
 ├── attachments/      # Encrypted binary attachments
 ├── index/           # Encrypted search index
 └── summary/         # Encrypted metadata summaries
```

**Core design patterns:**
- Interface-based abstraction for extensibility
- Dependency injection friendly
- Atomic file writes (write to .tmp, then rename)
- Per-file encryption with HKDF key derivation
- No plaintext ever written to disk

## Development Guidelines

**Always search Microsoft documentation (MS Learn) when working with .NET, Windows, or Microsoft features, or APIs.** Reference official Microsoft Learn documentation to find the most current information about capabilities, best practices, and implementation patterns before making changes.

### .NET Version Requirements

- **Minimum .NET version: .NET 9**
- **Never downgrade .NET versions**
- Development environments must always be upgraded to use .NET 9 or later
- The project targets: `net9.0`

### Code Style and Formatting

- **Don't commit formatting-only changes** unless specifically requested
- **Use tab spacing in code** (follow existing indentation patterns in the codebase)
- **Code comments:** Use Australian English spelling (e.g., "tokenise", "colour", "serialise")
- **Code identifiers:** Use US English spelling (e.g., "Tokenize", "Color", "Serialize" in class/method/variable names)
- **Documentation:** Use Australian English spelling in all non-code content (README.md, markdown docs, XML comments)

### .NET MAUI Best Practices

- Follow MVVM pattern recommendations where applicable
- Leverage platform-specific APIs through .NET MAUI abstractions
- Use `FileSystem.AppDataDirectory` for cross-platform file storage
- Utilize `SecureStorage` for sensitive data like encryption keys
- Ensure AOT compatibility (avoid reflection, dynamic code generation)

### Security and Encryption

- All data must be encrypted at rest using authenticated encryption (AES-GCM)
- Master keys are stored in platform `SecureStorage`
- Per-file keys derived using HKDF with file ID as context
- Atomic writes prevent partial/corrupted data on disk
- Never write decrypted content to disk, only to memory
- File extensions: `.dat` for encrypted data, `.tmp` for in-progress writes

### API Design

- Keep interfaces simple and focused
- Use async/await for all I/O operations
- Generic types for flexibility (`Task<T?>`, `SaveAsync<T>`)
- Optional parameters for extensibility (e.g., `attachments` parameter)
- Return meaningful types (`SearchResult`, not raw dictionaries)

### Testing Considerations

- Unit tests should mock `IEncryptionProvider` and `IIndexProvider`
- Integration tests should verify encryption/decryption round-trips
- Test atomic write behaviour (crash/failure scenarios)
- Verify cross-platform file path handling
- Test with various data types and edge cases (null, empty, large files)

### Dependencies

- Minimize external dependencies
- Use .NET BCL (Base Class Library) whenever possible
- Avoid platform-specific code in core abstractions
- Plugin architecture allows custom implementations (indexers, encryption)

### Documentation

- Update README.md when adding new features or changing APIs
- Keep `_docs/Specification.md` aligned with implementation
- Use code examples in Australian English prose
- API documentation should be clear and include usage examples

## .NET MAUI Specific Guidance

When working with .NET MAUI features:

1. **Platform APIs:** Access through .NET MAUI abstractions first, fall back to platform-specific code only when necessary
2. **File System:** Use `FileSystem` class from `Microsoft.Maui.Storage` namespace
3. **Secure Storage:** Use `SecureStorage` class for sensitive data persistence
4. **Cross-platform paths:** Always use `Path.Combine()` for path construction
5. **Lifecycle:** Consider app lifecycle events for background operations

## Extensibility Points


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matt-goldman/Cabinet](https://github.com/matt-goldman/Cabinet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
