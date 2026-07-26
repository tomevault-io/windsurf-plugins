---
trigger: always_on
description: SSH.NET is a .NET library for SSH-2 protocol communication, optimized for parallelism. It provides:
---

# GitHub Copilot Instructions for SSH.NET

## What this repository does

SSH.NET is a .NET library for SSH-2 protocol communication, optimized for parallelism. It provides:

- **SSH command execution** (synchronous and async)
- **SFTP** file operations (synchronous and async)
- **SCP** file transfers
- **Port forwarding** (local, remote, dynamic/SOCKS)
- **Interactive shell** via `ShellStream`
- **NetConf** protocol client
- **Multi-factor and certificate-based authentication**

Primary entry points are `SshClient`, `SftpClient`, `ScpClient`, and `NetConfClient`, all extending `BaseClient`.

---

## Core technologies

| Area | Technology |
|---|---|
| Language | C# (`LangVersion=latest`) with `#nullable enable` everywhere |
| Runtimes | .NET Framework 4.6.2, .NET Standard 2.0, .NET 8+, .NET 9+, .NET 10+ |
| Cryptography | BouncyCastle (`BouncyCastle.Cryptography`) |
| Logging | `Microsoft.Extensions.Logging.Abstractions` (`ILogger`/`ILoggerFactory`) |
| Testing | MSTest v4, Moq, Testcontainers (Docker for integration tests) |
| Build tooling | .NET SDK 10.0.100, Nerdbank.GitVersioning |
| Static analysis | StyleCop.Analyzers, Meziantou.Analyzer, SonarAnalyzer.CSharp |

---

## Code organization

```
src/Renci.SshNet/
├── Channels/          # SSH channel types (session, forwarded, X11…)
├── Common/            # Shared utilities, extension methods, custom exceptions
├── Compression/       # Zlib compression support
├── Connection/        # Socket connectors, proxy support (SOCKS4/5, HTTP), key exchange
├── Messages/          # SSH protocol message types
│   ├── Transport/
│   ├── Authentication/
│   └── Connection/
├── Security/          # Algorithms, key-exchange, cryptography wrappers
│   └── Cryptography/  # Ciphers, signatures, key types
├── Sftp/              # SFTP session, requests, responses
├── Netconf/           # NetConf client
└── Abstractions/      # Platform and I/O abstractions
```

Large classes are split into **partial class files** per concern (e.g., `PrivateKeyFile.PKCS1.cs`, `PrivateKeyFile.OpenSSH.cs`).

---

## Naming and style conventions

| Element | Convention | Example |
|---|---|---|
| Classes, methods, properties | PascalCase | `SftpClient`, `ListDirectory()`, `IsConnected` |
| Private fields | `_camelCase` | `_isDisposed`, `_sftpSession` |
| Interfaces | `I` prefix + PascalCase | `ISftpClient`, `IChannel` |
| Constants / static readonly | PascalCase | `MaximumSshPacketSize` |
| Local variables | camelCase | `operationTimeout`, `connectionInfo` |

**StyleCop is enforced.** Follow existing file conventions:

- `#nullable enable` at the top of every `.cs` file.
- `using` directives **outside** the namespace block, grouped with `System.*` first, then a blank line, then other namespaces.
- 4-space indentation (spaces, not tabs).
- XML doc comments (`///`) on all public and internal members; use `<inheritdoc/>` when inheriting from an interface.
- Describe exception conditions in `/// <exception cref="…">` tags.
- No Hungarian notation.

---

## Error handling

Use the existing exception hierarchy; do not throw `Exception` or `ApplicationException` directly.

```
SshException
├── SshConnectionException       # connection-level failures
├── SshAuthenticationException   # auth failures
├── SshOperationTimeoutException # operation timed out
├── SshPassPhraseNullOrEmptyException
├── ProxyException
├── SftpException
│   ├── SftpPermissionDeniedException
│   └── SftpPathNotFoundException
├── ScpException
└── NetConfServerException
```

- Annotate new exception classes with `#if NETFRAMEWORK [Serializable] #endif` and add the protected serialization constructor inside the same `#if` block, matching the pattern in `SshException.cs`.
- Surface async errors by storing them in a `TaskCompletionSource` or re-throwing via `ExceptionDispatchInfo.Throw()` — never swallow exceptions silently.
- Raise `ErrorOccurred` events on long-running components (e.g., `ForwardedPort`, `Session`) rather than propagating the exception across thread boundaries.

---

## API design

- **Every public blocking method should have a `…Async(CancellationToken cancellationToken = default)` counterpart.** Keep both in the same partial class file or co-located partial file.
- Validate all arguments at the top of public methods; prefer `ArgumentNullException`, `ArgumentException`, `ArgumentOutOfRangeException` with descriptive messages.
- Return `IEnumerable<T>` for sequences that are already materialized; use `IAsyncEnumerable<T>` when data streams asynchronously.
- Prefer `ReadOnlyMemory<byte>` / `Memory<byte>` over `byte[]` in new protocol-layer code.
- Do not expose mutable collections directly; use `.AsReadOnly()` or copy-on-return.

---

## Async and concurrency

- Use `async Task` / `async ValueTask` with `CancellationToken` for all new async methods.
- The socket receive loop and keep-alive timer run on dedicated background threads; protect shared state with `lock` or the custom internal `Lock` type used in `Session`.
- Prefer `TaskCompletionSource<T>` to bridge event-driven or callback-based code into the async model.
- Never block inside async code with `.Result` or `.Wait()` — this can cause deadlocks on synchronization-context-bound callers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sshnet/SSH.NET](https://github.com/sshnet/SSH.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
