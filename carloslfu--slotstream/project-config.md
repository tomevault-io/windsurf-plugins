---
trigger: always_on
description: Qwen3.8-Flash-Next (125B + 51B n-gram, 4-bit, ~104 GB) on Macs via SSD-streamed
---

# slotstream — agent instructions

Qwen3.8-Flash-Next (125B + 51B n-gram, 4-bit, ~104 GB) on Macs via SSD-streamed
experts and a slot cache. Read [PLAN.md](PLAN.md) for design, [MEASUREMENTS.md](MEASUREMENTS.md)
for every measured number and its method, `Tools/verify.sh` for the acceptance
battery. Work lands directly on `main`.

## Claims and measurement discipline (mistakes made 2026-08-29/30)

Every rule here is one this project already got wrong. They share a root:
**asserting from a document instead of checking the system.**

- **Before calling something blocked, check whether the repo already does it.**
  Four docs said closing the prefill gap "needs a Metal kernel, which needs
  Xcode", for two releases — while `GatedDelta.swift` shipped a JIT-compiled
  custom kernel as its fast path. The blocker was read off the risk register,
  which is about a *different* thing (mlx-swift's bundled shader library), and
  never tested. One `xcrun`, one four-line kernel, and it collapsed. A blocker
  that has never been reproduced is a rumour.
- **An estimator may not return a value outside the range it measured.** The
  decode curve extrapolated to 20 tok/s at 181/layer and over-promised 25 to
  45% through its own middle. Worse: immediately after fixing that, the prefill
  ladder was left returning 125 tok/s for chunk 4096, which nothing had
  measured. Cap at the last verified point, or measure a *ratio* at a config
  that fits and say so. Under-promising is the correct failure direction for a
  planner.
- **Re-anchoring a curve invalidates every number derived from it.** Fixing
  only the row in front of you produces contradictions readers will trust: the
  README's tier table ended up claiming a 32 GB Mac is faster than a 48 GB one.
  Regenerate the whole family — and prefer generating tables *from the tool*
  (`doctor --sim-ram N`) so they cannot drift from the code again.
- **A failing test is a bug in the test until proven otherwise.** Three "product
  failures" in one run were nested python-inside-shell quoting mangling the
  JSON; another two were a hardcoded version literal reporting a release bump as
  a regression. Reproduce by hand before believing a failure, and never pin a
  value a release will change.
- **`set -e` makes cleanup lethal.** `wait $PID` returns 143 after a `kill` and
  silently truncated the whole battery — it looked like a hang. Always
  `kill ... || true; wait ... || true`, or use a `trap ... EXIT` like
  `api_robustness.sh` does.
- **Benchmarks on a loaded machine are noise.** Single runs here vary 15%+, and
  one pair read 137.6 against 90.7 for the same config. Check reclaimable
  memory first, interleave A/B rounds, and discard anything measured while the
  machine is swapping. Report medians of paired rounds, never a best-of.
- **Explicit knobs bypass the safety clamp — that is their purpose, and it
  makes them dangerous.** `--experts-per-layer 181` is never resized by the
  availability clamp, and forcing it against 26.6 GB reclaimable drove the
  machine to 158 MB free and 13 GB of swap. `Planner.availabilityOverride` is
  worse: simulating 60 GB free made the governor allocate a *real* 25.4 GB pool
  and pushed swap to 39 GB. Bound both by `deviceAvailableGB()` before use.
- **Kill your own background waiters.** A poll loop watching a log that would
  never get its line sat in the task list for six hours looking like live work,
  and the "nothing is running" check missed it because it grepped for expected
  process names. Check the task list, not your assumptions about it.

## Memory safety — READ BEFORE RUNNING ANYTHING (incident 2026-08-28)

This Mac has **48 GB of unified memory shared with Carlos's live apps and
session**. On 2026-08-28 a session stacked test processes — a ~31.5 GB soak
server, a second test server, a browser pane, and builds — overcommitted the
machine and **crashed the whole system**. Every model process here is
multi-GB. These rules are mandatory:

1. **One model process at a time.** Never two servers; never `serve` plus a
   `run`/`elastic-check` concurrently. The binary now enforces this with a
   per-user file lock before model allocation. Still inspect `pgrep -fl
   slotstream` before heavy work; do not kill a process owned by another task
   without coordinating with its owner.
2. **Check reclaimable memory before every heavy step** (model launch, big
   build, verify run). Reclaimable = `vm_stat` free + purgeable + file-backed
   pages; `slotstream doctor` prints it as "reclaimable now". If what you are
   about to start does not fit with several GB to spare, do not start it.
3. **Tests use small explicit sizes** — `--memory-gb 8.1`..`10` — never auto,
   unless the large configuration is itself the measurement, and then nothing
   else heavy may be running.
4. **Kill every test process the moment its test ends**, and confirm.
5. `Tools/verify.sh` keeps every heavy gate between the 8.1 GB floor and a
   10 GB target. Equality tests use small pools because their property is
   size-independent; never restore a spare-RAM-driven large profile.
6. The engine caps MLX's allocator cache at 2 GB (`Engine.swift`,
   `MLX.Memory.cacheLimit`). Do not remove it: without the cap a 10 GB-target

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carloslfu/slotstream](https://github.com/carloslfu/slotstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
