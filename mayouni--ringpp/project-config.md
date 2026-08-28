---
trigger: always_on
description: **Ring++ is dependency-free.** It needs nothing but Ring itself:
---

# Ring++

**Ring++ is dependency-free.** It needs nothing but Ring itself:
no other package, no extension, no DLL, no sibling repository. It is
developed alongside Softanza and used by it, and is not part of it.

If you are an AI session working here, this file is the whole brief.

---

## The mission — restated by Mansour, 2026-08-23

Five commitments, in order. Review every change against them.

1. **More performant Ring, in Ring itself** — never by leaving the language.
2. **Build on Mahmoud's internal design** and take maximum advantage of it,
   keeping the tradeoff *equilibrated* between plain Ring and Ring++ code.
   Never fight the design, never break its culture.
3. **An educational framework with comparative testability** — same task,
   Ring and Ring++, side by side, byte-identical, measured — teaching the
   internal design of Ring *and its rationale*. For learners of low-level
   programming, the project is schoolcase material in Mahmoud Fayed's
   **patterns of thinking**, not in his implementation.
4. **Type safety for large Ring projects**, through the vendored
   tree-sitter checker and Ring's own `typehints` channel. This is the
   strategic centre: it is the practical answer to a real bank engineering
   team whose remaining concern about Ring at scale was exactly this.
   (The team is not named in public documents.)
5. **One shipped binary.** The CLI is a single prebuilt Zig binary that
   ships with the package. No C compiler, no clang, no toolchain is ever
   required or suggested to a user. The Zig *source* is provided; only an
   adapter of the CLI installs the Zig compiler.

Strategically: Ring++ makes Ring projects in business domains **more
governable** (static analysis) and **more efficient**, relying on nothing
but Ring. And because it builds on internals that may change, it maintains
an **abstract interface** — [docs/VM-CONTRACT.md](docs/VM-CONTRACT.md),
machine-checked by `rpp/probe.ring` on every load — kept small enough to
one day propose to Mahmoud as a contract both parties agree on.

**Descoped, 2026-08-23:** the compiled-kernel half (old T4–T7). The
headroom measurements stay as research (`bench/headroom/`,
`DESIGN_TOOLCHAIN.md`), but no compilation is promised, suggested, or
required. If it returns, it returns as its own proposal.

## The thesis, and why it is not "pointers are fast"

Ring++ does **not** exist because pointers beat Ring operations. On most work
they lose — measurably, by 3–28×, because every crossing from Ring into a C
function costs ~100 ns and the pointer route needs more crossings.

It exists because of one structural fact, measured in `docs/FINDINGS.md`:

> **A Ring string is copied every time it crosses a call boundary.
> A list is not.**

`RING_VM_STACK_PUSHCVAR` (`vm.h:230`) is a byte copy onto the VM stack.
Everything expensive about data-heavy work in Ring traces to that macro, and
everything Ring++ can honestly offer is a way of not paying it.

The target is banking, government and consumer platforms — high data volumes,
complex processing, optimisation, ML and AI. **Never gate the project on a
workload census**; those domains are the justification.

## No claim without a number

Every rule, every idiom and every example traces to a measurement in
`docs/FINDINGS.md`. **No rule ships without a number behind it.**

- **A/B on two builds or two code paths differing in exactly one thing**, on
  Ring 1.27 (`D:\ring127\bin\ring.exe`).
- **Report minima over repetitions**, never a single run. A single run once
  published `substr` at 53 µs and a 50,000× ratio; the true figures were
  12.5 µs and ~140×.
- Two significant figures. Say **"below the timer floor"**, never "0" —
  `clock()` has 1 ms resolution.
- **Always state the pattern the change HURTS.** A benchmark that shows only
  its good case is marketing. Three of the eight examples conclude that
  Ring++ is the wrong tool for the shape they demonstrate, and that is what
  makes the other five believable.

If a measurement disagrees with `FINDINGS.md`, **investigate before
publishing**. Example 08 disagreed by 13× and the gap turned out to be the
safety wrapper; example 07 disagreed by two orders of magnitude and the gap
was sub-state creation. Both became the useful half of their example.

## Examples are gates, not brochures

Each `examples/NN-*/example.ring` holds the raw-Ring path and the Ring++ path
side by side in one file, and:

1. **asserts byte-identical output BEFORE printing any speed number** — a
   false speed claim must not be able to reach the pass line;
2. prints the A/B, minima over repetitions;
3. states where Ring++ loses;
4. ends with `EXAMPLE nn OK`, which is what the runner looks for.

`examples/run-all.ps1` runs them **leashed** (see below) and is wired into
`tests/run-all.ps1`. Documentation that is not executed goes stale silently;
here a stale example fails the build.

## Never flood the machine

**Measured 2026-08-21 by freezing this machine three times.** A session
backgrounded `zig build` on a vendored engine — 186 steps compiling ggml,
wgpu, HarfBuzz, PCRE2, SQLite, libcurl, libuv and mbedtls — and ran repo-wide
scans while it built. Two hard freezes. Then it froze a **third** time on an
already-capped `zig build -j2` with nothing else running:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayouni/ringpp](https://github.com/mayouni/ringpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
