---
trigger: always_on
description: **Skemium** is a Java CLI tool that generates and compares [Debezium](https://debezium.io/) CDC (Change Data Capture) [Avro](https://avro.apache.org/) schemas from database tables. It detects whether database schema changes will break Debezium CDC production by comparing current vs. next versions of Avro schemas using Confluent Schema Registry compatibility checks.
---

# AGENTS.md

## Project Overview

**Skemium** is a Java CLI tool that generates and compares [Debezium](https://debezium.io/) CDC (Change Data Capture) [Avro](https://avro.apache.org/) schemas from database tables. It detects whether database schema changes will break Debezium CDC production by comparing current vs. next versions of Avro schemas using Confluent Schema Registry compatibility checks.

- **Group/Artifact**: `com.github.kafkesc:skemium`
- **Current version**: Defined in `pom.xml` (`<version>` tag)
- **License**: Apache 2.0
- **Main class**: `io.snyk.skemium.SkemiumMain`

### Three CLI commands

| Command | Purpose |
|---------|---------|
| `generate` | Connects to a PostgreSQL database, extracts table schemas, converts to Avro, saves to directory |
| `compare` | Compares two directories of generated Avro schemas for compatibility |
| `compare-files` | Compares two individual `.avsc` files for compatibility (supports external type resolution via `--include-schema`) |

## Requirements

- **JDK 21+** (AdoptOpenJDK recommended for development; GraalVM for native builds)
- **Maven 3.9+**
- **Docker** (required for tests — Testcontainers spins up PostgreSQL)
- Optional: [asdf](https://asdf-vm.com/) — run `asdf install` to get exact versions from `.tool-versions`
- Optional: [Taskfile](https://taskfile.dev/) — installed via asdf, provides shortcut tasks

### Exact versions (from `.tool-versions`)

| Tool | Version |
|------|---------|
| Maven | 3.9.12 |
| Java | adoptopenjdk-21.0.9+10.0.LTS |
| Snyk CLI | 1.1302.0 |
| Taskfile | 3.46.4 |

GraalVM is commented out in `.tool-versions` — uncomment `java oracle-graalvm-21.0.8` (and comment out the adoptopenjdk line) only for native binary builds.

## Essential Commands

> **Agent rule**: Always prefer `Taskfile.yml` tasks over raw `mvn` commands. Use `task <name>` when a matching task exists. Only fall back to crafting a direct `mvn` (or other) command when no Taskfile task covers the need.

### Taskfile Tasks (preferred)

```shell
task package                    # clean + build + test (mvn clean package)
task package.uber-jar           # clean + uber-jar, skips tests
task package.native-executable  # clean + native binary, skips tests (requires GraalVM)
task clean                      # mvn clean
task tag-version -- X.Y.Z       # Set version in pom.xml + git commit + git tag
task snyk.test                  # Run Snyk security scans
```

### Direct Maven (fallback only — use when no Taskfile task covers it)

```shell
mvn test                    # Run tests only (no Taskfile task for test-only)
mvn -B package              # CI-style build (batch mode)
```

## Code Organization

```
src/main/java/io/snyk/skemium/
├── SkemiumMain.java                 # Entry point, Picocli root command
├── BaseCommand.java                 # Abstract base for all commands (logging verbosity)
├── BaseComparisonCommand.java       # Abstract base for compare commands (compatibility, output, CI mode)
├── GenerateCommand.java             # `generate` subcommand
├── CompareCommand.java              # `compare` subcommand
├── CompareFilesCommand.java         # `compare-files` subcommand
├── CompareResult.java               # Result record for `compare`
├── CompareFilesResult.java          # Result record for `compare-files`
├── avro/
│   └── TableAvroSchemas.java        # Core data type: key/value/envelope Avro schemas for a table
├── cli/
│   └── ManifestReader.java          # Reads JAR MANIFEST.MF for version info (singleton pattern)
├── db/
│   ├── DatabaseKind.java            # Enum of supported DBs (currently only POSTGRES)
│   ├── TableSchemaFetcher.java      # Interface for fetching table schemas (extends AutoCloseable)
│   ├── CatalogSchemaAndTableTopicNamingStrategy.java  # Topic naming: <catalog>.<schema>.<table>
│   └── postgres/
│       ├── PostgresTableSchemaFetcher.java  # PostgreSQL implementation
│       └── PostgresSchemaRefreshable.java   # Package-local wrapper exposing hidden Debezium refresh()
├── helpers/
│   ├── Avro.java                    # Kafka Connect → Avro schema conversion + schema file generation
│   ├── Git.java                     # JGit helper for local repo info (commit, branch, tag)
│   ├── JSON.java                    # Jackson JSON serialization helpers (pretty, compact, from)
│   └── SchemaRegistry.java          # Compatibility checking + schema equality (JSON normalization)
└── meta/
    └── MetadataFile.java            # `.skemium.meta.json` metadata record

src/test/java/io/snyk/skemium/
├── WithPostgresContainer.java       # Base class for tests needing PostgreSQL (Testcontainers)
├── TestHelper.java                  # Test utilities (defines RESOURCES path constant)
├── GenerateCommandTest.java         # Integration tests for `generate`
├── CompareCommandTest.java          # Integration tests for `compare` + CI mode
├── CompareFilesCommandTest.java     # Tests for `compare-files` + CI mode + --include-schema

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snyk/skemium](https://github.com/snyk/skemium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
