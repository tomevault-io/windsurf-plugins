---
trigger: always_on
description: A playbook for AI agents collaborating on `stackchan-kai`. Read this once
---

# AGENTS.md

A playbook for AI agents collaborating on `stackchan-kai`. Read this once
per session before reaching for code. CLAUDE.md is the shared
human+agent guide; this file is agent-specific.

## At a glance

- **Stack:** `no_std` + `alloc` Rust on ESP32-S3 (CoreS3), embassy executor, defmt logging over USB-Serial-JTAG.
- **Domain model:** `Entity` + `Director` + `Modifier` / `Skill` traits in `stackchan-core` (pure, host-testable). Firmware composes a fixed modifier stack at 30 FPS in `render_task`.
- **Cross-task communication:** typed `Signal<RawMutex, T>` channels. Sensors publish via `signal()`; consumers drain via `try_take()`. Latest-wins semantics throughout.
- **Tests:** `stackchan-sim` runs modifiers against `FakeClock` for deterministic golden assertions. Firmware-side tests use on-device benches (`examples/*_bench.rs`).

## Session shapes

Most useful sessions fit one of these shapes. Recognising the shape early
keeps the work scoped.

### Shape 1: behavior change (modifier authoring)
1. Read the relevant modifier in `crates/stackchan-core/src/modifiers/`.
2. Add or change behavior; update the unit tests in the same file.
3. Add a sim-level integration test in `crates/stackchan-sim/src/lib.rs`.
4. `just check` — gates pass before flashing.
5. Optional: `just fmr` to confirm on hardware.

### Shape 1b: skill authoring
Skills are predicate-fired capabilities that write `mind.intent` /
`mind.attention` / `voice` / `events`; modifiers in later phases
translate that into face / motor.
1. Read `crates/stackchan-core/src/skill.rs` for the trait and pick
   any existing impl in `crates/stackchan-core/src/skills/` as a
   starting point.
2. Add the new file under `crates/stackchan-core/src/skills/`,
   re-export from `mod.rs`.
3. Register in `render_task` via `director.add_skill(&mut x)`.
4. If the skill's intent needs a visible response, add or extend a
   modifier (`Phase::Motion` for pose, `Phase::Expression` for
   face) that reads `mind.intent` / `mind.attention`.

### Shape 2: driver work (any of the chip-specific driver crates)
1. Read the crate's README to understand current scope.
2. Add registers, driver methods, or init steps; keep `embedded-hal-async` boundary clean.
3. Unit-test what you can on host (packet construction, register encoding).
4. Add or update a `crates/stackchan-firmware/examples/<chip>_bench.rs` that exercises the new path on hardware.
5. `just <chip>-bench` to verify on device.

For servo trim calibration specifically, `tools/bench-trim` parses the
`just bench` defmt output (`just bench | tee /tmp/scfmr.log` →
`bench-trim --input /tmp/scfmr.log`) and prints suggested
`head.pan_trim_deg` / `head.tilt_trim_deg` for `STACKCHAN.RON` instead of
grepping deltas by hand. For camera lens calibration, `tools/lens-calibration`
parses `just tracker-bench` output the same way (`just tracker-bench | tee
/tmp/scfmr.log` → `lens-calibration --input /tmp/scfmr.log`) and flags the
`tracker.flip_x` / `tracker.flip_y` mounting flags when the head chases away
from motion.

### Shape 3: firmware integration
1. Identify the `Signal<…, T>` channel(s) the new feature needs.
2. Producer goes in a per-peripheral task (`src/<chip>.rs`); consumer reads in `render_task`.
3. Update `Entity` (or a sub-component like `Face` / `Motor` / `Perception`) if the feature surfaces persistent state; remember to extend `Face::frame_eq` only if it's pixel-affecting.
4. `cargo +esp clippy --release -- -D warnings` from the firmware crate.
5. Hardware-verify boot and runtime via `just fmr`.

### Shape 4: docs / tooling
1. CLAUDE.md is shared (humans + agents). AGENTS.md is agent-only. `docs/` is for cross-cutting reference (e.g., `docs/errors.md`, `docs/http.md`).
2. Per-crate READMEs document API + gotchas — the pre-commit hook reminds you to review them when source changes.
3. justfile recipes are the project's idiomatic invocation surface — prefer adding a recipe to documenting a long invocation in prose.

### Shape 5: HTTP route or network feature
1. Wire format: parsers + validators in `crates/stackchan-net/src/{config,http_command,http_parse,bare_json}.rs`. Host-testable; unit tests live beside the parser.
2. Handler: `crates/stackchan-firmware/src/net/http.rs` matches requests by `(method, path)`; each route is a handler function.
3. Persisted state rides the RON schema (`stackchan_net::config::Config`) through `PUT /settings`'s atomic writeback — no parallel persistence paths.
4. Operator-driven routes show up in the dashboard: source lives under `web/src/` (Vite + Solid), and the firmware's HTTP responder embeds the built `web/dist/index.html.gz` via `include_bytes!` (see `crates/stackchan-firmware/src/net/respond.rs`). Run `just web-build` after touching `web/src/` — `just check-firmware` chains through it automatically.
5. `just check-firmware && just clippy-firmware && just build-firmware`, then curl smoke after flashing.
6. Document the route in `docs/http.md` (the canonical reference).

## Decision frameworks

- **Ask vs assume:** ask when the change spans multiple PRs, when a public API surface changes, or when a doc rewrite is implied. Otherwise assume and proceed; the user can course-correct.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andymai/stackchan-kai](https://github.com/andymai/stackchan-kai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
