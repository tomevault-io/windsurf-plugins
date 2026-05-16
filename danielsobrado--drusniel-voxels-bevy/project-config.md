---
trigger: always_on
description: This repo has an active rendering/performance workflow. Treat profiling as part of the implementation, not as optional cleanup after the fact.
---

# Agent Instructions

This repo has an active rendering/performance workflow. Treat profiling as part of the implementation, not as optional cleanup after the fact.

## Profiling Rules

1. Any change that can affect rendering, terrain meshing, props, shadows, water, post-processing, or frame timing should be measured.
2. Use release benches for performance claims:

```powershell
cargo run --release -- --bench bench/scenes/visual-regression.toml
```

3. Benchmark variants already exist for common A/B work:
   - `bench/scenes/visual-regression-high.toml`
   - `bench/scenes/visual-regression-performance100.toml`
   - `bench/scenes/visual-regression-live-lod.toml`
4. Read `bench-runs/<run>/summary.json` and compare before/after numbers. Do not add broad timing rows together because some are parent/child or overlapping brackets.
5. For render investigations, use the built-in counters and timing rows first. If a change claims an improvement, report which rows changed and by how much.
6. Run the regression guard when the change touches known bottlenecks:

```powershell
cargo run --bin bench_guard -- bench-runs/<run>/summary.json
```

7. If the change is visual, inspect the fixed checkpoint screenshots from the bench output. Do not accept a performance win that introduces visible regressions unless the tradeoff is intentional and documented.
8. Outside bench mode, `VOXEL_RENDER_TIMING=1` enables the same render timing capture in the debug timing CSV for local diagnosis.
9. For NAADF preview or GI changes, also measure initial visual stability. Run:

```powershell
rtk cargo run --release --features naadf -- --bench bench/scenes/visual-regression-naadf-startup-stability.toml
```

Inspect the staged screenshots (`settle-120`, `settle-240`, `settle-360`, `settle-540`, `settle-720`, `settle-899`, `settle-1200`, `settle-1499`) and report the first `frame`/`elapsed_secs` in `summary.json` where the image is fully textured rather than the blue silhouette/early occupancy preview. Treat `ready_wait_secs` and `render_ready_secs` as runtime readiness only; they do not prove visual texture stability. Include those timings, NAADF preview counters, and any startup trace phase CSVs from the run.

## Expected Workflow

1. Run a baseline bench for the relevant scene.
2. Make the change.
3. Re-run the same bench scene.
4. Compare `summary.json`, screenshots, and any relevant counters.
5. State the measured result plainly, including regressions if they exist.

If you did not profile a performance-sensitive change, say that explicitly instead of implying the result is verified.

## Gameplay, Spawn, And Collider Benches

Use the collider walk bench when changing spawn placement, player movement, terrain colliders, terrain readiness, or fall-through guards:

```powershell
rtk cargo run --release -- --bench bench/scenes/collider-walk-log.toml
```

This bench drives spawn-adjacent routes, a historical fall-through route, and a dig-crust checkpoint that digs beneath the player and verifies the hard crust rejects below-floor/bedrock edits. It logs player coordinates, validity, collider readiness, stall events, fall-through events, and dig/crust rejection counters, and uses simple path steering to turn at borders and avoid missing ground or steep/blocking terrain.

Runtime, editor viewport, and bench launches share the same runtime lock by default so only one Drusniel runtime runs at a time. Use `DRUSNIEL_BENCH_RUNTIME_LOCK` only when an isolated bench lock is intentional.

## Editor Runtime Verification

After any task that changes the editor, the Bevy runtime, Tauri integration, viewport behavior, or editor-visible UI, rebuild the editor runtime sidecar and restart the desktop editor so the change can be tested in the running app.

Use the existing editor scripts from `editor/frontend`; do not assume Rust code changes are visible in the editor until the sidecar has been rebuilt and the editor has been restarted.

## Rule

Always prefix shell commands with `rtk`.

Examples:

```bash
rtk git status
rtk cargo test
rtk npm run build
rtk pytest -q
```

## Meta Commands

```bash
rtk gain            # Token savings analytics
rtk gain --history  # Recent command savings history
rtk proxy <cmd>     # Run raw command without filtering
```

## Verification

```bash
rtk --version
rtk gain
which rtk
```

## Editor Diagnostics

Use `DRUSNIEL_EDITOR_DIAGNOSTICS=1` when starting the editor/runtime to enable heavy, structured diagnostics for viewport input, targeting, selection, native viewport attach/focus, and bridge behavior. Keep it disabled by default; enable it only for focused debugging sessions and include the relevant log excerpts when reporting findings.

When diagnostics are enabled, the Tauri shell also exposes a local-only screen simulation endpoint at `http://127.0.0.1:17778` for automated verification:

```bash
curl http://127.0.0.1:17778/health
curl http://127.0.0.1:17778/focus
curl "http://127.0.0.1:17778/screenshot?label=viewport-check"
curl "http://127.0.0.1:17778/move?space=viewport&x=100&y=100"
curl "http://127.0.0.1:17778/click?space=viewport&x=100&y=100&button=left"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielsobrado/drusniel-voxels-bevy](https://github.com/danielsobrado/drusniel-voxels-bevy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
