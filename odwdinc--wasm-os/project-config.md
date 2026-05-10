---
trigger: always_on
description: > Bare-metal Rust kernel. WebAssembly as the system ABI. Sprints 1–4 + A–G complete.
---

# AGENTS.md — WASM-First OS

> Bare-metal Rust kernel. WebAssembly as the system ABI. Sprints 1–4 + A–G complete.

---

## Project Status

Sprints 1–4 (MVP) and A–E, G (runtime completeness, isolation, scheduling, persistent FS, networking, in-OS WAT assembler) are complete.
Sprint F (JIT compilation) is in progress — detailed plan in [`JIT_Agile_plan.md`](JIT_Agile_plan.md). Target: NES emulator from 1200ms/frame → 16ms/frame (~75x speedup).

---

## Actual Source Layout

```
/
├── Cargo.toml                   # Workspace root (kernel + runner)
├── rust-toolchain.toml          # Pinned nightly toolchain
├── README.md
├── AGENTS.md                    # This file
├── CONTRIBUTING.md
├── MVP_Agile_plan.md            # Sprints 1–4 (complete)
├── Post_MVP_Agile_plan.md       # Sprints A–G
├── JIT_Agile_plan.md            # Sprints 0–6
│
├── kernel/                      # The entire working system lives here
│   ├── build.rs                 # Passes kernel stack size to linker
│   └── src/
│       ├── main.rs              # Entry point, boot sequence, macro definitions
│       ├── vga.rs               # Framebuffer writer, 8×8 font, scrolling
│       ├── scheduler.rs         # Round-robin cooperative scheduler (run loop)
│       ├── drivers/
│       │   ├── keyboard.rs      # PS/2 scancode decoder, try_next_key / next_key
│       │   ├── serial.rs        # 16550 UART, COM1 115200 8N1
│       │   ├── pit.rs           # 8253 PIT + 8259 PIC; ~100 Hz tick counter
│       │   ├── virtio_blk.rs    # Virtio 1.0 block device, DMA + page-table walk
│       │   ├── virtio_net.rs    # Virtio legacy NIC; virtqueue TX/RX, raw Ethernet frames
│       │   └── netstack/        # TCP/IP network stack
│       │       ├── mod.rs       # NetStack: ARP cache, TCP/UDP sockets, DHCP, send_ip
│       │       ├── arp.rs       # ARP table, request/reply encoding
│       │       ├── ethernet.rs  # Ethernet II frame parser/builder
│       │       ├── ip.rs        # IPv4 packet parser/builder, ip_chksum
│       │       ├── tcp.rs       # TCP segment parser/builder, TcpSocket state machine
│       │       ├── udp.rs       # UDP datagram parser/builder, UdpSocket
│       │       └── dhcp.rs      # DHCP DISCOVER/OFFER/REQUEST/ACK client
│       ├── interrupts/
│       │   ├── mod.rs           # IDT init
│       │   ├── idt.rs           # IDT structure and loading
│       │   └── handlers.rs      # IRQ handlers (keyboard, PIT)
│       ├── memory/
│       │   ├── mod.rs           # virt_to_phys (page-table walk), init
│       │   └── allocator.rs     # Bump allocator (global heap)
│       ├── fs/
│       │   ├── mod.rs           # In-memory file table, disk/write pools
│       │   ├── block.rs         # BlockDevice trait + static Ramdisk
│       │   ├── fat.rs           # FAT12/16/32 via rust-fatfs, BlockIo adapter
│       │   └── wasmfs.rs        # Legacy reference (not used at boot)
│       ├── shell/
│       │   ├── mod.rs           # Tokenizer, history, CWD, run_command dispatcher
│       │   ├── input.rs         # Non-blocking poll_once + blocking read_line
│       │   └── commands/        # One file per shell command
│       │       ├── asm.rs       # Assemble tiny WAT → WASM in-kernel
│       │       ├── cat.rs       # Print file contents
│       │       ├── cd.rs        # Change CWD
│       │       ├── clear.rs     # Clear screen
│       │       ├── df.rs        # FAT volume stats
│       │       ├── echo.rs      # Print arguments
│       │       ├── edit.rs      # Line-append editor
│       │       ├── help.rs      # List commands
│       │       ├── history.rs   # Show command history
│       │       ├── info.rs      # Module section info / tick count
│       │       ├── ls.rs        # List directory
│       │       ├── mkdir.rs     # Create directory
│       │       ├── ps.rs        # List WASM instance pool
│       │       ├── rm.rs        # Remove file
│       │       ├── run.rs       # Execute WASM synchronously
│       │       ├── save.rs      # Flush in-memory table to FAT
│       │       ├── tasks.rs     # task-run / task-kill / tasks
│       │       └── write.rs     # Write hex bytes as a file
│       └── wasm/
│           ├── mod.rs           # Module re-exports
│           ├── loader.rs        # Zero-copy WASM binary parser → Module<'_>
│           ├── engine.rs        # Instance pool, host registry, spawn/task API
│           ├── interp.rs        # Stack-machine interpreter, all opcodes
│           └── task.rs          # TaskState, task_spawn/kill/step/for_each
│
├── runner/                      # Host-side tool: wraps kernel ELF → BIOS disk image
│   └── src/main.rs
│
├── userland/                    # WASM source modules (.wat / .wasm)
│   ├── README.md
│   ├── hello/hello.wat          # Prints "Hello from WASM!\n"
│   ├── greet/greet.wat          # Prints a greeting string
│   ├── fib/fib.wat              # Recursive Fibonacci
│   ├── primes/primes.wat        # Sieve of Eratosthenes
│   ├── counter/counter.wat      # Counting demo (cooperative yield)
│   ├── collatz/collatz.wat      # Collatz sequence
│   └── httpd/httpd.wat          # Minimal HTTP/1.0 server on :8080
│
├── wasm-test/                   # Integration tests for the WASM interpreter
│   ├── src/lib.rs
│   └── tests/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [odwdinc/wasm-os](https://github.com/odwdinc/wasm-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
