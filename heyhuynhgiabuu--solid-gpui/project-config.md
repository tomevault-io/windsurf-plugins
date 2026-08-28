---
trigger: always_on
description: **Read [README.md](./README.md) first.** This file tells you how to work here
---

# AGENTS.md — solid-gpui

**Read [README.md](./README.md) first.** This file tells you how to work here
without breaking things. Every "do not" below exists because a review or probe
caught the exact bug it prevents. Violating them is how this repo dies.

## What this project is

Solid 2 components rendered by **Zed's GPUI** through an **out-of-process**
Rust helper:

```
Solid reactivity → NDJSON mutation batches (stdio) → retained tree → GPUI frame
     (Bun/Node)         @solid-gpui/client           crates/helper      Metal
```

Out-of-process is a deliberate architecture decision (`.pi/artifacts/DECISIONS.md`,
ADR 002): the helper owns its main thread and native event loop, so there is no
Zed fork, no ThreadsafeFunction, and one code path per OS. Do not "simplify" it
into an in-process addon.

## Clean-room rules (legal, non-negotiable)

- [lxsmnsyc/solid-gpui] (MIT since ~2026-08) may be referenced for
  research and learning (user re-authorized 2026-08-25 after an earlier
  off-limits period). Default discipline: read docs/README/workflows for
  architecture and roadmap insight; porting actual source code requires
  attribution headers + THIRD_PARTY_NOTICES.md entry like any MIT source.
  Never present their code as ours; cite when a design is taken from there.
- Rich text/markdown/diff rendering is ported from [Comet] (MIT) with
  attribution headers — that is the legal source for those subsystems.

[lxsmnsyc/solid-gpui]: https://github.com/lxsmnsyc/solid-gpui

[Comet]: https://github.com/zeronsh/comet

## Commands

```sh
bun install                                # after changing workspace deps
bun run test                               # ALL bun suites (--conditions=browser, see trap below)
bun --conditions=browser test packages/solid   # one package
bun run typecheck                          # tsc ×3 (protocol, client, solid)
cargo test -p solid-gpui-protocol -p solid-gpui-helper
cargo clippy --all-targets && cargo fmt --all -- --check
bun run smoke:node                         # client under real Node 24
bun run demo:solid                         # counter window (~4 s, GUI)
```

GUI tests open real windows. Set `SOLID_GPUI_SKIP_GUI_TESTS=1` on headless CI.

## THE TRAP: solid-js resolves to non-reactive SSR stubs by default

`solid-js@2` maps the default `node` condition to `dist/server.js` — SSR stubs
with **no reactivity** (upstream [solidjs/solid#2569]). Effects run once at
mount, then silently never again. No error. No warning. Your UI just freezes.

Every entry point that touches Solid MUST pass `--conditions=browser`
(`bun --conditions=browser …`). It is already encoded in the root `test`
script, the `solid` package script, and `demo:solid`. If you add a new script,
command, or CI job that imports solid-js, encode it there too — then prove
reactivity is live (a signal set must produce new mutations within `flush()`).

[solidjs/solid#2569]: https://github.com/solidjs/solid/issues/2569

## Architecture invariants (violating these broke the app before)

1. **Validation and rendering agree.** If the renderer would silently drop it
   (e.g. children of text nodes), validation must reject it — otherwise the
   ack's `applied` count lies. (`crates/protocol/src/retained.rs`)
2. **Parentless ≠ acyclic.** A parentless *ancestor* (the root!) appended into
   its own descendant creates a cycle. `attach` walks ancestors and enforces
   `MAX_DEPTH = 256`; render recursion depends on that bound. Do not remove
   either check. (`retained.rs`, Slice 4 review Major)
3. **Mirror attach semantics on both sides of the wire.** Universal's
   `reconcileArrays` moves call `insertNode` for nodes *already* in the parent;
   our shadow tree must retain-before-splice exactly like the Rust `attach`.
   Skipping the retain grew duplicate entries and emitted invalid duplicate
   `removeChild`s. (Slice 5 review Critical)
4. **A failed batch poisons the renderer.** Mutations are spliced before send;
   a rejection means shadow and wire MAY have diverged. Re-sending could
   double-apply leading mutations. Poison (reject future flushes) and remount.
   Do not add requeue logic without redesigning apply idempotency.
5. **The container is virtual.** `#root` never crosses the wire. Mounting emits
   `setRoot`; a second mount destroys the previous root before setting the new
   one (otherwise old subtrees leak forever).
6. **Unknown style keys are accepted; unknown event types are errors.** Style
   keys are forward-compatible; events require the helper to know them. Keep
   the two closed/open sets distinct — they are not symmetric by accident.
7. **ElementId 0 does not exist.** Ids are 1..=2^32-1 on both sides. Report
   wire numbers at full width: `got as u32` once truncated 4294967298 → 2.

## Cross-language contract discipline

The protocol lives twice (TS + Rust). Any change must:

1. Update both implementations and both error taxonomies in lockstep
   (`invalidJson / unsupportedVersion / unknownOp / unknownEventType /
   unknownElementType / invalidShape`; replies: `ack / error{decodeFailed,
   applyFailed}`).
2. Regenerate the emission snapshot:
   `cargo test -p solid-gpui-protocol regenerate_rust_emission -- --ignored`
   then commit `packages/protocol/fixtures/rust-emitted-batch-01.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heyhuynhgiabuu/solid-gpui](https://github.com/heyhuynhgiabuu/solid-gpui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
