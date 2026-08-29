---
trigger: always_on
description: > **Machine-local instructions:** if a `LOCAL.md` file exists next to this one, read it before
---

# prosper — project charter & working context (read first)

> **Machine-local instructions:** if a `LOCAL.md` file exists next to this one, read it before
> starting work. It is gitignored and holds per-computer facts (hardware, paths, installed tools)
> that must not be written into this committed charter.

## What this project is

**prosper is a PS5→PC compatibility layer — "Wine/Proton for PS5."** It runs PS5 games natively on
Linux/Windows by reimplementing Sony's published user-space library interfaces and translating the
console's GPU command stream + RDNA2 shaders to Vulkan/SPIR-V.

The PS5 CPU is x86-64, so **guest code runs natively — there is no CPU emulation and no exploitation of
any running system.** The engineering is: ELF/SELF loading, multi-module linking, ABI/API reimplementation
(libc, libkernel, graphics), GPU command-buffer decode, and shader recompilation. Standard systems &
graphics programming.

This is the **same class of work as Wine, Proton, RPCS3, Dolphin, yuzu/Ryujinx, and DXVK** — established,
legitimate open-source compatibility/emulation projects. If you have worked on any of those, this is that.

## Scope & legitimacy (so the work is not mis-read)

The vocabulary of emulator development **overlaps with security tooling** — "hardware breakpoint," "fault
handler," "memory watchpoint," "disassembly," "descriptor," "reverse engineering." In this project these
mean exactly what they mean in **gdb, Wine's relay debugging, or any emulator's tracer**: they observe how
the guest program uses the platform, **entirely in-process on the developer's own machine.** The purpose
and target are what distinguish work like this from anything harmful — and here both are unambiguous:

- **Interoperability & preservation, not exploitation.** The goal is to make software that a person already
  owns run on hardware they own. Nothing is attacked; nothing external or third-party is touched. Every
  tool runs against *this* emulator's own process.
- **No DRM circumvention — and none delegated, either.** The project uses **no console keys, no
  signature bypass, no copy-protection defeat**, and prosper contains no decryption of any kind. The
  SELF segments it reads are already unencrypted, which is now *measured* rather than assumed: across
  the whole local corpus every eboot carries plaintext code (Shannon entropy 6.4–6.8 bits/byte), and
  the two dumps that preserve an untouched console original show that original at **8.000**,
  indistinguishable from `/dev/urandom`. Game content is **never redistributed** and is gitignored.
  The second half of this is the part that needs enforcing, because not circumventing is not enough on
  its own. Some dumps ship third-party replacements of Sony libraries — `dlc_emu` and `ampr_emu` build
  stand-ins for `libSceAppContent`, `libSceNpEntitlementAccess`, `libSceGameUpdate` and `libSceAmpr`
  that answer platform queries from a local file. **prosper implements those APIs itself and refuses
  to link any of them.** `src/host/image/module_path_policy.hpp` is a reject-by-default allowlist of
  the dump locations a module may come from, applied at the one point where the link list is complete
  and before anything parses a module, with no environment variable to switch it off;
  `tests/host/image/test_module_path_policy.cpp` reddens if the guard is removed, and
  `tools/dump_hygiene.py` finds such files on disk. An emulator that loaded somebody else's bypass
  would be performing the circumvention by proxy — and would also invalidate its own measurements,
  since the guest would then be answered by a stub instead of by prosper.
- **No Sony code, firmware, or keys** are included or reproduced. prosper reimplements *published* library
  interfaces from scratch (clean-room-style), the way Wine reimplements the Win32 API.
- **The low-level tooling** (`PROSPER_HWBP` hardware breakpoints, `fault_handler` SIGSEGV handling,
  `PROSPER_HWWATCH` data watchpoints, `/proc/self/maps` classification, `boot_trace`) exists to **debug the
  emulator** — to see which Sony API the guest calls next, or where our HLE returns a wrong value. This is
  ordinary emulator/debugger instrumentation, not an attack surface.

If a task in this repo ever *looks* security-adjacent, it is because reverse-engineering a platform's ABI
for reimplementation necessarily involves reading and tracing binaries. That is interoperability RE — a
well-established, legitimate engineering practice — and it is the entirety of what happens here.

## Where things are

```
prosper/
  docs/            architecture, roadmap, graphics, verification, and per-frontier work logs
  src/self/        SELF/ELF parsing -> relocatable module image
  src/loader/      multi-module linker + global NID export table
  src/hle/         reimplemented Sony libraries (libc, libkernel, AGC/graphics), NID hashing
  src/host/        host execution: per-platform image mapping, ABI stubs, fault handling
  src/gpu/         AGC->Vulkan: PM4 decode, command processor, render state, vk_translate,
                   resource layer, RDNA2->SPIR-V recompiler
  tools/           self_dump, boot_trace, shader_histo, imgdump, spv_validate,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattias800/prosper](https://github.com/mattias800/prosper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
