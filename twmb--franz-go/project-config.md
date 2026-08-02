---
trigger: always_on
description: kfake is a fake Kafka broker implementation for testing the kgo client library. Supports transactions, consumer groups, fetch sessions, ACLs, and full produce/fetch lifecycle.
---

# kfake Development Context

## Overview

kfake is a fake Kafka broker implementation for testing the kgo client library. Supports transactions, consumer groups, fetch sessions, ACLs, and full produce/fetch lifecycle.

## Running

Standalone server (has `//go:build none` tag):
```bash
go run -tags none main.go -l debug    # -l/--log-level: none, error, warn, info, debug
```

Test suite:
```bash
./run_tests.sh [options]
  -t, --test PATTERN     Test pattern (Txn, Group, Txn/range, Group/sticky)
  -n, --iterations NUM   Max iterations (default: 50)
  -r, --records NUM      Number of records (default: 500000)
  --race                 Enable race detector (uses 5min timeout)
  -l, --log-level LEVEL  Log level for both client and server (debug, info)
  --client-log LEVEL     Log level for kgo test client only
  --server-log LEVEL     Log level for kfake server only
  --clean                Kill servers and remove /tmp/kfake_test_logs
  -k, --kill             Kill processes on ports 9092-9094 and exit
```

Logs go to `/tmp/kfake_test_logs/` (server.log, client.log). On test failure, the server stays alive for debugging.

Run specific tests with standard Go:
```bash
go test -v -run TestACL ./pkg/kfake/   # ACL tests
go test -v -run TestGroup ./pkg/kfake/ # Group tests
```

Probe a live kfake server with rpk:
```bash
rpk topic list -X brokers=localhost:9092
rpk topic consume foo -X brokers=localhost:9092
rpk group describe <group> -X brokers=localhost:9092
```

Analyze test logs:
```bash
python3 tools/analyze_logs.py dual-assign                     # 848 dual-assignment failure
python3 tools/analyze_logs.py txn-timeout                     # transaction timeout root cause
python3 tools/analyze_logs.py client-trace client.log -p 3    # client debug log tracing
rpk cluster txn list -X brokers=localhost:9092                # live transaction state
```

## Key Files

- `00_produce.go` - Produce handler with transaction support
- `01_fetch.go` - Fetch handler with read_committed support and fetch sessions (KIP-227)
- `txns.go` - Transaction coordinator, EndTxn, TxnOffsetCommit
- `data.go` - Partition data, LSO calculation, batch storage, config types
- `groups.go` - Consumer group management, JoinGroup/SyncGroup/Heartbeat
- `acl.go` - ACL storage, checking logic, and authorized operations
- `config.go` - Cluster configuration options
- `cluster.go` - Main cluster struct, request routing, state initialization

## Broker Config Handling

Broker configs (`bcfgs`) are stored as `atomic.Pointer[map[string]*string]` on the cluster for race-safe access. Writes use copy-on-write via `storeBcfgs`. `validateBrokerConfig` (standalone function) validates types (Int/Long must be numeric) before writes.

Key config-driven values:
- `consumerHeartbeatIntervalMs()` reads `group.consumer.heartbeat.interval.ms` from bcfgs, default `defHeartbeatInterval` (5000 normally, 100 in test binaries)
- `maxMessageBytes()` reads from topic config then bcfgs, default `defMaxMessageBytes` (1048588)

Config maps in `data.go`: `validBrokerConfigs`, `validTopicConfigs`, `configDefaults`, `configTypes`

## Consumer Group State Machine (Classic)

States: `Empty` -> `PreparingRebalance` -> `CompletingRebalance` -> `Stable`

Rebalance triggers in `handleJoin`:
- **Stable**: Rebalance if leader rejoins OR any member's metadata changed
- **CompletingRebalance**: If member has different metadata, trigger new rebalance

## Consumer Group (Next-Gen / KIP-848)

kfake supports the next-gen consumer group protocol (ConsumerGroupHeartbeat API). The kgo client gates this behind a context opt-in: `should848()` checks for context value `"opt_in_kafka_next_gen_balancer_beta"`.

Key files:
- `groups.go` - kfake handler for ConsumerGroupHeartbeat, member/epoch management, assignment computation
- `pkg/kgo/consumer_group_848.go` - kgo client-side 848 lifecycle

Test helpers (`newClient` in behavior_test.go and helpers_test.go) automatically set the opt-in context. Direct `kgo.NewClient` calls with `kgo.ConsumerGroup` must add `kgo.WithContext(context.WithValue(ctx, "opt_in_kafka_next_gen_balancer_beta", true))` manually. No `//nolint` is needed: the lint config disables staticcheck's SA1029 globally and excludes revive's context-keys-type in `_test.go`, so the plain string key passes lint on its own.

KIP-848 assignors: uniform (default, maps to kgo sticky balancer) and range.

The heartbeat interval defaults to 5000ms but is lowered when testing to support faster partition movement reconciliation.

### 848 Reconciliation

The reconciliation follows Kafka's cooperative assignment builder pattern. Per-member state:
- `lastReconciledSent` - what the server told the member to own
- `partitionsPendingRevocation` - partitions told to revoke, awaiting client confirmation
- `targetAssignment` - what the server wants the member to own (from `computeTargetAssignment`)

Both `lastReconciledSent` and `partitionsPendingRevocation` contribute to the partition epoch map. A partition with epoch != -1 is owned and can't be claimed by another member.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twmb/franz-go](https://github.com/twmb/franz-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
