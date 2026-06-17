---
trigger: always_on
description: Codex is a new programming language, self-sustaining compiler, tools, operating system, repository protocol, trust lattice, encoding, and more. We take the best of type theory, language design, aesthetics, security research, and actual practice. We leave everything else behind. If we didn't build it, we don't trust it. Codex is a new computational substrate intended to be impervious to all currently known attack vectors by-design.
---

# CLAUDE.md — Codex Project Instructions

## What This Is

Codex is a new programming language, self-sustaining compiler, tools, operating system, repository protocol, trust lattice, encoding, and more. We take the best of type theory, language design, aesthetics, security research, and actual practice. We leave everything else behind. If we didn't build it, we don't trust it. Codex is a new computational substrate intended to be impervious to all currently known attack vectors by-design.

The project was started 3/14/2026.

### The Founding Vision (docs/PM/Stories/Vision/NewRepository.txt)

The original prompt that started the project:

> the new repository. condense all the good ideas humans have had in
> github, sourceforge, etc into a new language. start from first
> principles, find the best implementation, the best abstraction. port it
> to a language that can be transpiled to any old human designed language,
> it abstracts them all into a single perfect language. it is the basis
> for all future code. it exists for human reading and machine. it should
> read like a book. fulfill liskov's hopes for cobol. then we delete
> github and sourceforge entirely fully replaced with a single, ideal
> solution. write the book.

From there the design grew into: a literate-programming language where
prose is load-bearing, a type system with dependent types / linear types /
effect types, a content-addressed repository protocol replacing Git
(facts, proposals, verdicts, trust lattice), a unified environment
(Reader, Writer, Verifier, Explorer, Executor, Narrator, Historian), and
transpilation targets from Rust to WASM to LLVM IR. The full founding
document is in the file above.

## Docs Index

On session start, read ALL live docs using parallel agents. This costs
~20K tokens (~2% of context) and eliminates an entire class of mistakes
where an agent doesn't know about a prior decision, a known condition,
or a design that's relevant to the current task. Skip every
`docs/Designs/*/Done/`, the `docs/Designs/History/` archive, and
`docs/PM/Done/` — those are historical and recoverable from
`p4 filelog` if needed.

### Mandatory Reading (read directly, not via agent)

- `docs/VisionAndVirtues.md` — founding vision, non-negotiables, engineering virtues
- `docs/DevelopersGuide.md` — language syntax, types, CPL, seed rebuild procedure
- `docs/DevelopersRulebook.md` — foreword quire catalog, library rules
- `docs/OperatorsManual.md` — build process, test harness, VM setup, debugging
- `docs/ArchitectsSketchbook.md` — memory layout, registers, allocators, phase maps

### Also Read (via parallel agents at session start)

- `docs/PM/CurrentPlan.md` — current plan
- `docs/PM/BACKLOG.md` — outstanding work items
- `docs/Agents/PerforceProcess.md` — shelve/revert/sync protocol
- `docs/Designs/*/Active/` — ALL active designs, one section per concern: `Compiler/`, `Language/`, `Memory/`, `OS/`, `Hardware/`, `Backends/`, `Build/`, `Test/`, `Tools/`, `Features/`, `Projects/`, and `Apps/<project>/` (e.g. `Apps/CodexMagic/`, `Apps/Explorer/`). Each section has its own `Active/` + `Done/`; historical piles live under `docs/Designs/History/`
- `docs/PM/Stories/Vision/` — founding prompts
- `docs/Test/` — known conditions, crash investigations
- `docs/Reference/` — UEFI specs, AMI Aptio, paper index
- `docs/ReadingNotes/` — observations from external projects (NVlabs/Sana, etc.)

## Current State

**The compiler is a hard fixed point of itself on bare metal.** Codex
compiles itself end-to-end on bare metal (codex-vm x86-64, no OS, no
libc), and the output of that self-compile compiled by itself is
byte-identical to itself. No C# anywhere in the chain.

The canonical artifact is `seed/Codex.cdx` — a ~2.1 MB
self-sustaining CDX binary, bootable via codex-vm (or QEMU multiboot).
The CDX is the root of trust.

`tools/codex-vm.exe` is a ~4500-line C program (WHP hypervisor) that
emulates: PCI bus, xHCI USB (mass storage + HID keyboard + UVC camera),
Intel HDA audio with host waveOut, Bochs VBE display, NE2K NIC with
NAT, IDE disk, HPET, IOAPIC, ACPI/SMBIOS tables, UEFI firmware
(LocateProtocol, Block I/O, memory map, auto-extract PE from GPT
images), VGA text, GOP framebuffer, PS/2, CMOS RTC, PC speaker.
Build with `tools/build-vm.ps1`.

### Bootstrap History — 2026-04-24: The cord is cut

All four bootstraps green for the first time, 41 days from project start:

| Bootstrap | Path | Result |
|---|---|---|
| BS1 | .NET → C# | Legacy — locked |
| BS1.1 | .NET → Codex | Legacy — locked |
| BS2 (pingpong) | bare-metal → CDX | CDX fixed point: stage 1 CDX = stage 2 CDX |
| BS3 | bare-metal → CDX | CDX fixed point (standalone, from pingpong output) |

BS1 and BS1.1 used the C# reference compiler to bootstrap
the selfhost. The reference compiler is **permanently retired** — do not
edit, invoke, or rebuild it. The whole `old/` tree remains in the depot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damiant3/NewRepository](https://github.com/damiant3/NewRepository) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
