---
trigger: always_on
description: Movy is a Schwung **tool module** for Ableton Move. The UI (TypeScript →
---

# CLAUDE.md — Movy

Movy is a Schwung **tool module** for Ableton Move. The UI (TypeScript →
`ui.js`) runs in the shadow-UI QuickJS context; it presents the active chain
slot's synth parameters on the 8 knobs and is also a **native-Move-style
4-track step sequencer**. The sequencer's musical engine is a **Rust DSP**
(`dsp.so`) that schwung loads as the co-running overtake DSP.

Device: `ableton@move.local`

**Plans:** Save all implementation plans to `movy/plans/` (not the repo root `plans/`).

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
  UI probes `ping` and re-issues the DSP load until the version matches —
  this is how a redeployed engine hot-reloads.
- **Engine sets must be blocking** (`host_module_set_param_blocking`): the
  `overtake_dsp:` param SHM is a single slot, so non-blocking writes (and even
  schwung's own DSP-load request) are routinely lost.
- **Never scp over a dlopen'd `dsp.so` in place** — overwriting a mapped
  `.so`'s inode corrupts its pages and crashes MoveOriginal. `deploy.sh`
  ships it scp-to-temp + `mv` (fresh inode).
- Live pad notes are sounded **directly** (`shadow_send_midi_to_dsp`,
  channel = track) for zero latency; the engine only **records** them (no
  double trigger). Recorded notes are suppressed until the clip wraps.
- **Note-offs come from the ledger, never from current state.**
  `keyboard/held-notes.ts` records `padNote → { track, pitch }` at note-on;
  `noteOff`/`drumPadOff` take neither a track nor a `DrumConfig`. Deriving
  either at release time strands notes whenever the active track, module, or
  view changed mid-hold. All `0x8n` sends go through `release.ts:emitNoteOff`.
  `app/unload.ts` (`globalThis.onUnload`, called by the host on *every*
  teardown) releases the ledger plus the engine's open gates read from
  `seqState.activeNotes` — the DSP is unloaded right after, so nothing else
  can close them.
- The engine has no filesystem; the UI ferries persisted state via
  `host_read_file`/`host_write_file` (`src/seq/persist.ts`).

### PErformance

PErformance is very important, make sure you think about it for implementation and add new and run existing performance tests for the new features


### Cost efficient usage

if you are opus or fable 5 try to optimize token usage and make it cost efficient while make sure the code is reviewed by you. if there is an option to use subagent, use it only if it reduces limit usage

### Build / deploy / test the engine

```bash
cd engine && cargo test            # pure seq-core logic (host)
./scripts/build-dsp.sh             # cross-compile aarch64 → dist/dsp.so (glibc <= 2.35)
./scripts/deploy.sh                # builds ui.js + dsp.so, deploys both (atomic .so)
./scripts/test-seq.sh              # device e2e: transport, steps, record, session, persistence
```

If MoveOriginal dies, recover with the davebox restart sequence (root SSH;
the user must run it): stop `move-launcher`, pkill the schwung stack, start
`move-launcher`.

---

## Dev loop

Run tests in this order at the end of every task:

Run `npm run build:browser` first (refreshes `dist/esm`), then in order
(or just `npm test`, which builds + runs all six):

```bash
# 1. Local (always) — viewmodel/business logic assertions
node browser-test/logic.mjs

# 1a. Local (always) — replays all 76 dumped modules; asserts layout invariants
#     + a per-module snapshot (browser-test/dump-expect.json). After an
#     intentional layout change: node browser-test/dump-replay.mjs --update
node browser-test/dump-replay.mjs

# 1b. Local (always) — full init/tick/MIDI loop → setLED (drum grid, multi-step)
node browser-test/app-loop.mjs

# 2. Local (always) — framebuffer pixel-diff vs baselines (pure node, no browser)
node browser-test/screenshot.mjs

# 3. Local (always) — performance regression (fill_rect count, IPC call count, render time)
node browser-test/perf.mjs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DimaDake/schwung-movy](https://github.com/DimaDake/schwung-movy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
