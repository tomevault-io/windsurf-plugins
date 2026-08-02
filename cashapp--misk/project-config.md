---
trigger: always_on
description: Misk (Microservice Container in Kotlin) is an open source microservice container from Cash App.
---

# Misk

Misk (Microservice Container in Kotlin) is an open source microservice container from Cash App.

See [README.md](README.md) for more information about the library.

## Language

- Main language is Kotlin

## Commands

### Build

```bash
bin/gradle build --warn
```

### Test

Prefer running tests in this order: specific test class > submodule > entire library.

```bash
# Run a specific test class (preferred)
bin/gradle :misk-jdbc:test --tests "misk.jdbc.RealTransacterTest" --warn

# Run tests for a submodule
bin/gradle :misk-jdbc:test --warn

# Run all tests (least preferred - very slow)
bin/gradle test --warn
```

### Lint / Type Check

```bash
bin/gradle check --warn
```

If the public API has changed, update the `.api` files:

```bash
bin/gradle :misk-jdbc:apiDump
```

## Structure

```
misk/
├── misk/                                    # Core Misk framework
├── misk-action-scopes/                      # Action scopes for request handling
├── misk-actions/                            # Web actions framework
├── misk-admin/                              # Admin dashboard
├── misk-api/                                # API utilities
├── misk-audit-client/                       # Audit logging client
├── misk-aws/                                # AWS SDK v1 integration
├── misk-aws-dynamodb/                       # AWS DynamoDB v1 integration
├── misk-aws2-dynamodb/                      # AWS DynamoDB v2 integration
├── misk-aws2-s3/                            # AWS S3 v2 integration
├── misk-aws2-sqs/                           # AWS SQS v2 integration
├── misk-backoff/                            # Backoff and retry utilities
├── misk-bom/                                # Bill of Materials for dependency management
├── misk-clustering/                         # Clustering support
├── misk-clustering-dynamodb/                # DynamoDB-based clustering
├── misk-config/                             # Configuration management
├── misk-core/                               # Core utilities
├── misk-cron/                               # Cron job scheduling
├── misk-crypto/                             # Cryptography utilities
├── misk-datadog/                            # Datadog integration
├── misk-docker/                             # Docker utilities for testing
├── misk-events/                             # Event handling
├── misk-events-core/                        # Core event abstractions
├── misk-events-testing/                     # Event testing utilities
├── misk-exceptions-dynamodb/                # DynamoDB exception handling
├── misk-feature/                            # Feature flags
├── misk-gcp/                                # Google Cloud Platform integration
├── misk-grpc-reflect/                       # gRPC reflection
├── misk-grpc-tests/                         # gRPC testing utilities
├── misk-hibernate/                          # Hibernate ORM integration
├── misk-hibernate-testing/                  # Hibernate testing utilities
├── misk-hotwire/                            # Hotwire/Turbo integration
├── misk-inject/                             # Dependency injection utilities
├── misk-jdbc/                               # JDBC and database utilities
├── misk-jobqueue/                           # Job queue abstractions
├── misk-jooq/                               # jOOQ integration
├── misk-launchdarkly/                       # LaunchDarkly feature flags
├── misk-launchdarkly-core/                  # LaunchDarkly core
├── misk-lease/                              # Distributed leasing
├── misk-lease-mysql/                        # MySQL-based leasing
├── misk-logging/                            # Logging utilities
├── misk-mcp/                                # Model Context Protocol
├── misk-metrics/                            # Metrics abstractions
├── misk-metrics-digester/                   # Metrics digester
├── misk-moshi/                              # Moshi JSON integration
├── misk-policy/                             # Policy enforcement
├── misk-prometheus/                         # Prometheus metrics
├── misk-proto/                              # Protocol buffer utilities
├── misk-rate-limiting-bucket4j-dynamodb-v1/ # Rate limiting with DynamoDB v1
├── misk-rate-limiting-bucket4j-dynamodb-v2/ # Rate limiting with DynamoDB v2
├── misk-rate-limiting-bucket4j-mysql/       # Rate limiting with MySQL
├── misk-rate-limiting-bucket4j-redis/       # Rate limiting with Redis
├── misk-redis/                              # Redis client
├── misk-redis-lettuce/                      # Redis Lettuce client
├── misk-sampling/                           # Sampling utilities
├── misk-schema-migrator-gradle-plugin/      # Schema migration Gradle plugin
├── misk-service/                            # Service lifecycle management
├── misk-slack/                              # Slack integration
├── misk-sqldelight/                         # SQLDelight integration
├── misk-sqldelight-testing/                 # SQLDelight testing utilities
├── misk-tailwind/                           # Tailwind CSS integration
├── misk-testing/                            # Testing utilities
├── misk-testing-api/                        # Testing API abstractions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cashapp/misk](https://github.com/cashapp/misk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
