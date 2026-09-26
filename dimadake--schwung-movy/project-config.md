---
trigger: always_on
description: Movy is a Schwung **tool module** for Ableton Move. The UI (TypeScript →
---

# CONVENTIONS.md — Movy

Movy is a Schwung **tool module** for Ableton Move. The UI (TypeScript →
`ui.js`) runs in the shadow-UI QuickJS context; it presents the active chain
slot's synth parameters on the 8 knobs and is also a **native-Move-style
4-track step sequencer**. The sequencer's musical engine is a **Rust DSP**
(`dsp.so`) that schwung loads as the co-running overtake DSP.

Device: `ableton@move.local`

**Plans:** Save all implementation plans to `movy/plans/` (not the repo root `plans/`).

## Aider setup

This repository is part of a parent `cld` workspace containing several related
repositories. Aider agents should treat sibling repos as **context only**, not
as files to edit unless the user explicitly asks and has added them to the chat.

- `../schwung` — the main Schwung runtime/shadow-UI host. This is the most
  important sibling: many host API facts in this file are sourced from
  `schwung/src/shadow/shadow_ui.js`.
- Other module repos may sit alongside `movy` under the same parent directory.
  Until they are listed in `.aider.conf.yml` (or added to the chat manually),
  aider does not see them.

If you need aider to include those sibling repositories in its repo-map, add
them to `.aider.conf.yml` under `read:`. Do not add broad `read:` globs that
pull in build artifacts or the sibling repos' own `.aider` files.

---

## Context discipline

A tool call is a full model turn — the whole conversation gets re-sent on
every one — so round-trip *count* is the cost, not output size. Optimize for
fewer calls, not smaller ones.

- **Batch independent shell calls into one message.** Firing them one at a
  time pays a full round trip each even when none depends on another's
  result.
- **Device work: one `ssh` round trip, not three.** The clear-log → act →
  check-log cycle run as three separate `ssh ableton@move.local` calls is the
  single most common device pattern in this repo's session history. Use
  `scripts/dev-probe.sh log` instead — it clears the log, optionally injects a
  MIDI event, polls for the pattern, and dumps matching lines inside one ssh
  call. `scripts/dev-probe.sh status` does the same for reachability + deployed
  `ui.js` md5 + log-enabled state.
- **Grep or read a line range before reading a whole file.** `Read` on an
  entire file is the most expensive call type per-invocation in this repo. If
  you're hunting one symbol, `grep -n` it first and read just that range.

---

## Sequencer (engine + UI)

The sequencer spans two layers; keep musical truth in the engine and only a
mirror in the UI.

- **Engine — `engine/` (Rust workspace):** `seq-core` (pure logic: clock,
  clips/notes, scheduler, recording, sessions, persistence — host-testable
  with `cargo test`) + `movy-dsp` (`cdylib` → `dsp.so`, implements schwung's
  `plugin_api_v2`; every FFI entry point catches panics so an engine bug can
  never abort MoveOriginal). Spec/design: `plans/2026-06-12-sequencer-*.md`.
- **UI — `src/seq/`:** `engine.ts` (the only IPC: one batched `cmd`
  set_param/tick + a `status` poll), `state.ts` (mirror), `router.ts`
  (first-look MIDI dispatch — sequencer events never touch the param-page
  handlers) with `router-steps.ts` / `router-pads.ts` / `router-buttons.ts`
  holding the three halves it dispatches to (the step row, pads + held chord,
  and the modal/edit buttons; transport, encoders and arrows stay in
  `router.ts`, which also re-exports the others' public surface so callers keep
  one import site), `leds.ts`/`session.ts`/`render.ts` (cached LEDs, clip grid,
  Loop Overview strip), plus `loop-mode.ts`, `step-edit.ts`, `edit-ops.ts`,
  `pads.ts`, `persist.ts`, `colors.ts`, `constants.ts`.

### Hard rules (learned on device — do not relearn)

- **ENGINE_VERSION must match** between `engine/crates/movy-dsp/src/lib.rs`
  and `src/seq/constants.ts` (`build-dsp.sh` fails the build otherwise). The
  UI probes `ping` and re-issues the DSP load until the version matches.
- **A redeployed `dsp.so` does NOT hot-reload — the stack must restart.** The
  shim dlopens the engine by path, and glibc returns the library already loaded
  under that path for as long as MoveOriginal lives, so the version gate above
  just loops: it re-issues the load and the shim answers with the old binary.
  `deploy.sh` therefore restarts the stack whenever the shipped `dsp.so`
  differs (`--no-restart` opts out, and says loudly that the old engine is
  still running). The restart must run **as root** — MoveOriginal is root's, so
  `restart-move.sh` as the `ableton` user pkills nothing and still exits 0.
  Bumping ENGINE_VERSION once for two different builds hides this completely:
  both answer `ping` with the same string, and the stale one looks current.
- **Engine sets must be blocking** (`host_module_set_param_blocking`): the
  `overtake_dsp:` param SHM is a single slot, so non-blocking writes (and even
  schwung's own DSP-load request) are routinely lost.
- **Never scp over a dlopen'd `dsp.so` in place** — overwriting a mapped
  `.so`'s inode corrupts its pages and crashes MoveOriginal. `deploy.sh`
  ships it scp-to-temp + `mv` (fresh inode).
- Live pad notes are sounded **directly** (`shadow_send_midi_to_dsp`,
  channel = track) for zero latency; the engine only **records** them (no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DimaDake/schwung-movy](https://github.com/DimaDake/schwung-movy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
