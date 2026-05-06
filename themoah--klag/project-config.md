---
trigger: always_on
description: Klag is a Kafka Lag Exporter built with Vert.x 4.5.22. Monitors consumer lag and group states with Prometheus/Datadog/OTLP metrics.
---

# CLAUDE.md

## Project Overview

Klag is a Kafka Lag Exporter built with Vert.x 4.5.22. Monitors consumer lag and group states with Prometheus/Datadog/OTLP metrics.

## Build Commands

```bash
# Requires Java 21 - use SDKMAN if needed: sdk use java 21.0.9-tem
# Local: use ./gradlew | CI: uses gradle directly (no wrapper JAR committed)

./gradlew compileJava          # Compile
./gradlew test                 # Run tests
./gradlew assemble             # Package (creates fat JAR)
./gradlew run                  # Run with hot-reload
```

## Architecture

Vert.x reactive framework with `Future<T>`-based async API.

```
src/main/java/io/github/themoah/klag/
├── MainVerticle.java          # Entry point, HTTP router, lifecycle
├── config/AppConfig.java      # HTTP_PORT, KAFKA_HEALTH_CHECK_INTERVAL_MS
├── health/                    # KafkaHealthMonitor, HealthCheckHandler, HealthStatus, VersionHandler
├── kafka/                     # KafkaClientService[Impl], KafkaClientConfig
├── metrics/                   # MetricsCollector, MicrometerReporter, PrometheusHandler
│   ├── velocity/              # LagVelocityTracker, TopicLagHistory
│   ├── hotpartition/          # HotPartitionDetector, HotPartitionConfig, StatisticalUtils
│   └── timelag/               # TimeLagEstimator, TimeLagConfig, OffsetTimestampTracker, PartitionOffsetHistory
└── model/                     # Records: ConsumerGroupLag, ConsumerGroupState, PartitionOffsets, LagVelocity, etc.
```

## HTTP Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/healthz` | Liveness probe (always 200) |
| `/readyz` | Readiness probe (200 if Kafka UP, 503 if DOWN) |
| `/metrics` | Prometheus scrape endpoint (if enabled) |
| `/version` | Build information |

## Environment Variables

**App:** `HTTP_PORT` (8888), `KAFKA_HEALTH_CHECK_INTERVAL_MS` (30000), `VERTX_USE_VIRTUAL_THREADS` (false)

**Kafka:** `KAFKA_BOOTSTRAP_SERVERS` (localhost:9092), `KAFKA_REQUEST_TIMEOUT_MS` (30000), `KAFKA_CHUNK_COUNT` (1), `KAFKA_CHUNK_DELAY_MS` (0)

**Metrics:** `METRICS_REPORTER` (none/prometheus/datadog/otlp), `METRICS_INTERVAL_MS` (60000), `METRICS_GROUP_FILTER` (glob pattern), `METRICS_JVM_ENABLED` (false)

**Hot Partition Detection:**
- `HOT_PARTITION_ENABLED` (true) - Enable/disable hot partition detection
- `HOT_PARTITION_SIGMA_MULTIPLIER` (2.0) - Standard deviations for outlier threshold
- `HOT_PARTITION_MIN_PARTITIONS` (3) - Minimum partitions per topic for detection
- `HOT_PARTITION_MIN_SAMPLES` (3) - Minimum samples for throughput calculation
- `HOT_PARTITION_BUFFER_SIZE` (20) - Samples to retain per partition

**Time-Based Lag Estimation:**
- `TIME_LAG_ENABLED` (true) - Enable/disable time-based lag estimation
- `TIME_LAG_MIN_MESSAGES` (100) - Minimum lag messages required for time-to-close estimates
- `TIME_LAG_INTERPOLATION_BUFFER_SIZE` (60) - Number of offset/timestamp points per partition for interpolation
- `TIME_LAG_STALE_PRODUCER_THRESHOLD_MS` (180000) - Time in ms before a producer with no offset progress is considered stale

**Logging:** `LOG_LEVEL`, `LOG_LEVEL_KLAG`, `LOG_LEVEL_KAFKA`, `LOG_LEVEL_HEALTH`, `LOG_LEVEL_METRICS`

**OTLP Configuration (when METRICS_REPORTER=otlp):**

*Standard OpenTelemetry Variables:*
- `OTEL_EXPORTER_OTLP_ENDPOINT` - Base endpoint (e.g., http://localhost:4318)
- `OTEL_EXPORTER_OTLP_METRICS_ENDPOINT` - Metrics-specific endpoint (overrides base)
- `OTEL_EXPORTER_OTLP_HEADERS` - Authentication headers (format: key1=value1,key2=value2)
- `OTEL_EXPORTER_OTLP_METRICS_HEADERS` - Metrics-specific headers (overrides general)
- `OTEL_METRIC_EXPORT_INTERVAL` - Export interval in milliseconds (default: 60000)
- `OTEL_SERVICE_NAME` - Service name for resource attributes (default: klag)
- `OTEL_RESOURCE_ATTRIBUTES` - Additional resource attributes (format: key1=value1,key2=value2)

*Custom Variables (override OTEL_* vars):*
- `OTLP_ENDPOINT` - Direct endpoint URL (default: http://localhost:4318/v1/metrics)
- `OTLP_STEP_MS` - Export interval in milliseconds (default: 60000)
- `OTLP_HEADERS` - Authentication headers (format: key1=value1,key2=value2)
- `OTLP_RESOURCE_ATTRIBUTES` - Resource attributes (format: key1=value1,key2=value2)

*Note:* Protocol is HTTP only (port 4318). Aggregation temporality is cumulative.

*Example for Grafana Cloud:*
```bash
METRICS_REPORTER=otlp
OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-prod-us-east-0.grafana.net/otlp
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic <base64-encoded-credentials>
OTEL_SERVICE_NAME=klag-production
```

*Example for Local OpenTelemetry Collector:*
```bash
METRICS_REPORTER=otlp
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_SERVICE_NAME=klag-dev
OTEL_RESOURCE_ATTRIBUTES=environment=development,cluster=local
```

## Metrics Exposed

- `klag.consumer.lag[.sum/.max/.min]` - Consumer lag (per partition and aggregated)
- `klag.partition.log_end_offset`, `klag.partition.log_start_offset`
- `klag.consumer.committed_offset`, `klag.consumer.group.state`
- `klag.topic.partitions` - Partition count per topic
- `klag.consumer.lag.velocity` - Lag velocity (messages/second × 100)

**Hot Partition Metrics (conditional - only reported when outliers exist):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themoah/klag](https://github.com/themoah/klag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
