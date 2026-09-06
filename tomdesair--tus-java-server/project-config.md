---
trigger: always_on
description: When working on this project, always read the [`README.md`](README.md) file to obtain full context on project architecture, features, configuration options, and dual protocol version support (Tus 1.0.0 & IETF RUFH).
---

# Agent Instructions

## Project Context
When working on this project, always read the [`README.md`](README.md) file to obtain full context on project architecture, features, configuration options, and dual protocol version support (Tus 1.0.0 & IETF RUFH).

## GitHub CLI (`gh`) & Git Usage
- Always run `git` commands (e.g., `git status`, `git diff`, `git add`, `git commit`, `git push`) and `gh` commands unsandboxed (setting `BypassSandbox: true` when calling `run_command`) to ensure git hooks, local tools, and remote repository authentication work without sandbox errors.
- When running `gh` commands in this project via an automated agent environment, ensure you bypass the default `GITHUB_TOKEN` environment variable. The agent environment may have an invalid `GITHUB_TOKEN` set, which `gh` prioritizes over valid keyring credentials, resulting in an `HTTP 401: Bad credentials` error.

**Workaround:** Prefix `gh` commands with `env -u GITHUB_TOKEN` to force the CLI to use the valid keyring authentication.

Example:
```bash
env -u GITHUB_TOKEN gh pr create --title "..." --body "..."
```

## Git Branching Strategy
Any new feature, bugfix, or improvement must be developed in a separate branch that starts with either `feature/` or `bugfix/` and has a meaningful but short name (e.g., `feature/lock-contention-resolution` or `bugfix/fix-upload-timeout`).

## Releases and Documentation

### README.md
When a new feature is introduced, the `README.md` file must be updated with information on this new feature (e.g. configuration, usage).

### CHANGELOG.md
For any new feature, big improvements, or fixes, the `CHANGELOG.md` file must be updated to describe the changes added in this version. Use a release version header (e.g., `## [1.0.0-3.2]`) instead of `## [Unreleased]`. Derive this next release version from the SNAPSHOT version declared in the `pom.xml` file by removing the `-SNAPSHOT` suffix. Make sure to not add duplicate headers.

### Release Process
When performing a release, please strictly follow the instructions outlined in the [docs/RELEASE.md](docs/RELEASE.md) documentation file.

## Developer Guidelines & Code Architecture

### 1. Spring Boot & Java Requirements
- **Java Version**: The project is configured for **Java 17** (or newer) to align with Spring Boot 3.x requirements.
- **Jakarta EE / Servlets**: Always use `jakarta.servlet.*` package imports instead of the legacy `javax.servlet.*` packages.

### 2. Extension Architecture & Protocol Applicability
- Every protocol extension MUST extend `AbstractTusExtension` and declare its applicability via `isApplicable(HttpMethod, ProtocolVersion)`.
- `TusFileUploadService` MUST NOT contain protocol-specific conditionals, version branching, or hardcoded error handling; all protocol-specific validation and execution logic belongs inside dedicated `RequestValidator` and `RequestHandler` implementations.

### 3. Explicit Parameter Passing & No Request Attributes
- Do NOT use magic string servlet request attributes (such as `"me.desair.tus.uploadLockingService"` or `"me.desair.tus.protocolVersion"`).
- Pass dependencies such as `UploadLockingService` and `ProtocolVersion` explicitly as typed method parameters through `TusExtension` and `RequestHandler` interface methods. When expanding interfaces, always use Java `default` methods to preserve backward compatibility.

### 4. Serializable UploadInfo & Backward Compatibility
- The `UploadInfo` class is stored on disk serialized. If you modify fields in `UploadInfo`, you **must** preserve the `serialVersionUID = -8751200491586638308L` to ensure pre-existing uploads on disk do not trigger `InvalidClassException` upon deserialization.
- Backward compatibility is paramount for this project. Breaking changes should only be done if all other options lead to ugly code and design. Breaking changes require a new major version.
- **Release Scope for Backward Compatibility**: Only maintain backward compatibility for classes, methods, or public API signatures that are present in the latest official Git release tag. Signatures, classes, or helper methods introduced in unreleased commits or feature branches do not require backward compatibility and should be refactored or deleted directly.

### 5. Lock Contention Resolution & InterruptibleInputStream
- Request handlers that stream payload bytes to storage (`CorePatchRequestHandler`, `RufhCreationPostRequestHandler`, `RufhAppendPatchRequestHandler`) MUST wrap body input streams in `InterruptibleInputStream` and register them via `lockingService.registerInputStream(...)`. This ensures concurrent `HEAD` and `DELETE` requests can interrupt ongoing byte streams cleanly and resolve lock contention.

### 6. Problem Details Value Objects & Structured JSON Serialization
- Model RFC 7807 problem details as immutable domain value objects (`HttpProblemDetails`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomdesair/tus-java-server](https://github.com/tomdesair/tus-java-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
