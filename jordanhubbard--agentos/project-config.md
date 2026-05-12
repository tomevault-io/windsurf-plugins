---
trigger: always_on
description: **This file is mandatory reading for any AI agent or developer working on this repository.**
---

# agentOS — Agent Development Guidelines

**This file is mandatory reading for any AI agent or developer working on this repository.**
**Violations of these rules will result in rejected PRs.**

---

## What This Project Is

agentOS is a **seL4 Microkit-based operating system for AI agents**. It runs on bare metal.
There is no libc. There is no POSIX. There is no userland in the traditional sense.
The entire OS is a set of cooperating **Protection Domains (PDs)** communicating via
seL4 IPC. Only seL4 itself runs in Ring 0. Everything else — every OS service, every
device driver, every guest OS manager — runs in outer rings as Microkit PDs.

This is not a Linux distro. This is not a container runtime. If you don't understand
seL4 Microkit IPC, read the [seL4 Microkit manual](https://trustworthy.systems/projects/microkit/manual.html)
before writing any code.

---

## The Rules

### 1. NO UI IN THIS REPOSITORY

There is no HTML. No JavaScript. No CSS. No ncurses. No interactive terminal UI.
No dashboard. No web interface. No WebSocket bridges. **Nothing that a human looks at.**

agentOS is for agents. Agents use APIs. If someone wants a UI, they build it as an
**external project** that consumes agentOS IPC contracts. That external project is not
this repository.

The **only** exception is `agentctl` — a CLI build/launch/management tool that prints
structured output (JSON or tab-separated) to stdout and exits. It is a tool, not a UI.

**What this means in practice:**
- Do not add `.html`, `.css`, `.js`, `.mjs`, `.jsx`, `.tsx`, `.vue`, `.svelte` files. Ever.
- Do not add `package.json`, `node_modules`, `yarn.lock`, `bun.lockb`. Ever.
- Do not add ncurses, readline, or any interactive input handling to PD code.
- Do not add "pretty printing", progress bars, or human-readable formatting to PD output.
- PD output goes through `MSG_LOG_WRITE` to the log drain. Period.

### 2. PURE ASSEMBLY + C + RUST

The core of agentOS is written in:
- **Assembly** (seL4 bootstrap, architecture-specific stubs)
- **C** (kernel PDs, device drivers, core services) — freestanding, no libc
- **Rust** (userspace servers, SDK, higher-level services) — `no_std` where applicable

**No interpreted languages in core.** Python and Node.js are permitted ONLY in:
- `tools/` — external build/debug tooling (clearly marked as host-side)
- `sdk/python/` — Python SDK for external consumers
- CI scripts

They must **never** appear in `kernel/`, `services/`, `libs/`, or `userspace/servers/`.

### 3. seL4 IS THE ONLY RING 0 CODE

No PD may claim kernel privilege. No PD may bypass seL4 IPC. No PD may directly
manipulate hardware without holding the appropriate seL4 device frame capability
granted by the root task.

If you find yourself writing code that "needs kernel access" — you're doing it wrong.
Redesign as an IPC contract with the appropriate PD.

### 4. API CONTRACTS PRECEDE IMPLEMENTATION

Before writing **any** new PD code:

1. Define the IPC contract in a header: `include/contracts/<pd_name>_contract.h`
2. Add opcodes to `agentos_msg_tag_t` in `agentos.h`
3. Define request/reply structs in the contract header
4. Write a failing test in `tests/contracts/<pd_name>_test.c`
5. **Then** implement the PD

A PR that adds implementation without a contract header and test will be rejected.

### 5. EVERY PD IS AN API ENDPOINT

Every Protection Domain exposes exactly **one** IPC contract. That contract — defined
in its `_contract.h` header — is the **only** way to interact with the PD.

A contract header defines:
- Channel IDs (cross-referenced to `agentos.h` constants)
- Message opcodes (cross-referenced to `agentos_msg_tag_t`)
- Request structs: `struct <pd>_req_<opcode>`
- Reply structs: `struct <pd>_reply_<opcode>`
- Error codes: `enum <pd>_error` (0 is always success)
- Invariants: preconditions, postconditions, ordering constraints

### 6. GENERIC BEFORE SPECIFIC

agentOS provides **OS-neutral generic device PDs** for all standard device classes:
- `serial_pd` — serial I/O
- `net_pd` — networking
- `block_pd` — block storage
- `usb_pd` — USB devices
- `timer_pd` — timers/RTC
- `irq_pd` — interrupt routing

**No guest OS may implement its own driver for any device class that has a generic PD.**

If a guest OS needs device functionality the generic PD cannot provide:
1. File a GitHub issue: `[device-waiver] <guest_os> requires <device_class>`
2. Document: what the generic PD can't do, what extension would fix it, why the
   extension can't be made
3. Get approval from the project owner **before writing any code**

PRs with guest-specific drivers and no approved waiver issue: **rejected**.

### 7. TESTS ARE THE SOURCE OF TRUTH

The test suite in `tests/` is the authoritative specification of API behavior.
**A contract not tested is not a contract.**

- Every IPC opcode must have at least one test
- Every PR that adds/modifies an opcode must add/update tests
- PRs that reduce passing test count: **rejected**
- `make test` must pass on both `TARGET_ARCH=aarch64` and `TARGET_ARCH=x86_64`

---

## Repository Structure

```
agentOS/
├── kernel/agentos-root-task/
│   ├── include/
│   │   ├── agentos.h                    # Master header: opcodes, types, constants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhubbard/agentos](https://github.com/jordanhubbard/agentos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
