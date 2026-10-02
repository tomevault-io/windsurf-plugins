---
trigger: always_on
description: **Every timed run reads its checkpoint from `~/models` on the machine doing the timing.
---

# goinfer — agent notes

## Models: stored in the archive, benchmarked from local disk

**Every timed run reads its checkpoint from `~/models` on the machine doing the timing.
The archive is storage, never a read path for a measurement — on either machine.**

| | bench set — the ONLY place a row may be measured from | the archive, which is NOT a bench surface |
|---|---|---|
| `nobara-pc` (amd64/CUDA) | `~/models` on NVMe | **`/srv/models`** — local, but a 5400 rpm SMR disk |
| MacBook (arm64/Metal) | `~/models` on internal SSD | **`/Volumes/…`** — the SMB mount of the same disk |

Both forbidden roots are named above because **neither prohibition catches the other**.
`/srv/models` is *local storage on the box that measures CUDA*, so "benchmark from local
disk" reads as permission for it; `/Volumes/` is the only path the older phrasing named,
and it does not exist on Linux. A run off either one measures a 5400 rpm SMR disk — over
the LAN as well, in the `/Volumes/` case — instead of the engine. **It does not error.** It
returns a plausible, wrong number. Any row whose model path starts with `/srv/models` or
`/Volumes/` is void and must be re-measured after a `models-pull`.

The full storage table, the `models-pull` / `models-push` usage, and the reason the share is
deliberately not automounted live in `docs/benchmarks.md` § "Model storage" — **that section
is the authority; do not restate its details here.** Enough to work from: `models-pull <name>`
copies archive → `~/models` (resumable, rsync over SSH, must be on the LAN — Tailscale SSH
intercepts port 22 and hangs), `models-push <name>` goes the other way and refuses to claim
success unless byte counts match both sides.

## Benchmarking

`docs/benchmarks.md` is provenance-gated: a number enters a table only with machine,
checkpoint+quant, greedy/seed, pinned versions, date, thermal note, and local-disk path.
Read its Methodology section before adding or changing any measurement, and reproduce via
`scripts/bench_peer.py` — **not** `scripts/bench_compare.sh`, which runs in-process Go benchmarks and
by its own design note drives no peer. Putting its output beside a peer number divides a kernel
throughput by an end-to-end one; that is what produced the retired "0.5B 1.78×" claim.
`bench_compare.sh` is still the right tool for goinfer-vs-goinfer work, and only that. Peer
comparisons must be same-session interleaved. Same-build cross-session drift is ~0.7% RMS for peers on nobara (tail
to ~5%); goinfer's own is measured once, 3.6%, in a sampled cell. A cross-session ratio is not a ratio
(`docs/measurements/noise-registry.md`, TE3).

CUDA rows are anchored to a specific NVIDIA driver version. Changing the driver invalidates
comparability and requires a deliberate re-anchor, not a silent carry-forward.

## Working in the tree

**Five Go modules**, not one: the root, `gpu/`, `cuda/`, `metal/`, `demo/agent/`. `go.work` is
gitignored and **mandatory** for cross-module work; a `GOWORK=off` build of a submodule resolves
the root from the proxy at its last published tag, so it can fail with "method not found" on
perfectly good code. That failure is expected between releases, not a bug.

**Build tags gate real code.** `realckpt` (heavy, real checkpoints), `goinfer_testhooks`
(cross-module test seams), `cuda` / `gpu` / `metal`. A change that compiles untagged can be
broken under a tag — this has shipped to `main` at least twice. `go vet -tags realckpt ./...`
and `go vet -tags 'cuda goinfer_testhooks' ./cuda/` are cheap; run the ones your change touches.

**`gofmt -l .` is a CI gate and nothing here auto-formats.** Run it across every module you
touched before committing.

**`staticcheck` is also a CI gate, and a LOCAL RUN CAN SILENTLY CHECK NOTHING.** Install CI's pinned
version once — `go install honnef.co/go/tools/cmd/staticcheck@v0.8.0` — then run the tagged variants
(`-tags 'cuda goinfer_testhooks' ./cuda/...`, `-tags 'gpu goinfer_testhooks' ./gpu/...`).

**Do NOT combine `go run …@v0.8.0` with `GOOS=linux`.** `go run` then builds a LINUX staticcheck and
fails to exec it on darwin — `exec format error`, **and the shell still reports exit 0**. Use an
INSTALLED native binary and set `GOOS` only for the analysis target:
`GOOS=linux GOARCH=amd64 CGO_ENABLED=0 ~/go/bin/staticcheck -tags cuda ./...`.

**An empty staticcheck result is indistinguishable from one that never ran, so prove the gate can go
red.** A four-line throwaway package with an unused struct field must print
`field deadField is unused (U1000)`. That is the exact defect that held CI red across three pushes on
2026-08-28, and the same class `cmd/gate/mutation.go` was built to prevent.
A `staticcheck` on PATH can be years older than the toolchain and then fails to analyse ANYTHING,
emitting only `internal error in importing "internal/cpu" (unsupported version: 4)` — which looks
like a broken tool rather than an unrun gate, and exits without checking your code. Measured
2026-08-28: a dead struct field (U1000) shipped in `decoder/mtp.go` and held CI red across **three
pushes**, because `gofmt` and `go vet` were run and staticcheck was not.

**Check CI after pushing.** `gh run list --limit 5`. Red does not announce itself, and a break rides

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [townsendmerino/goinfer](https://github.com/townsendmerino/goinfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
