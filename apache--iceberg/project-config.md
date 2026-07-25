---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!--
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
-->

# Apache Iceberg — Agent Instructions

Project conventions, architecture, and coding patterns synthesized from 58,000+ review comments across 4,300+ merged PRs.

## Security Model

When assessing potential vulnerabilities or calibrating automated security findings, use [`SECURITY-THREAT-MODEL.md`](SECURITY-THREAT-MODEL.md) as the authoritative detailed description of Iceberg's security boundaries, trust assumptions, and non-boundaries.

## Architecture

### Module Boundaries

- **API** (`api/`): Public interfaces and types. Changes affect every engine and catalog. API breaks are almost never acceptable.
- **Core** (`core/`): Table spec implementation. Must be engine-agnostic. No Spark/Flink references. Properties should apply to all catalogs.
- **Data** (`data/`): Generic data layer (DeleteFilter, BaseDeleteLoader, readers/writers). Behavior should be general, not engine-specific.
- **Spark** (`spark/`): Spark integration only. Tests here validate integration, not core behavior.
- **Flink** (`flink/`): Same principle as Spark — integration tests only.
- **REST Catalog** (`open-api/`): OpenAPI spec for catalog interop. Precision in spec text is critical.
- **AWS/GCP/Azure**: Cloud-specific catalog implementations. Don't leak cloud-specific assumptions into core.

The `api/` module has the strongest stability guarantees — breaking changes are almost never allowed. Other modules with public APIs (`iceberg-data`, `iceberg-parquet`, and others marked in `build.gradle`) may have breaking changes in minor releases, but they must be justified and all changes are tracked via `revapi`. New interface methods in any of these modules must include default implementations.

### High-Sensitivity Areas

- **`TableMetadata`**: Changes ripple through all engines and catalogs. Use `TableMetadata.Builder`; produce proper metadata updates for REST.
- **`SnapshotProducer` / `MergingSnapshotProducer`**: The commit path. Validations must use established patterns.
- **`ManifestGroup` / `ManifestReader`**: Container reuse causes bugs in parallel code. Callers must `copyWithoutStats` if holding references.
- **Serialization** (parsers): Never use Jackson annotations. Custom `XxxParser.toJson/fromJson` only. JSON keys use kebab-case. Optional fields only written when present.
- **REST spec**: Check for ambiguity, over-constraining, missing client-side guidance. POST for deltas, PUT for full-state replacement.
- **Scan planning**: Metrics must not leak across `TableScan` refinements. Timers must be thread-safe (parallel manifest scanning).

## Design Patterns

- **Refinement**: `TableScan` methods return new independent scans. State must not leak between refinements.
- **`CloseableIterable`** over `Stream`: Iceberg's standard lazy collection. Always close iterables.
- **Null over `Optional`**: Use `null` for missing values. `Optional` is not used.
- **Builder pattern**: For complex creation. Never require passing `null` for optional parameters.
- **Package-private by default**: Only make things public with demonstrated need.
- **Postel's Law**: Accept case-insensitive input, produce canonical output.
- **`Tasks.foreach`**: For bulk operations with parallelism, retry, and error handling.
- **Immutable metadata**: `TableMetadata`, `Schema`, `PartitionSpec`, `SortOrder` produce new instances via builders.
- **Metadata updates for REST**: All mutations must produce serializable `MetadataUpdate` objects.
- **`SerializableTable`**: Wrap table references for Spark/Flink serialization. Don't serialize the catalog.
- **Validate at boundaries**: `Preconditions` at public entry points; internal methods assume invariants hold.
- **Spec version gating**: Version 2+ features must check `formatVersion >= 2` with clear errors for v1 tables.

## Coding Conventions

### API Design

- New public methods require strong justification. Prefer package-private.
- Never break APIs. Add default implementations to new interface methods.
- Don't introduce deprecated methods in brand-new interfaces.
- Use `@SuppressWarnings({"unchecked", "rawtypes"})` internally rather than widening public signatures.
- Prefer builders over multi-argument create methods.
- Keep the `Table` API small. Utility methods go in helper classes.
- Operations should be idempotent. Return final state from `apply()`, not intermediate changes.
- Minimize third-party types (JTS, Guava) in public APIs. `StructLike` equality requires `StructLikeWrapper`.

### Naming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/iceberg](https://github.com/apache/iceberg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
