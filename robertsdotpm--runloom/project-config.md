---
trigger: always_on
description: Full derivations for the invariants below: [docs/dev/RUNTIME_GOTCHAS.md](docs/dev/RUNTIME_GOTCHAS.md).
---

# runloom — project guidance

Full derivations for the invariants below: [docs/dev/RUNTIME_GOTCHAS.md](docs/dev/RUNTIME_GOTCHAS.md).

## Build & test
- Target **free-threaded CPython 3.14t** (M:N is only real with the GIL off):
  `~/.pyenv/versions/3.14.4t/bin/python3.14`, `PYTHON_GIL=0`. (Default as of 2026-07-07:
  3.14 carries the gh-116738 stdlib-C-module free-threading audit — e.g. heapq
  now holds the list critical section, fixing a 3.13t SIGSEGV on concurrent
  shared-heap access that is NOT a runloom bug. 3.13t builds remain available
  for p488 reproduction.)
- Build `python setup.py build_ext --inplace`; run with `PYTHONPATH=src`.
- Run the suite via `tests/run_isolated.py` (one file/subprocess — in-process
  `pytest tests/` flakes on cross-file state leaks).

## No hosted CI
- **Never add GitHub Actions / `.github/workflows/*.yml`** (leave
  `workflows-disabled/` disabled). Hosted CI isn't free; this has been asked for
  repeatedly.
- Gate locally: **`scripts/check_all_fast.sh` before any merge**;
  `scripts/check_all_extensive.sh` for a risky/large merge.

## Agent-shell gotchas
- Each shell caps `RLIMIT_NOFILE` at 4096 — raise it in the SAME block before a
  socket bench: `sudo -n prlimit --pid $$ --nofile=8388608:8388608` (doesn't
  persist). Kernel ceilings via `sudo -n sysctl -w …` (`vm.max_map_count`, ~2
  VMAs/goroutine, bites first at N≫100K).
- Shells run `set -e` — prefix multi-step blocks with `set +e` so a nonzero step
  (e.g. `pkill` with no match) doesn't abort the block.
- Deletions: `safe-rm`, never `rm`.

## Benching
- Measure the runtime, not setup: parallel `SO_REUSEPORT` acceptors, establish
  all N first, count round-trips over a fixed window. Race-free counters (one
  `bytearray(N)` slot per goroutine — a shared `+= 1` loses increments GIL-off).
- Loopback packets traverse the host nft ruleset (~14% throughput tax) — pass
  `--netns` to big_100 for a clean number.

## Scheduler invariants
- **A freed `runloom_g` struct never returns to the OS.** `slab_free` retains it
  (refcount 0, magic DEAD); a stale dup-wake reaches `hub_submit`, which reads
  `g->refcount` via `try_incref` — only sound while the struct is a valid g.
  Freeing → garbage refcount → SIGSEGV (arm64). Guard: `tools/verify/cbmc/sched_qref_cbmc.c`.
- **Signals deliver INTO the parked goroutine, not via the scheduler.** A handler
  raising during a cooperative wait propagates out of *that call*; the idle
  scheduler carries one out of `run()` only when nothing is parked. Path:
  `runloom_netpoll_signal_wake` + the `RUNLOOM_NETPOLL_SIGNALED` sentinel.
- **Future-completion wakes are call_soon-FIFO.** `wake_safe` keeps its
  same-thread fast-path (ready-ring push), detected by PEEKing `runloom_tls_sched`
  — never `runloom_sched_get()` (mallocs on a foreign waker). Guard:
  `tests/test_differential_asyncio.py` (sc_call_soon_fifo).
- **Preemption never yields mid object-destruction.** Both yield sites gate on
  `runloom_tstate_in_destruction` and defer (trigger stays armed); yielding inside
  a `tp_dealloc` freezes a half-dead object across a GC-safe point → UAF. Don't
  reroute via the eval-breaker.
- **Cooperative primitives are foreign-OS-thread-safe.** A non-goroutine thread
  (a patched `Lock` in an mp.Queue `_feed` thread) must detect no-goroutine (TLS
  peek NULL) and block on the real OS — never park a non-existent g, never lazily
  alloc sched state (`peek_current`, never `sched_get`).
- **Parked-fiber frames are made GC-visible by the frames anchor.** The
  free-threaded collector credits PEP-703 deferred stackrefs (code objects,
  functions, deferred locals) only on LIVE tstate `current_frame` chains; a parked
  fiber's frames live in `g->snap`, invisible — so with the specializing
  interpreter on (TLBC), their deferred-only referents were freed early → resume
  UAF (the p565/p524 crash). `module_gcframes.c.inc` registers ONE GC-tracked
  anchor whose `tp_traverse` (stop-the-world only) walks the fiber registry + the
  base-snap registry and visits every parked chain (greenlet-PR#511 visit set,
  transcribed in `runloom_iframe.c`). Consequences that are now memory-safety
  load-bearing: (1) the fiber registry (`runloom_greg`) must reach the anchor —
  `RUNLOOM_GREG_OFF` loses (the anchor refuses to activate blind), and ANY new
  spawn path that bypasses `runloom_greg_link` reopens the blind spot; (2) the
  single-thread drain's caller frames must stay registered via the base-snap
  registry (`runloom_base_snap_register`, one node per drain, paired at the single
  exit); (3) the snap seam ordering L1–L5 (frozen in `runloom_sched_pystate.c.inc`
  comments — `valid=1` last & safepoint-free on snap; `current_frame`/`c_stack_refs`
  restored before `valid=0` on load; `valid=0` last on snap_clear) must not be
  reordered; (4) the anchor is never immortal and its traverse never allocates;
  (5) `gc.freeze()` is neutralised by a gc-`start` callback that thaws the anchor.
  **TLBC stays ON iff `runloom_c.gc_frames_active`** — `runtime.py`'s
  `_tlbc_reexec_if_needed` re-execs with `PYTHON_TLBC=0` only when the anchor is
  inactive. **greenlet coexistence on 3.14t still needs `PYTHON_TLBC=0`** (its own

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertsdotpm/runloom](https://github.com/robertsdotpm/runloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
