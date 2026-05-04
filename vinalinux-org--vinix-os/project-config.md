---
trigger: always_on
description: **Plan chi tiết**: [/home/haidoan2098/.claude/plans/t-i-mu-n-b-n-trace-velvet-mitten.md](/home/haidoan2098/.claude/plans/t-i-mu-n-b-n-trace-velvet-mitten.md)
---

# CLAUDE.md — VinixOS Project Guide

**Plan chi tiết**: [/home/haidoan2098/.claude/plans/t-i-mu-n-b-n-trace-velvet-mitten.md](/home/haidoan2098/.claude/plans/t-i-mu-n-b-n-trace-velvet-mitten.md)

Roadmap P0-P8 + U1 + U2, timeline, architecture decisions, và scope cuts đều nằm trong plan đó. CLAUDđE.md này là rules + conventions bắt buộc mọi session.

---

## 1. Hard Constraints

BẮT BUỘC tuân thủ mọi lúc. Không có ngoại lệ.

**Memory — KHÔNG được:**

- Dùng `malloc`/`free` — chỉ static allocation (kernel) hoặc `kmalloc/kfree` nội bộ (sau P1)
- Dùng buffer lớn trên stack (kernel stack nhỏ)

**Standard Library — KHÔNG được:**

- `#include <stdio.h>`, `<stdlib.h>` trong kernel code
- Dùng `printf()` — phải dùng `uart_printf()` (sau P8: `pr_info/pr_err/pr_warn/pr_debug`)
- Dùng bất cứ libc function nào — chỉ dùng `libc/src/string.c` nội bộ

**Hardware Access — BẮT BUỘC:**

- Truy cập register chỉ qua `mmio_read32(addr)` / `mmio_write32(addr, val)`
- Lấy register address và bit definition từ AM335x TRM — KHÔNG được đoán
- **KHÔNG hardcode peripheral address** trong `kernel/` — dùng `platform/bbb/` constants

**Toolchain — KHÔNG được:**

- Mix `arm-none-eabi` và `arm-linux-gnueabihf`
- Build kernel trước userspace (thứ tự: `userspace` → `kernel`, luôn luôn)

**VinCC Subset C — CHỈ áp dụng cho user program biên dịch bằng VinCC. KHÔNG áp dụng cho kernel:**

- `struct`, `union`, `enum`, `float`, `double`
- `++` / `--` — dùng `i = i + 1`
- Variadic function, standard library
- Hơn 4 tham số hàm (r0–r3 theo AAPCS)

**Design — KHÔNG được:**

- Dùng Linux BSP hoặc SDK thương mại
- Dùng emulator — chỉ chạy trên hardware thật
- Thêm abstraction layer không cần thiết

**Scope hardcoded (không mở rộng khi chưa có consumer):**

- MAX_TASKS = 5
- Signal set: `SIGKILL`, `SIGSEGV` (KHÔNG `SIGTERM`, `SIGCHLD`)
- `wait()` chỉ wait-any (KHÔNG `waitpid(pid)`)
- Concurrency primitive: spinlock + atomic + wait_queue (KHÔNG mutex, KHÔNG semaphore)
- KHÔNG pipe syscall ở v1 (defer v1.1)
- KHÔNG TCP (UDP/ICMP/ARP only nếu P7)
- KHÔNG editor (device dùng `echo >` redirect)

---

## 2. Rule 7 — 100% Tự Viết (TUYỆT ĐỐI)

**Mọi dòng code trong repo phải được tay gõ, hiểu, chịu trách nhiệm.**

### KHÔNG ĐƯỢC copy/fork/port từ

- libc: musl, glibc, newlib
- Userspace: BusyBox, toybox
- Network: lwIP, uIP, picoTCP
- Filesystem: FatFs, SDFat, Linux ext4 code
- Compiler: TCC, GCC, chibicc, c4, PCC
- Kernel: Linux (any tree), FreeBSD, NetBSD, Minix, xv6, seL4, Zephyr
- Driver: bất kỳ upstream driver code nào

### Reference code ở `docs_trainingAI/drivers/*/source/`

- CHỈ được **đọc để hiểu logic và sequence**
- PHẢI **viết lại từ đầu** theo naming + convention VinixOS
- KHÔNG **copy nguyên block code** — mỗi dòng phải hiểu và adapt
- Pattern thuật toán (ví dụ: free-list malloc K&R, recursive descent parser) OK để học, nhưng **code phải tay gõ**

### Khi nghi ngờ "có đang copy không"

1. Đóng reference source
2. Code lại từ memory + hiểu biết
3. Nếu phải mở reference lần 2-3 cho cùng function → rewrite hoàn toàn sau khi đóng

### Checklist trước commit cuối mỗi phase

- [ ] Không có file copy từ upstream
- [ ] Git log message: "Implement X from scratch", không "Port X from Y"
- [ ] Nếu học pattern từ Linux/BSD: ghi `/* Pattern reference: Linux <file>:<func> — re-implemented */` ở function header, không dùng code
- [ ] Mọi comment là tiếng nói của bạn, không paraphrase

**Narrative with investor:** *"100% Made in Vietnam by hand. Không asterisk."*

---

## 3. Project Context

**VinixOS** — bare-metal ARM platform tự xây từ đầu, chạy trực tiếp trên BeagleBone Black (SoC AM3358, Cortex-A8, ARMv7-A). Không Linux, không emulator, không SDK thương mại.

### 4-layer HAL architecture

```text
kernel/           — generic C: mm, sched, vfs, proc, ipc (KHÔNG đụng khi port)
arch/arm/         — ARMv7 CPU: MMU asm, context switch, exception vector, cache ops
platform/bbb/     — AM3358 SoC + BBB board: memory map, clocks, IRQ numbers, device table
drivers/          — driver impls (omap_uart, omap_hsmmc, cpsw, lcdc, ...)
```

**Port sang SoC khác** = viết `platform/<new>/` + driver mới. `kernel/` không đổi dòng nào.

### Component

- **VinixOS** — kernel + bootloader + userspace system tools (init/sh/ls/cat/ps/...), C + ARM assembly
- **vinixlibc** — POSIX subset libc tự viết ~5K LOC (sau P6)
- **VinCC** — Python cross-compiler **trên host**: Subset C → ARMv7 ELF32 cho end-user C program chạy trên VinixOS (laptop compile → copy ELF vào SD → BBB execute)

### VinCC scope — chỉ cross-compile user program

VinCC CHỈ dùng để compile chương trình C do end-user viết (hello.c, custom.c), link với vinixlibc, chạy ở tầng userspace VinixOS trên BBB.

VinCC **KHÔNG** compile:

- Kernel / bootloader
- vinixlibc
- VinixOS system tools (init, sh, ls, cat, ps, ...)
- Bất cứ thứ gì thuộc repo VinixOS — những cái đó dùng `arm-none-eabi-gcc` full C

### Toolchain

- VinixOS code (kernel + bootloader + vinixlibc + system tools) → `arm-none-eabi-gcc` (bare-metal, full C)
- End-user C programs (Subset C) → VinCC Python trên host → ELF output copy vào SD

### Thứ tự build

```bash
make -C VinixOS userspace   # TRƯỚC — kernel nhúng shell payload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vinalinux-Org/Vinix-OS](https://github.com/Vinalinux-Org/Vinix-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
