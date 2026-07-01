---
trigger: always_on
description: This file provides guidance for AI assistants working with the ftpserverlib codebase.
---

# CLAUDE.md

This file provides guidance for AI assistants working with the ftpserverlib codebase.

## Project Overview

**ftpserverlib** is a Go library for building FTP servers using [afero](https://github.com/spf13/afero) as the backend filesystem. It implements RFC 959 and numerous extensions, providing a clean, driver-based architecture for customization.

**Repository**: `github.com/fclairamb/ftpserverlib`

## Build and Test Commands

```bash
# Build
go build -v ./...

# Run tests (standard)
go test -v ./...

# Run tests with race detection (as CI does)
go test -parallel 20 -v -race -coverprofile=coverage.txt -covermode=atomic ./...

# Run linter (requires golangci-lint v2.4.0+)
golangci-lint run

# Format code
gofmt -w .
goimports -w -local github.com/fclairamb/ftpserverlib .
```

## Project Architecture

### Single Package Design

The entire library is a single `ftpserver` package with files organized by responsibility:

| File(s) | Purpose |
|---------|---------|
| `server.go` | Main `FtpServer` struct, initialization, listener management |
| `client_handler.go` | Per-client connection state machine and command parsing |
| `handle_auth.go` | USER, PASS, AUTH, PROT, PBSZ commands |
| `handle_dirs.go` | CWD, CDUP, MKD, RMD, PWD commands |
| `handle_files.go` | STOR, RETR, LIST, NLST, MLST, MLSD, DELE, SIZE, etc. |
| `handle_misc.go` | SYST, FEAT, NOOP, QUIT, SITE, STAT, HELP commands |
| `transfer_pasv.go` | Passive mode data connections (PASV, EPSV) |
| `transfer_active.go` | Active mode data connections (PORT, EPRT) |
| `driver.go` | All interface definitions (MainDriver, ClientDriver, extensions) |
| `consts.go` | FTP status codes and constants |
| `errors.go` | Custom error types (DriverError, NetworkError, FileAccessError) |
| `asciiconverter.go` | ASCII mode CRLF/LF conversion |

### Driver-Based Architecture

Users implement interfaces to customize server behavior:

```go
// Required: Main authentication and configuration
type MainDriver interface {
    GetSettings() (*Settings, error)
    ClientConnected(cc ClientContext) (string, error)
    ClientDisconnected(cc ClientContext)
    AuthUser(cc ClientContext, user, pass string) (ClientDriver, error)
    GetTLSConfig() (*tls.Config, error)
}

// Required: Filesystem operations (wraps afero.Fs)
type ClientDriver interface {
    afero.Fs
}
```

### Extension Pattern

Optional features use interface assertion:

```go
// In handler code:
if hasher, ok := c.driver.(ClientDriverExtensionHasher); ok {
    // Extension is supported, use it
    hash, err := hasher.ComputeHash(name, algo, start, end)
}
```

Available extensions:
- `MainDriverExtensionTLSVerifier` - TLS certificate authentication
- `MainDriverExtensionUserVerifier` - Pre-auth user validation
- `MainDriverExtensionPostAuthMessage` - Custom post-auth messages
- `MainDriverExtensionPassiveWrapper` - Wrap passive listeners
- `MainDriverExtensionQuitMessage` - Custom quit messages
- `ClientDriverExtensionAllocate` - ALLO command support
- `ClientDriverExtensionSymlink` - SITE SYMLINK support
- `ClientDriverExtensionFileList` - Custom directory listing
- `ClientDriverExtentionFileTransfer` - Custom file transfer handles
- `ClientDriverExtensionRemoveDir` - Distinguish RMD from DELE
- `ClientDriverExtensionHasher` - Custom hash implementations
- `ClientDriverExtensionAvailableSpace` - AVBL command support
- `ClientDriverExtensionSite` - Custom SITE subcommands

## Coding Conventions

### Naming
- Exported: `PascalCase` (e.g., `FtpServer`, `MainDriver`, `Settings`)
- Unexported: `camelCase` (e.g., `clientHandler`, `transferHandler`)
- Command handlers: `handle{COMMAND}` (e.g., `handleUSER`, `handleRETR`)
- Test files: `*_test.go` colocated with implementation

### Enums
Use `type X int8` with `iota`:
```go
type HASHAlgo int8
const (
    HASHAlgoCRC32 HASHAlgo = iota
    HASHAlgoMD5
    HASHAlgoSHA1
    HASHAlgoSHA256
    HASHAlgoSHA512
)
```

### Error Handling
- Use custom error types: `DriverError`, `NetworkError`, `FileAccessError`
- Wrap errors with context: `fmt.Errorf("operation failed: %w", err)`
- Check errors with `errors.Is()`: `errors.Is(err, ErrStorageExceeded)`
- Use `getErrorCode()` to map Go errors to FTP status codes

### Synchronization
- No global mutexes (only per-client)
- `paramsMutex` (RWMutex) protects public API fields in `clientHandler`
- `transferMu` protects transfer connection state
- `sync.WaitGroup` for command-to-transfer coordination

### Design Principles
- **No sleep**: Use proper synchronization, not time delays
- **No panic**: Propagate errors, don't crash
- **No global sync**: Each client manages its own state

## Linter Configuration

The project uses golangci-lint v2 with strict settings (`.golangci.yml`):

- **Line length**: 120 characters max
- **Function length**: 80 lines / 40 statements max
- **Cyclomatic complexity**: 15 max
- **Cognitive complexity**: 30 max
- **Import organization**: stdlib, third-party, then local (`github.com/fclairamb/ftpserverlib`)

Key enabled linters: `gosec`, `errcheck`, `errorlint`, `gocyclo`, `gocognit`, `funlen`, `dupl`, `unparam`, `staticcheck`

## Testing

### Test Infrastructure
- Tests use `github.com/stretchr/testify` (both `assert` and `require`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fclairamb/ftpserverlib](https://github.com/fclairamb/ftpserverlib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
