---
trigger: always_on
description: These instructions apply to the whole repository. The primary codebase is the
---

# Repository Instructions

## Scope

These instructions apply to the whole repository. The primary codebase is the
Rust Cargo workspace in `crates/`; `mavsdk-demo/` contains Python helper scripts
for external MAVSDK experiments and should not drive Rust architecture choices.

## Project Shape

- `crates/mavsim-core`: simulator state, physics, world/environment, vehicle
  models, sensors, gimbal, geospatial projection, and public core types.
- `crates/mavsim-mavlink`: MAVLink/PX4 HIL bridge, endpoint routing, runtime
  loop, serial/UDP/TCP transport handling, and dialect bindings generated at
  build time from `mavlink/message_definitions/*.xml`.
- `crates/mavsim-ui`: `eframe`/`egui` desktop UI, keyboard commands, HUD/report
  panels, asset loading, software viewport fallback, and `wgpu` viewport
  callback rendering.
- `crates/mavsim-bin`: `mavsim` CLI compatibility parsing, startup/shutdown
  wiring, logging, and headless vs GUI mode selection.
- Runtime assets live in `models/`, `environment/`, and
  `mavlink/message_definitions/`. Keep paths relative to the repository root
  unless an existing API clearly expects otherwise.

## Rust Quality Bar

- Treat each Rust change as part of a long-lived simulator. Prefer simple,
  readable, maintainable code over clever compactness, and do not stop at "it
  compiles" when behavior, tests, or error handling are still unclear.
- Before changing code, identify the source of truth, existing local patterns,
  failure modes, and invariants that Rust's type system can enforce. When
  requirements are ambiguous, make the smallest safe assumption and document it
  only where future maintainers need the context.
- Prefer clear, typed domain APIs over ad hoc strings or loosely shaped data.
  Public options should flow through `SimConfig` and the typed enums in
  `mavsim-core` where possible.
- Make invalid states unrepresentable where practical: use enums for finite
  states, newtypes for domain-specific values, private fields with validating
  constructors, `NonZero*` types when useful, and `Result<T, E>` for validation
  that can fail.
- Keep simulation behavior deterministic when lockstep is enabled. Time changes
  should go through `Simulator::advance_time`, `tick_fixed`, or
  `tick_fixed_after_external_time_advance` as appropriate; avoid accidental
  double-advancement.
- Preserve PX4/SITL compatibility. Be careful with MAVLink message IDs,
  heartbeat/init semantics, `HIL_ACTUATOR_CONTROLS`, deprecated
  `HIL_CONTROLS`, `HIL_STATE_QUATERNION`, QGC/SDK forwarding, and filtering of
  HIL messages.
- Use `f64` for simulator math where the code already does. Convert deliberately
  at MAVLink, UI, or GPU boundaries and keep units obvious in names or nearby
  context.
- Handle fallible IO and transport setup with `anyhow::Context` in application
  and runtime code. Use typed errors such as `ConfigError` where callers or
  tests need exact error variants.
- Avoid `unwrap`/`expect` in production paths unless failure is truly impossible
  or unrecoverable during startup/build setup. Tests may use them for clarity.
- Do not ignore errors silently. If discarding an error or send failure is
  intentional, keep the scope narrow and leave a short reason nearby.
- Keep lock scopes small around `Arc<Mutex<Simulator>>`. Do not hold a simulator
  lock while performing blocking IO, sleeping, or calling into UI painting.
- Avoid hidden global mutable state, broad catch-all modules, speculative
  generics, excessive macros, and new traits that do not represent a real
  boundary or multiple plausible implementations.
- Reuse existing allocation patterns in hot paths: scratch buffers,
  `*_into(...)` functions, `Vec::with_capacity`, and `clear`/`reserve` are used
  intentionally in viewport and runtime loops.
- Do not edit generated MAVLink output under `target/`. Change XML definitions
  in `mavlink/message_definitions/` or `crates/mavsim-mavlink/build.rs` instead.
- Preserve the current public API smoke tests when changing exports from
  `lib.rs`.

## Python Quality Bar

- Python code currently lives under `mavsdk-demo/` and is a support/demo area
  for MAVSDK and fake PX4 workflows. Keep it isolated from Rust crate
  architecture and avoid moving simulator business logic into Python.
- Use the Python version pinned by `mavsdk-demo/.python-version` and manage the
  environment with `uv` using the existing `pyproject.toml` and `uv.lock`.
- Prefer clear, typed data containers (`dataclass`, small classes, constants)
  over dictionaries for state that has behavior or protocol meaning.
- Keep MAVLink/PX4 constants named explicitly and close to the code that uses
  them. Do not hide protocol message IDs, mode bits, units, or frame
  conventions behind ambiguous helper names.
- Keep scripts deterministic enough for tests: inject time, connection objects,
  or message streams where practical instead of sleeping or opening sockets in
  unit tests.
- Avoid broad exception swallowing. If a demo script handles an operational
  failure, log or print enough context for a user running the script to diagnose
  the endpoint, message type, or command involved.
- Keep top-level scripts runnable from `mavsdk-demo/` and avoid hard-coded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RustedBytes/mavsim](https://github.com/RustedBytes/mavsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
