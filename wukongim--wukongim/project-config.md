---
trigger: always_on
description: This scenario owns fixed public HTTP load tests for `/conversation/list` and
---

# Conversation QPS release gate

This scenario owns fixed public HTTP load tests for `/conversation/list` and
`/conversation/sync` on real single-node and three-node clusters with 256 hash slots.

Run: `WK_E2E_CONVERSATION_QPS=1 WK_E2E_CONVERSATION_QPS_REPORT=/tmp/conversation-qps.json GOWORK=off go test -tags=e2e ./test/e2e/message/conversation_qps -run TestConversationQPSReleaseGate -count=1 -timeout=18m -p=1 -v`

Keep thresholds and dataset sizes in `profile.json`; do not add environment overrides.
Prepare 600 generated groups through public APIs within five minutes. Use the
existing authenticated `/bench/v1/channel-runtime/evict` endpoint to unload only
those exact generated ranges on every node, respecting its busy-runtime guards.
Preparation may wait at most one minute for busy work to settle; HTTP failures
fail immediately. Do not restart processes or mix recovery traffic into the
measurement. Require zero active runtimes across all roles before and after
every phase and zero additional loads during warmup and measurement. No eviction
runs during reads. Warm disk caches are intentional; this is not a physical
cold-disk benchmark. Round-robin requests cover 24 users and all ingress nodes.
Validate every response's exact persisted message identities. Use scheduled
arrivals, bounded workers and a queue of max(workers, ceil(offered QPS * P99
budget in seconds)) entries. All scheduling/queue delay counts toward latency.
Use no measured retries, and fail on drops, errors,
incomplete pages, low QPS, high P99, runtime loads, membership writes, or missing
metrics. Record CPU, heap and allocations; enforce the reviewed per-request
allocation ceiling for each topology in addition to the QPS/P99 floor.
Only non-Linux local diagnostics may record unavailable process CPU as null;
Linux release runs require the CPU counter on every node.

The JSON report must include all 12 base cases, seven stress windows (eight endpoint results),
and source/profile/binary identities. Also retain driver GOMAXPROCS, bounded host/driver
counter snapshots around stress windows, per-scheduled-second timing/drop buckets
and at most 16 slow samples per endpoint in the original release run. These
observations do not change workload, workers, thresholds or enable profiling.
Local dirty-tree reports are diagnostic only; publication requires clean exact-tag
source evidence. Keep pure acceptance-policy tests in the default unit tier.

Capacity diagnosis is separate from publication:
`WK_E2E_CONVERSATION_CAPACITY=1 WK_E2E_CONVERSATION_CAPACITY_REPORT=/tmp/conversation-capacity/report.json GOWORK=off go test -tags=e2e ./test/e2e/message/conversation_qps -run '^TestConversationQPSCapacity$' -count=1 -timeout=20m -p=1 -v`.
Reuse the exact persisted fixture. Diagnose the representative page-100 case for
both endpoints and topologies; the release gate still covers all 12 page cases.
Use 16 driver workers per node, ten-second
bounded doubling probes (at most seven), one midpoint probe, and a fifteen-second
confirmation. Report the confirmed lower bound and observed rejected rate,
not an exact production capacity. Profile page-100 cases separately using the
existing authenticated debug API, with bounded CPU/alloc profiles per node and
a separate driver CPU profile. Profiled phases are excluded from capacity claims.
Only queue drops, latency limits, HTTP 503 refusals and the exact observed
legacy HTTP 400 head/recent-message backpressure and request-admission envelopes (pinned by unit tests) are expected
overload, and each rejects that offered rate. Other HTTP/transport errors,
malformed or incomplete successful
responses and any runtime activation or membership mutation fail diagnosis.

For a local combined run, set `WK_E2E_CONVERSATION_CAPACITY_WITH_GATE=1` alongside
the release-gate environment and run `TestConversationQPSReleaseGate` with a
20-minute timeout. It runs the same six gate phases first on each topology, then
two unprofiled capacity cases using the same prepared fixture. The report must
contain 12 passing base phases, seven passing stress windows and four confirmed capacity cases. CI does not
set this diagnostic flag and retains the fixed 18-minute gate deadline.

Throughput diagnosis is separate from the gate and capacity claims:
`WK_E2E_CONVERSATION_DIAGNOSIS=1 WK_E2E_CONVERSATION_DIAGNOSIS_REPORT=/tmp/conversation-diagnosis/report.json GOWORK=off go test -tags=e2e ./test/e2e/message/conversation_qps -run '^TestConversationQPSDiagnosis$' -count=1 -timeout=35m -p=1 -v`.
It reuses the exact fixture and capacity staircase for page 100 on both
topologies, then records three successive 60-second windows at each confirmed
rate. Preserve refused windows as rejected evidence, never a stability pass;
unexpected errors, mutations or runtime activation abort the diagnosis. Capture
bounded existing per-node CPU/GC, hydration and RPC metrics around every window.
CPU/allocation and five-second execution-trace captures run separately from
steady load. Trace bodies are capped at 64 MiB per node and use only the existing
authenticated debug API. Prefer Linux for CPU and wait attribution; do not
compare absolute capacity across different operating systems as an optimization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WuKongIM/WuKongIM](https://github.com/WuKongIM/WuKongIM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
