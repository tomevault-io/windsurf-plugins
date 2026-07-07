---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Luminate is a high-cardinality observability system built in Go. The key architectural decision is a **storage abstraction layer** that supports two backends:
- **ClickHouse**: Horizontally scalable (3-10 pods), stateless, production-ready
- **BadgerDB**: Single instance, embedded, zero dependencies

## Build and Development Commands

### Common Commands
```bash
# Build
make build-local        # Build for current platform
make build             # Build for all platforms (Linux, macOS)

# Test
make test              # Run all tests
make test-coverage     # Generate coverage report
go test -v ./pkg/...   # Run tests for specific package

# Run single test
go test -v -run TestMetricValidation ./pkg/models

# Development
make run               # Build and run with default config
make dev               # Run with hot reload (requires air)

# Docker
make docker-build      # Build Docker image
make docker-run        # Run container locally

# Kubernetes
make k8s-deploy        # Deploy with ClickHouse (scalable)
make k8s-deploy-badger # Deploy with BadgerDB (single instance)
make k8s-logs          # View logs
make k8s-status        # Check deployment status
```

### Testing Specific Components
```bash
# Test storage interface
go test -v ./pkg/storage/...

# Test with race detector
go test -race ./...

# Benchmark
go test -bench=. -benchmem ./pkg/storage/badger
```

## Architecture Overview

### Core Abstraction: MetricsStore Interface

Located in [pkg/storage/interface.go](pkg/storage/interface.go:19-44):

```go
type MetricsStore interface {
    Write(ctx context.Context, metrics []Metric) error

    QueryRange(ctx context.Context, req QueryRequest) (QueryResult[MetricPoint], error)
    Aggregate(ctx context.Context, req AggregateRequest) (QueryResult[AggregateResult], error)
    Rate(ctx context.Context, req RateRequest) (QueryResult[RatePoint], error)

    ListMetrics(ctx context.Context, req DiscoveryRequest) ([]string, error)
    ListDimensionKeys(ctx context.Context, req DiscoveryRequest) ([]string, error)
    ListDimensionValues(ctx context.Context, req DiscoveryRequest) ([]string, error)

    Estimate(ctx context.Context, req StoreStatsRequest) (StoreStats, error)
    DeleteBefore(ctx context.Context, req DeleteRequest) error
    Health(ctx context.Context) error
    Close() error
}
```

**Key Design:** Single interface with two implementations. BadgerDB does bounded streaming/manual aggregations, ClickHouse uses native SQL aggregations. Discovery and delete operations are tenant-scoped at the interface boundary; query methods return execution metadata for result limits and planner feedback.

**Series Identity:** Use the WS2 canonical `models.SeriesHash` helper for cardinality, ClickHouse `series_hash UInt64`, INTEGRAL partitioning, and rollup full-series keys. Do not re-hash raw dimension maps independently.

### Data Model

See [pkg/models/metric.go](pkg/models/metric.go:11-26):

**Validation Rules:**
- Metric names: `^[a-zA-Z_][a-zA-Z0-9_]*$` (1-256 chars)
- Dimension keys: Same pattern (1-128 chars)
- Dimension values: 1-256 chars, UTF-8
- Timestamps: `[now - 7 days, now + 1 hour]`
- Values: Must be finite (no NaN/Inf)
- Max 20 dimensions per metric

### Aggregation Types

Nine types supported ([pkg/storage/interface.go](pkg/storage/interface.go:56-64)):
- Basic: AVG, SUM, COUNT, MIN, MAX
- Percentiles: P50, P95, P99 (for latency tracking)
- Time-weighted: INTEGRAL (for resource consumption like CPU-seconds)

## Implementation Guidelines

### When Implementing Storage Backends

**BadgerDB** ([pkg/storage/badger/](pkg/storage/badger/)):
- Design key schema for efficient time-range scans (e.g., `metric:<name>:<timestamp>:<hash(dimensions)>`)
- Implement aggregations using in-memory iterators
- Use prefix scans for filtering
- Focus on write throughput optimization
- Accept slower query performance vs ClickHouse

**ClickHouse** ([pkg/storage/clickhouse/](pkg/storage/clickhouse/)):
- Use batch inserts with `PrepareBatch()`
- Map aggregation types to SQL functions:
  - P95 → `quantile(0.95)(value)`
  - INTEGRAL → compute deltas with `lagInFrame(...) OVER (PARTITION BY series_hash ORDER BY timestamp)`; never use a global timestamp window across different series
- Use `Map(String, String)` for dimensions column
- Partition by `toYYYYMM(timestamp)`
- Use bloom filter indexes for high-cardinality dimensions

### API Handler Pattern

All handlers should follow this pattern:

```go
func (h *Handler) Query(w http.ResponseWriter, r *http.Request) {
    // 1. Parse JSON request
    var req JSONQuery
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "INVALID_REQUEST", err)
        return
    }

    // 2. Convert to storage request
    storageReq, err := req.ToStorageQuery(r.Context())
    if err != nil {
        respondError(w, http.StatusBadRequest, "INVALID_QUERY", err)
        return
    }

    // 3. Execute query
    result, err := h.store.QueryRange(r.Context(), storageReq)
    if err != nil {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanketn26/luminate](https://github.com/sanketn26/luminate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
