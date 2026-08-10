---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

WARP is an embeddable MoE inference engine in C11 (no third-party runtime
deps) that keeps a model's dense trunk resident and streams routed experts
from disk, using the remaining RAM as a bounded expert cache. Its proof
point is Kimi K3 (2.78T params, 982 GiB container) on a 64 GB laptop.

Three things follow from that and shape everything else:

- **Disk I/O is the budget, not arithmetic.** ~53% of a K3 decode step is
  expert reads. Optimizations are judged on bytes read per token and on
  cache hit rate, not on FLOPs.
- **RAM is a hard ceiling, not a hint.** `waste_cfg.ram_budget_bytes`
  bounds *everything* the engine allocates. Exceeding it means the OS
  pages, and a paged "cache hit" is slower than the disk read it replaced.
- **Correctness is measured against an oracle**, not asserted. Every layer
  is diffed against a PyTorch reference (`tools/kimi_ref.py`,
  `tools/vision_ref.py`, `tools/k3parts_ref.py`, `tools/kda_ref.py`).

## Commands

```bash
make                     # libwaste.a, waste CLI, libwaste.$(SOEXT), libwastevq
make test                # build the test binaries (separate target from `make`)
make check               # tests/run.sh — the whole C-side suite (rebuilds first)
make serve-check         # the Python server suite (needs libwaste.so/.dylib)
make asan                # full rebuild under ASan/UBSan + the suite, then clean
make fuzz FUZZ_RUNS=200  # container parser fuzzer
make fuzz-asan           # what CI runs: fuzzer against an instrumented build
make WASTE_ENABLE_METAL=1   # accelerators are build-time options
make CC=x86_64-w64-mingw32-gcc-posix   # Windows cross-build (arch from -dumpmachine)
```

`make` and `make test` are separate on purpose — a stale test binary is one
of the two failure modes `tests/run.sh` was written to catch, so it always
rebuilds both before running anything.

### Running the suite, and single checks

```bash
tests/run.sh                          # $HOME/models/kimi-linear.waste by default
tests/run.sh /path/to/model.waste     # a real container unlocks the skipped checks
tests/run.sh /nonexistent             # forces the synthetic container (what CI does)
```

With no container it builds a few-MB synthetic one via
`tools/make_test_container.py` and reports SKIP — loudly — for anything
needing real weights. A fresh clone on macOS is 30 pass / 13 skip; with
both K3 and Kimi-Linear containers on disk it is 45 checks, 43 of which
pass. Linux skips more, having no `uv` in `Dockerfile.test`: 26 / 16.

The download-script checks start `tests/range_server.py` on an ephemeral
port and read the number back through `--port-file`. Keep it that way — a
hardcoded port fails on a machine already using it, and it fails as
"resume" rather than as "that port is taken".

Env it reads: `WASTE_REF_MODEL` (container — **point it at a default
`convert.py` conversion**, i.e. a 4-bit trunk *and* VQ3R experts; a
`--trunk8` container is a shape nobody ships, and running the suite on one
is how the Q4G load path stayed broken through green runs — the same
applies to `--index-bits 6`, which exercises a different kernel and a
different record fmt), `WASTE_REF_SRC` (source safetensors, for
the round-trip), `WASTE_ORACLE` (logits from `tools/kimi_ref.py` — **must be
the same token ids run.sh uses**, or a mismatched dump looks exactly like an
engine bug; setting it also turns off both generating an oracle from the
container and the provenance check on the shipped fixture, so it is the one
way to compare against weights that are not the ones under test), `K3_DIR`
(the K3 release directory, for the XTML differential).

Individual checkers, after `make test` (all binaries land at the repo root):

```bash
python3 tools/make_test_container.py /tmp/tiny.waste     # anything below needs a container
./test_forward /tmp/tiny.waste 3,7,11,5 out.bin 0        # forward pass; 0 = no generation steps
WASTE_CHUNK=1 ./test_forward ...                         # chunked prefill instead of sequential
./test_container /tmp/tiny.waste/experts-L0.bin 2        # ONE bank + expected record count
./test_image /tmp && ./test_state MODEL && ./test_tokenizer MODEL "text"
./test_k3parts out.bin && uv run --with torch python tools/k3parts_ref.py out.bin

python3 -m unittest discover -s tests/serve -t . -p "test_*.py"   # all serve tests
python3 -m unittest tests.serve.test_regions -t .                 # one module
K3_DIR=... python3 -m unittest tests.serve.test_xtml.TestAgainstUpstream -t .
```

Python reference checks run through `uv run --with torch --no-project` —
torch is never a repo dependency and never in the inference path.

### Useful runtime env

`WASTE_PROFILE=1` (phase timings), `WASTE_CACHE_MB=N` (expert cache size in
the test harness), `WASTE_BACKEND=cpu` (disable SIMD/accelerator dispatch,
for bisecting numeric diffs), `WASTE_VERIFY=1` (crc32 every record on the
read path), `WASTE_THREADS`, `WASTE_CPUS` (cpu list the pool binds to —
`--cpus` on the CLI and the server, Linux and Windows; refused rather than
ignored elsewhere, see docs/ENGINE.md "Thread placement"),
`WASTE_DIRECT=0` (keep the page cache),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqliteai/warp](https://github.com/sqliteai/warp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
