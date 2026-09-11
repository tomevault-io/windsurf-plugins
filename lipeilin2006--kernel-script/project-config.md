---
trigger: always_on
description: `kernel-script` is a Windows-only Rust workspace with four crates:
---

# kernel-script Agent Guide

## Project Scope

`kernel-script` is a Windows-only Rust workspace with four crates:

- `ks-core`: shared `no_std` protocol and ABI definitions.
- `ks-driver`: `no_std` WDM kernel driver. It performs target-process memory reads and writes.
- `ks-service`: SYSTEM user-mode service. It owns the TCP IPC server, driver handle, driver request dispatch, and user-mode process enumeration.
- `ks-gui`: user-mode egui/eframe OpenGL GUI and Lua runtime. It owns the Lua VM, coroutine scheduler, and asynchronous Named Pipe client.
- `ks-installer`: elevated egui GUI that manages the driver and service with `sc.exe` commands only.

`ks-installer` is an elevated egui GUI that manages the driver and backend
service with `sc.exe` only (`create`/`start`/`stop`/`delete`). It copies no
files; driver, service, GUI, and installer must sit in the same directory.

The deployment package uses this flat layout:

```text
driver-package/
├── ks-driver.sys
├── ks-service.exe
├── ks-gui.exe
└── ks-installer.exe
```

The intended data flow is:

```text
Lua coroutine / egui
    -> ks-gui Tokio TCP client
    -> ks-service Tokio TCP server
    -> driver worker / DeviceIoControl
    -> ks-driver
```

Process enumeration and process-name-to-PID lookup are service responsibilities. Do not add process enumeration back to the kernel driver unless there is a documented kernel-only requirement.

## Workspace Rules

- Keep `ks-core` dependency-free and `#![no_std]` compatible.
- Do not add Tokio, Lua, GUI, or user-mode Windows APIs to `ks-core` or `ks-driver`.
- Keep the driver limited to memory operations and the minimum required IOCTL surface.
- The driver device uses an explicit SYSTEM-only DACL (`D:P(A;;GA;;;SY)`). The service runs as SYSTEM; administrators and standard users must not open the device directly.
- The driver also binds the first successful device opener to its `EPROCESS`; subsequent create/control requests from another process are rejected. This is defense in depth, not a replacement for a service-specific DACL.
- Use explicit little-endian wire encoding. Do not expose Rust struct layout on the TCP protocol.
- Validate lengths, counts, addresses, PIDs, and frame sizes at every trust boundary.
- Use `windows-sys` with narrow feature lists when possible.
- Do not reintroduce removed synchronous Lua APIs. GUI Lua IPC APIs must remain asynchronous.
- Do not call blocking network operations, `block_on`, or synchronous driver operations from the GUI render thread.
- Lua VM objects must only be accessed by the GUI Lua thread. Never send `Lua`, `Thread`, `Function`, or registry keys to worker threads.
- Background workers may send only task IDs and owned plain data back to the GUI thread.

## GUI and Lua Lifecycle

The GUI frame lifecycle is:

```text
check_hot_reload
    -> poll/resume async Lua coroutines
    -> OnUpdate
    -> OnRender
    -> Lua GC
```

Rules:

- `OnRender` must only draw UI and read cached results.
- `OnUpdate` may start or poll asynchronous work but must not wait for network completion.
- Use `memory.async_*` plus `await_async` for process and memory operations.
- `start_async` creates a Lua coroutine and the Rust-side scheduler resumes it on later frames.
- Hot reload destroys the old Lua VM and therefore invalidates all old Lua coroutines.
- Multiple Lua scripts are loaded from `scripts/*.lua`; they run on the GUI Lua thread and share only scalar values explicitly stored through `shared.set/get/delete`.
- Do not let a coroutine hold an egui UI borrow across a yield.

Supported asynchronous Lua operations include:

```lua
memory.async_read_i32(pid, address)
memory.async_read_bytes(pid, address, size)
memory.async_read_rva(pid, relative_address, size)
memory.async_write_i32(pid, address, value)
memory.async_write_bytes(pid, address, data)
memory.async_write_rva(pid, relative_address, data)
memory.async_read_mdl(pid, address, size)
memory.async_write_mdl(pid, address, data)
memory.async_read_mdl_rva(pid, relative_address, size)
memory.async_write_mdl_rva(pid, relative_address, data)
memory.async_get_process_base(pid)
memory.async_list_processes()
memory.poll_async(task_id)
```

Typical usage:

```lua
start_async(function()
    local pid = await_async(memory.async_get_pid("notepad.exe"))
    local value = await_async(memory.async_read_i32(pid, "0x1407FFF0"))
    print(value)
end)
```

## IPC and Protocol

The GUI-to-service transport is the local Windows Named Pipe `\\.\pipe\KernelScript`.

- `ks-service` uses Tokio Windows Named Pipes and `BytesMut` for asynchronous framed reads.
- The pipe rejects remote clients and uses a bounded four-instance server. The
  transport type alone is not authentication; keep its Windows security
  descriptor restrictive if the service launch model changes.
- Complete frames should be transferred with `BytesMut::split_to(...).freeze()` where ownership is needed.
- Do not use `payload.to_vec()` merely to extend a frame lifetime.
- The service uses a blocking boundary for synchronous `DeviceIoControl` calls. This is expected; the GUI must never observe that blocking operation.
- Process enumeration uses Windows Toolhelp APIs in `ks-service/src/process.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lipeilin2006/kernel-script](https://github.com/lipeilin2006/kernel-script) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
