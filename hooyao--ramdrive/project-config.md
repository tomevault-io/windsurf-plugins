---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Build
dotnet build

# Run (debug, from src/RamDrive.Cli)
dotnet run --project src/RamDrive.Cli -- --RamDrive:MountPoint="R:\\" --RamDrive:CapacityMb=64

# Run tests (unit + integration — requires WinFsp installed)
dotnet test

# AOT publish (requires Visual Studio C++ Build Tools + vswhere in PATH)
dotnet publish src/RamDrive.Cli/RamDrive.Cli.csproj -c Release -r win-x64 -o ./publish-aot
```

**Prerequisites:** .NET 10 SDK, [WinFsp](https://winfsp.dev/rel/) 2.x (install with Developer files).

## Architecture

```
WinFspRamAdapter (IFileSystem — all WinFsp callbacks, zero managed heap alloc on hot path)
    │
    ▼
RamFileSystem (directory tree, path resolution, global structure lock)
    │
    ▼
FileNode (file/dir metadata + PagedFileContent for files)
    │
    ▼
PagedFileContent (per-file nint[] page table + ReaderWriterLockSlim)
    │
    ▼
PagePool (NativeMemory.AllocZeroed + ConcurrentStack<nint> free list)
```

The WinFsp binding is provided by the [`WinFsp.Native`](https://www.nuget.org/packages/WinFsp.Native) NuGet package (namespace `WinFsp.Native`), which offers:
- `FileSystemHost` — high-level host bridging `IFileSystem` to native WinFsp
- `WinFspFileSystem` — low-level API for direct function pointer manipulation
- Full AOT-compatible P/Invoke layer with `[LibraryImport]` source generators

**All file data lives in NativeMemory (outside GC heap).** This is the core design decision — zero GC pressure for I/O operations.

## Key Design Decisions

### PagePool (Memory/PagePool.cs)
- Fixed-size pages (default 64KB) allocated via `NativeMemory.AllocZeroed`
- `ConcurrentStack<nint>` as lock-free LIFO free list
- `RentBatch`/`ReturnBatch` use `TryPopRange`/`PushRange` for single-CAS multi-page operations
- Two modes: on-demand allocation (lazy) or pre-allocate all pages at startup
- CAS loop for thread-safe capacity enforcement without locks

### PagedFileContent (Memory/PagedFileContent.cs)
- `nint[]` page table — index maps to native page pointer, `nint.Zero` = sparse (unallocated)
- **Three-phase write** to minimize write-lock hold time:
  1. Read lock: scan which pages need allocation
  2. No lock: batch-allocate pages from PagePool via `RentBatch`. On failure return pages and report DISK_FULL.
  3. Write lock: assign page table entries + memcpy only. Race fallback: if a page was allocated by a concurrent writer between Phase 1 and 3, use single `Rent()`.
- **Sparse SetLength**: extending a file only expands the page table (`nint.Zero` entries); no pages are reserved or allocated. Pages are allocated on demand in Write.
- Full protocol verified with TLA+ — see `tla/RamDiskSystem.tla`.
- `ReaderWriterLockSlim` per file — concurrent reads don't block each other
- Truncation zeros partial page data and batch-returns freed pages

### RamFileSystem (FileSystem/RamFileSystem.cs)
- Single global `_structureLock` for all tree mutations (create/delete/move)
- `Dictionary<string, FileNode>` with `StringComparer.OrdinalIgnoreCase` for Windows paths
- Path format: backslash separated, root is `"\"`

### WinFspRamAdapter (RamDrive.Cli/WinFspRamAdapter.cs)
- Implements `WinFsp.Native.IFileSystem` for use with `FileSystemHost`
- Caches `FileNode` in `FileOperationInfo.Context` to avoid repeated path lookups
- Deletion is deferred: `CanDelete` validates; actual removal in `Cleanup` when `CleanupFlags.Delete`
- `WriteFile`: `writeToEndOfFile` flag means append; `constrainedIo` clamps to logical size
- `GetVolumeInfo`: reports "NTFS" with `PersistentAcls | CasePreservedNames | UnicodeOnDisk`
- All hot-path methods return synchronous `ValueTask<T>` — zero managed heap allocation
- Timestamps use FILETIME (ulong) via `DateTime.ToFileTimeUtc()`

## WinFsp Notes

- `fileSystemName` must be `"NTFS"` for elevated process compatibility.
- `GetFileSecurityByName` returns the file's stored security descriptor. Root default: `O:BAG:BAD:P(A;OICI;FA;;;SY)(A;OICI;FA;;;BA)(A;OICI;FA;;;WD)`. `GetFileSecurity`/`SetFileSecurity` implemented via `RawSecurityDescriptor` merge. The `OICI` ACE flag = `OBJECT_INHERIT_ACE | CONTAINER_INHERIT_ACE`; without it WinFsp's kernel-side `FspCreateSecurityDescriptor` drops every ACE when computing the SD for a newly-created child, leaving children with an empty DACL and reopen returning `ACCESS_DENIED`. See spec `default-security-descriptor`.
- `ReadDirectory` uses native buffer with `WinFspFileSystem.AddDirInfo` / `EndDirInfo` — no IEnumerable allocation.
- `SetMountPointEx` crashes on `"R:\"` (trailing backslash). Mount point format: `"R:"` = `DefineDosDevice` (invisible to disk tools), `"\\.\R:"` = Mount Manager (visible to all apps, requires admin or `MountUseMountmgrFromFSD=1` registry key). `WinFspHostedService` tries `\\.\R:` first, falls back to `R:` on failure.
- **STATUS_PENDING async**: Must build a fresh stack-allocated `FspTransactRsp` with `Size`/`Kind`/`Hint` fields. Do NOT reuse `OperationContext->Response` — it may be invalidated after returning `STATUS_PENDING`. Save `Request->Hint` before returning, echo it back in the response.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hooyao/RamDrive](https://github.com/hooyao/RamDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
