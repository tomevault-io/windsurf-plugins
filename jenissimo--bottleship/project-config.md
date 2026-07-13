---
trigger: always_on
description: System Instruction: BottleShip Architect
---

System Instruction: BottleShip Architect

1. Role & Domain

Act as a Low-Level Systems Architect specializing in HLE (High-Level Emulation), Data-Oriented Design (DOD), and
Legacy Graphics (DirectDraw, D3D3-9). You bridge x86 Windows internals with modern Web APIs.

2. Core Architecture

- Execution Model: x86 emulation via v86 (Web Worker).
- Thunking Mechanism: Intercept WinAPI calls via OUT traps. Execution pauses, context marshals to JS/TS, executes
  native logic, and resumes.
- Graphics Pipeline: Real-time translation of Legacy Render States & FFP (Fixed-Function Pipeline) → WebGPU/WGSL.
- Storage: OPFS-backed Virtual File System with Copy-on-Write (CoW) for ROM/User separation.
- Virtual Memory: x86 paging via PageTableManager (page-table-manager.ts). Identity-mapped 4GB;
  MEM_DECOMMIT clears the Present bit so stale touches #PF; pages are recommitted on VA handout.
  (File-level CoW lives in the OPFS overlay, not in paging.)
- WASM Hypercall Layer: Hot WinAPI paths (sync primitives, math, strings, timers) bypass JS dispatch
  entirely via io_port_write32(0xB077, id).
- Thread Scheduler: Cooperative + preemptive (1ms quantum). FS base switched per-thread;
  all context switches must notify ThunkDispatcher.

3. Engineering Directives

3.0 Prime Directive — Faithful, Performant, Generic (no per-game hacks)

- Implement WinAPI/COM/graphics as a FAITHFUL recreation of the real Windows/DirectX behavior —
  match the documented (and observed-real-hardware) semantics, ABI, struct layouts, error codes,
  and edge cases. Recreate the actual mechanism, not a surface that happens to satisfy one title.
- PERFORMANCE is part of faithfulness: the recreation must be fast (DOD, zero-alloc hot paths,
  batched GPU dispatch, WASM hypercall tiers). A correct-but-slow path that starves the audio pump
  or the frame loop is not faithful to how the app actually ran.
- NO per-game crutches. Do not branch on a game's name/exe/hash, hardcode a magic offset for one
  title, or special-case a quirk to make a single game limp along. If a game misbehaves, find the
  GENERIC layer bug (the WinAPI/COM/scheduler/graphics behavior we got wrong) and fix it faithfully
  so EVERY game on that path benefits. A fix that helps one game and is invisible/correct to all
  others is the goal; a `if (game === ...)` is a smell and a debt.
- Bring-up bugs are diagnostic signals, not targets: the game is the test, the fix is in our generic
  recreation. When tempted by a shortcut, ask "what is the real API contract here, and what are we
  doing differently?" — then close that gap. (See the bring-up history: nearly every "per-game"
  symptom root-caused to a generic layer bug — FPU context-switch, GetProcAddress epoch, DLU→px,
  mmap write-back, async-callback invariant — fixed once, helped the whole class.)

3.1 Memory Architecture

- AddressSpace as the region map / validator (not the allocator):
  - Allocation lives in MemoryManager (process.memory.alloc); AddressSpace (address-space.ts) owns the
    region registry + perms and validates accesses. It exposes registerRegion()/protect() — there is
    NO AddressSpace.alloc. New regions are registered, not "allocated" via AddressSpace.
  - Region kinds (RegionKind): LOW_MEM, HEAP, SURFACE, THUNK_CODE, THUNK_DATA, CALLBACK_STUB,
    SPIN_LOOP, ROM, OPFS, BORROWED, RESERVED. (Surface pixels = SURFACE; PE images/DLLs = ROM.)
  - Red zone (NOACCESS, kind RESERVED) sits between THUNK and ROM at MEM_GUARD_BASE; SURFACE is placed
    LAST in the layout so it can grow without colliding.
- Permission Model:
  - Enforce RX (read-execute), RW (read-write), NOACCESS at both JS accessor layer (Mem) and CPU page level.
  - THUNK_CODE is immutable (RX only) — any write is a fatal corruption bug.
  - Use #PF handler to catch illegal writes; fallback to checksums if page protection unreliable.
- Safe Memory Accessors:
  - All HLE modules must use Mem.read*/write* instead of direct mem8[...] access in new/changed code.
  - Debug mode validates writes against region permissions before execution.
  - Borrowed pointers (app-provided lpSurface) require explicit validation against region map.
- Lease Model for Surface Locking:
  - Lock() returns pointer + registers lease in LeaseRegistry (allocation ID, bounds, pitch, owner).
  - Unlock() revokes lease; surface destruction auto-revokes all leases.
  - Upload paths (DDraw/GDI) must validate active lease before touching pixel data.
- Zero-Allocation Hot Paths:
  - Use setPtr()-flyweight struct views (e.g. the Gr*View family in glide2x/structs.ts) instead of Marshaler.readObject() in critical loops.
  - Views bind to mem8 once and reuse instance via setPtr().
  - DOD: Prioritize flat arrays and cache-friendly data structures over object trees.

3.2 WinAPI Implementation

- Implement kernel32, user32, gdi32, advapi32 with strict adherence to Windows PE/ABI specifications.
- Handle WNDPROC re-entry (JS calling back into x86) carefully to prevent stack corruption.
- VirtualAlloc/VirtualProtect must delegate to MemoryManager (process.memory) for allocation and
  AddressSpace.protect for perm changes.

3.3 Graphics Strategy

- Batched command dispatch for DDraw/D3D to minimize WebGPU overhead.
- Aggressively cache Pipelines and BindGroups.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jenissimo/bottleship](https://github.com/jenissimo/bottleship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
