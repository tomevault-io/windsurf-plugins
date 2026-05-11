---
trigger: always_on
description: Cross-platform native sandbox for running arbitrary commands in isolated environments.
---

# tokimo-package-sandbox

Cross-platform native sandbox for running arbitrary commands in isolated environments.

- **Linux**: bubblewrap (`bwrap`) + seccomp-bpf with optional eBPF L4 observer
- **macOS**: Apple Seatbelt (VZVirtualMachine / `arcbox-vz`)
- **Windows**: Hyper-V Host Compute Service (HCS) via a client-service architecture

## Architecture (Windows)

```
host process (library)  ──named pipe──▶  tokimo-sandbox-svc.exe (LocalSystem)
                                                │
                                                └── ComputeCore.dll (HCS API) ──▶ Hyper-V micro-VM
```

The library (`src/windows/`) connects to the SYSTEM service over `\\.\pipe\tokimo-sandbox-svc` using a JSON length-prefixed wire protocol (`src/windows/protocol.rs`). The service (`src/bin/tokimo-sandbox-svc/`) boots a Linux kernel+initrd (with a per-session VHDX clone for rootfs isolation) via HCS Schema 2.x, mounts the workspace over Plan9/vsock, and bridges the init control protocol over AF_HYPERV/HvSocket back to the library.

Two deployment modes:
- **MSIX** (`packaging/windows/`, `scripts/build-msix.ps1`): recommended for production — registers service name `TokimoSandboxSvc` via `desktop6:Service`.
- **CLI install** (`--install` / `--uninstall`): registers service name `tokimo-sandbox-svc` (lowercase-kebab) — for development. The two names are intentionally different so both can coexist on the same machine.
- **Console mode** (`--console`): foreground dev mode, no SCM registration needed.

## Windows APIs — all through the `windows` crate (verified)

**No hand-written FFI, no manual `extern "system"` blocks.** Every Win32 call goes through the `windows = "0.62"` crate. The only exception is `ComputeCore.dll` (HCS API), loaded dynamically via the `windows` crate's own `LoadLibraryW` + `GetProcAddress`.

The verified API surface, grouped by file:

### `src/windows/client.rs` (library-side named pipe client)

| Crate feature | Items used |
|---|---|
| `Win32_Foundation` | `ERROR_PIPE_BUSY`, `GENERIC_READ`, `GENERIC_WRITE`, `GetLastError` |
| `Win32_Security` | `SECURITY_ATTRIBUTES` |
| `Win32_Storage_FileSystem` | `CreateFileW`, `FILE_FLAGS_AND_ATTRIBUTES`, `FILE_SHARE_NONE`, `OPEN_EXISTING` |
| `Win32_System_Pipes` | `WaitNamedPipeW` |
| `windows::core` | `HSTRING` |
| std | `std::os::windows::io::FromRawHandle` |

### `src/windows/safe_path.rs` (TOCTOU-safe canonicalization)

| Crate feature | Items used |
|---|---|
| `Win32_Foundation` | `CloseHandle`, `GENERIC_READ`, `HANDLE` |
| `Win32_Storage_FileSystem` | `BY_HANDLE_FILE_INFORMATION`, `CreateFileW`, `FILE_ATTRIBUTE_REPARSE_POINT`, `FILE_FLAG_BACKUP_SEMANTICS`, `FILE_FLAG_OPEN_REPARSE_POINT`, `FILE_SHARE_DELETE`, `FILE_SHARE_READ`, `FILE_SHARE_WRITE`, `GetFileInformationByHandle`, `OPEN_EXISTING` |
| `windows::core` | `HSTRING` |

### `src/bin/tokimo-sandbox-svc/imp/mod.rs` (service main)

| Crate feature | Items used |
|---|---|
| `Win32_Foundation` | `CloseHandle`, `GetLastError`, `HANDLE`, `HLOCAL`, `HWND`, `INVALID_HANDLE_VALUE`, `LocalFree` |
| `Win32_Security` | `SECURITY_ATTRIBUTES`, `PSECURITY_DESCRIPTOR` |
| `Win32_Security_Authorization` | `ConvertStringSecurityDescriptorToSecurityDescriptorW`, `SDDL_REVISION_1` |
| `Win32_Security_WinTrust` | `WinVerifyTrust`, `WINTRUST_ACTION_GENERIC_VERIFY_V2`, `WINTRUST_DATA`, `WINTRUST_DATA_0`, `WINTRUST_DATA_PROVIDER_FLAGS`, `WINTRUST_DATA_REVOCATION_CHECKS`, `WINTRUST_DATA_STATE_ACTION`, `WINTRUST_DATA_UICONTEXT`, `WINTRUST_FILE_INFO`, `WTD_CHOICE_FILE`, `WTD_UI_NONE` |
| `Win32_Storage_FileSystem` | `FlushFileBuffers`, `PIPE_ACCESS_DUPLEX`, `ReadFile`, `WriteFile` |
| `Win32_System_Pipes` | `ConnectNamedPipe`, `CreateNamedPipeW`, `DisconnectNamedPipe`, `GetNamedPipeClientProcessId`, `PIPE_READMODE_MESSAGE`, `PIPE_TYPE_MESSAGE`, `PIPE_UNLIMITED_INSTANCES`, `PIPE_WAIT` |
| `Win32_System_Registry` | `HKEY`, `HKEY_LOCAL_MACHINE`, `KEY_READ`, `REG_VALUE_TYPE`, `RegCloseKey`, `RegOpenKeyExW`, `RegQueryValueExW` |
| `Win32_System_Threading` | `OpenProcess`, `PROCESS_NAME_FORMAT`, `PROCESS_QUERY_LIMITED_INFORMATION`, `QueryFullProcessImageNameW` |
| `windows::core` | `HSTRING`, `PCWSTR`, `PWSTR` |
| std | `std::os::windows::ffi::EncodeWide`, `std::os::windows::ffi::OsStrExt` |

### `src/bin/tokimo-sandbox-svc/imp/hcs.rs` (ComputeCore.dll loader)

| Crate feature | Items used |
|---|---|
| `Win32_Foundation` | `FreeLibrary`, `HLOCAL`, `HMODULE`, `LocalFree` |
| `Win32_System_LibraryLoader` | `GetProcAddress`, `LoadLibraryW` |
| `windows::core` | `HSTRING`, `PCSTR` |

### `windows-service` crate (SCM integration)

From `windows-service = "0.8"` (only used in `imp/mod.rs`):

- `windows_service::service_dispatcher::start`
- `windows_service::define_windows_service!`
- `windows_service::service::{ServiceAccess, ServiceControl, ServiceControlAccept, ServiceErrorControl, ServiceExitCode, ServiceInfo, ServiceStartType, ServiceState, ServiceStatus, ServiceType}`
- `windows_service::service_control_handler::{self, ServiceControlHandlerResult}`
- `windows_service::service_manager::{ServiceManager, ServiceManagerAccess}`

### Cargo features declared but **NOT used** in source


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tokimo-lab/tokimo-package-sandbox](https://github.com/tokimo-lab/tokimo-package-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
