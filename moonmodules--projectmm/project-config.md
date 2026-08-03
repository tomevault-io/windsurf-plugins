---
trigger: always_on
description: A high-performance multi-platform system that drives large LED installations and DMX lighting fixtures. ESP32 is the primary target. Also runs on Teensy, macOS, Windows, Linux, and Raspberry Pi. C++20. CMake.
---

# CLAUDE.md

## What This Is

A high-performance multi-platform system that drives large LED installations and DMX lighting fixtures. ESP32 is the primary target. Also runs on Teensy, macOS, Windows, Linux, and Raspberry Pi. C++20. CMake.

See `docs/architecture.md` for system design. This file contains only rules and constraints for working on the project.

## Principles

- **Common patterns first.** This repo is meant to be a recognisable example of good practice across code, docs, tests, and UI, not a Frankenstein of bespoke conventions only the authors understand. Hold every decision against it, especially in core architecture and documentation. Before introducing a pattern, name a widely-used project / framework / canonical resource that uses it; if you can't, treat it as bespoke and justify the divergence in a one-line comment at the introduction site. A new contributor with general C++/web experience should recognise the pattern within 30 seconds. Bespoke choices are allowed (header-only light modules, the MoonModule lifecycle, present-tense docs), but each carries its reason at the place it's introduced.
- **Industry standards, our own code.** Reach for the established, recognisable solution: the textbook *algorithm* (a DC-blocker high-pass, a Hann window, RMS, an integer-square-root) AND the textbook *name* for every variable, function, and UI control. That's *Common patterns first* applied to both domains, core and light: take the textbook approach over a clever or borrowed one. Study the prior art hard, whatever sharpens our thinking: repos, datasheets, vendor sites. Respect it, learn from it, credit it by name in the `history/` digests and per-module "Prior art" sections. Then write every line fresh against our own architecture: **carry the ideas forward, but write our own code rather than copying theirs or tracing their structure.** The method that guarantees it: spec from primary sources (ESP32 / peripheral / sensor datasheets, Espressif docs, reference standards), pin the behaviour as tests (unit + scenario), and let the worker-bee agents implement against the process ([CLAUDE.md](CLAUDE.md)) and architecture ([architecture.md](docs/architecture.md)). The result is independent by construction, not a renamed copy.
- **Minimalism means elegance, not fewest features.** Minimalism here is *consistency, reusability, no duplication, minimal memory, and the fastest hot path* — not "the smallest feature set" or "the fewest lines." A recognizable, proven construct (recursion, a textbook algorithm, a clean data structure) that ticks those boxes is *more* minimal than a hand-rolled special case, even when it's more code, because it's reusable and consistent rather than a one-off. So **reach for the established, beautiful solution**, and when you *know* a complex system will need a capability, build the cleanest *complete* version of it rather than a crippled subset that forces hacks elsewhere later (a JSON reader that can't read arrays is not "minimal" — it pushes ugliness outward). Guardrail: the construct must be *recognizable* (see *Common patterns first*), not bespoke cleverness — a contributor with general experience should recognise *what* it is within 30 seconds, even if the full *why* takes longer. Beauty and consistency win over raw line count.
- **Complexity lives in core; domain modules stay simple.** When something is genuinely hard — a recursive parser, a bounded arena, a scheduling or mapping algorithm — it belongs in the **core**, written once as the elegant standard construct (see *Minimalism means elegance*), so every **domain module** (a light effect/driver/layout/modifier, DevicesModule, …) gets to stay short and obvious by leaning on it. A domain module that is accreting parsing, buffer juggling, or clever bookkeeping is a smell: that complexity wants to move down into a core primitive the module then calls in a line or two. The test: a domain module should read like *what it does*, not *how the hard parts work* — the "how" is core's job. (Example: the device-list persistence — the recursive JSON reader is core; DevicesModule's restore is "parse, fill my array," a dozen plain lines.) This is the same coin as *Core grows slower than the domain*: core earns its size by absorbing complexity that would otherwise be duplicated, badly, across many simple modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonModules/projectMM](https://github.com/MoonModules/projectMM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
