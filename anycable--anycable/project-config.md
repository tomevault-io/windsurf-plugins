---
trigger: always_on
description: Loaded as context when an assistant works inside `etc/benchi/`. This file is
---

# Guidance for running Benchi benchmarks

Loaded as context when an assistant works inside `etc/benchi/`. This file is
for **operating** the benchmarks; for modifying Benchi itself, see
`README.md` and the implementation plan in `docs/plans/`.

## Always pass `--non-interactive`

When you run a scenario binary on the user's behalf, pass `--non-interactive`.
Without it, Benchi auto-detects a TTY and emits lifecycle logs plus a
progress bar on stderr — useful for a human, noisy for you. The flag
suppresses everything except the key=value summary on stdout, which is what
you want to parse and report back.

```sh
go run ./scenarios/stress_publications \
  -c 1000 -r 100 -d 10s -S 10 -s 2 --non-interactive
```

Running from outside `etc/benchi/`? Either `cd` first (Benchi is a nested
Go module) or use `go run -C etc/benchi ./scenarios/...`.

## Required flags

The scenario refuses to run with zero defaults — there is no "safe accidental
benchmark." If you omit any of `-c`, `-r`, `-d`, `-S`, `-s`, the binary
exits non-zero with a usage message on stderr. This is intentional;
do not work around it.

| Flag | Meaning |
|---|---|
| `-c` | clients (WebSocket connections) |
| `-r` | publishes per second |
| `-d` | publish window duration (Go duration syntax: `1s`, `10s`, `500ms`) |
| `-S` | size of the stream universe (streams named `"1"`..`"S"`) |
| `-s` | per-client subscription count (must be ≤ `-S`) |

Optional knobs with safe defaults: `--drain-timeout=30s`, `--max-inflight=4096`,
`--publish-workers=64`, `--publish-batch=64`, `--setup-failure-tolerance=0`,
`--seed=1`.

`--publish-workers` is the size of the HTTP worker pool draining the
publisher queue. The default (64) is sized for the **embedded-server
scenario** where each POST to `/broadcast` triggers synchronous fan-out
across every subscribed client — high publisher concurrency multiplies
server-side work and can exhaust the Go OS-thread limit at high `-c`
(`runtime: failed to create new OS thread`). When pointing the publisher
at an **external broadcaster** (NATS, Redis, an edge service) each POST
is cheap and values up to ~1024 are reasonable; raise it to find that
broadcaster's ceiling.

`--publish-batch` is the maximum number of broadcasts a single worker
will coalesce into one POST under backpressure (`[{stream,data}, …]`
array body, supported by the AnyCable broadcast endpoint). At low rates
batches collapse to size 1 with no added latency; when the broadcast
endpoint can't keep up, batching amortizes per-request overhead and is
often the difference between a useful number and a 200× under-report.

## Picking a configuration

Average per-publish fan-out is `c × s / S`. Total expected throughput is
roughly `r × c × s / S` messages per second. Standard ladder, lowest to
highest load:

| Profile | Command | Target msgs/sec |
|---|---|---|
| Smoke (sanity check) | `-c 10 -r 10 -d 1s -S 1 -s 1` | 100 |
| Mid baseline | `-c 1000 -r 100 -d 10s -S 10 -s 2` | 20,000 |
| Realistic mixed | `-c 1000 -r 500 -d 10s -S 50 -s 10` | 100,000 |
| Full fan-out | `-c 10000 -r 100 -d 10s -S 1 -s 1` | 1,000,000 |

Pick the lowest profile that exercises the dimension the user cares about.
If they want hub regression detection, prefer "Realistic mixed" — every
dimension is non-zero so a change anywhere shows up.

## Reading the summary

The summary prints two families of rate metrics — throughput (messages
received by clients) and observed publish rate (POSTs the server
accepted) — each as an overall average plus four sliding-window
statistics (window size = `-d`, sampled every 100ms). The fast triage:

| Key | What it means | What to flag |
|---|---|---|
| `throughput_msgs_per_sec` | **Overall average:** total messages received divided by total receive-timeline wall seconds (publish + drain). | A whole-run average — useful as one-line summary, but `max` is the better success metric since a long drain tail can drag this number down. |
| `throughput_max_msgs_per_sec` | **Headline / success metric.** Highest receive rate sustained over any sliding `-d`-second window. | Compare to `expected_msgs_per_sec`. Above ~95% = clean run. Below ~80% = the system never managed the target rate even momentarily — real fan-out regression. |
| `throughput_min_msgs_per_sec` | Lowest receive rate over any full `-d`-second window. | A very low min means a stall — useful when investigating GC pauses, broker reconnections, or scheduler hiccups. |
| `throughput_p50_msgs_per_sec` | Median windowed rate. The "typical sustained" rate across the receive timeline. | If `p50` ≪ `max`, the peak was brief; the run is bursty rather than steady. Investigate with `min` and `p95`. |
| `throughput_p95_msgs_per_sec` | 95th-percentile windowed rate. The "near-peak" rate the system held most of the time. | If `max ≈ p95` the system was steady at peak; if `max ≫ p95` the peak was a single window. |
| `expected_msgs_per_sec` | Theoretical ceiling: `r × c × s / S`. | Constant for a given config — printed so `max / expected` is one division away. |
| `clients_complete` / `clients_short` | Reconciliation: did every client receive its share? | Any `clients_short > 0` is a **validity** failure. Report it; don't gloss over it. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anycable/anycable](https://github.com/anycable/anycable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
