---
trigger: always_on
description: This is the entry point to the guide set for using `code.hybscloud.com/uring` correctly. Use it when generating or reviewing caller-side code around the package boundary: runtimes, protocol stacks, services, and review work that needs a precise Linux `io_uring` boundary.
---

# code.hybscloud.com/uring Agent Guide

This is the entry point to the guide set for using `code.hybscloud.com/uring` correctly. Use it when generating or reviewing caller-side code around the package boundary: runtimes, protocol stacks, services, and review work that needs a precise Linux `io_uring` boundary.

Read [`uring/README.md`](README.md) first for the package overview and API examples. Then use this guide to decide which facts belong at the boundary, which policy belongs in the caller layer, and which lifecycle transitions must remain visible.

## Purpose

Treat `code.hybscloud.com/uring` as a narrow Linux `io_uring` boundary. It sets up rings, encodes SQEs, observes CQEs, carries `user_data` identity, exposes capability evidence, and records ownership transitions visible at that boundary.

Keep caller policy in the layer that invokes `code.hybscloud.com/uring`: retry/backoff, poll cadence, scheduling, parking, routing, route retirement, parser state, protocol branching, completion routing, cancellation and timeout policy, safety checks, and service lifecycle. Use this guide for caller-side design and review, not as a package-maintenance manual.

The `INDEX.md` files under [`agents/`](agents/) are readable navigation: use them to choose the guide files required by a task. [`agents/references.md`](agents/references.md) is a natural-language research guide, and the [`agents/workflow/`](agents/workflow/) files are readable procedure: the gates, obligations, checklists, and staged loop you apply to a task. The remaining topic files under `boundary/`, `formalization/`, `lift/`, and `runtime/` carry the formal notation for tasks that need it, including abstraction from caller-side Go into the guide notation and compilation from checked notation back into Go. Theory terms in those files are useful only when they classify a concrete guide obligation: effect and resumption, context and coeffect, session frontier, runner observation, denotation, compilation, or verification evidence. This file is the readable entrypoint.

```text
beyond code.hybscloud.com/uring = broader systems built on the layer above: protocol stacks, services, applications
above code.hybscloud.com/uring  = the layer that calls the package directly: runtimes, event loops, adapters
caller layer = above ∪ beyond   = policy + protocol + routing + poll cadence + scheduling + cancellation/timeout policy
code.hybscloud.com/uring        = ring setup + SQE encode + CQE observe + ownership/capability facts
below code.hybscloud.com/uring  = Linux io_uring ABI + syscall/device/memory-order constraints
```

The caller layer has two parts. The layer *above* `code.hybscloud.com/uring` is the code that calls the package directly: runtimes, event loops, and adapters that ask the package to encode, submit, wait, observe, decode, recycle, or release one boundary action at a time. The layer *beyond* `code.hybscloud.com/uring` is the broader set of systems built on that layer, such as protocol stacks, services, and applications that never touch a ring directly. This guide and every file under `agents/` serve the whole caller layer: the same boundary facts, formalization, abstraction from caller-side Go, and compilation back to Go apply both to the direct above layer and to the wider beyond layer. The topic files abbreviate that caller layer as `C = caller(module) = A ∪ D`, with `A = above(module)` and `D = beyond(module)`.

## Required Reading

For code generation, review, or repair work above or beyond `code.hybscloud.com/uring`, read the complete `code.hybscloud.com/iox` package first. The outcome rules in this guide rely on that package's model.

When a task uses a fact owned by another package, read that package before using the fact:

- `code.hybscloud.com/iox` for stream, datagram, outcome, byte-progress, and `Backoff` contracts.
- `code.hybscloud.com/zcall` for Linux entry results and errno facts below the boundary.
- `code.hybscloud.com/iofd` for descriptor ownership and close lifecycle.
- `code.hybscloud.com/sock` for socket descriptors and address facts.
- `code.hybscloud.com/iobuf` for buffer pools, alignment, buffer views, and recycle ownership.
- `code.hybscloud.com/spin` for spin wait, yield, and spin lock primitives.
- `code.hybscloud.com/lfq` for caller-owned bounded FIFO mailbox and queue handoff when work crosses goroutines before reaching the serialized ring owner.
- `code.hybscloud.com/framer` for frame-boundary and codec facts when framed byte streams are part of the task.
- `code.hybscloud.com/cove` for explicit context, requirements, and safety evidence.
- `code.hybscloud.com/kont` for suspension and one-shot resumption.
- `code.hybscloud.com/sess` for protocol frontiers, branching, endpoint transitions, and session close semantics.
- `code.hybscloud.com/takt` for runner movement, polling, resumption, completion memory, completion routing, and route-indexed stream carriers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hayabusa-cloud/uring](https://github.com/hayabusa-cloud/uring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
