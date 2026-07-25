---
trigger: always_on
description: StarRocks is a high-performance, cloud-native analytical database system designed for real-time analytics and ad-hoc queries. It features a streamlined architecture with both shared-nothing and shared-data deployment modes, supporting sub-second query performance for complex analytical workloads.
---

# GitHub Copilot Instructions for StarRocks

## Project Overview

StarRocks is a high-performance, cloud-native analytical database system designed for real-time analytics and ad-hoc queries. It features a streamlined architecture with both shared-nothing and shared-data deployment modes, supporting sub-second query performance for complex analytical workloads.

**Key Technologies:**
- **Backend (BE)**: C++ - Core analytical engine, storage layer, and query execution
- **Frontend (FE)**: Java - SQL parsing, query planning, metadata management, and coordination
- **Java Extensions**: Java - External connectors and UDF framework
- **Testing**: Python - Integration tests and SQL test framework

## Architecture Components

### Backend (be/) - C++
The core analytical engine responsible for data storage, processing, and query execution:

**Core Components:**
- `be/src/exec/` - Query execution operators (scan, join, aggregate, etc.)
- `be/src/storage/` - Storage engine (tablets, rowsets, segments, compaction)
- `be/src/exprs/` - Expression evaluation and vectorized computation
- `be/src/formats/` - Data format support (Parquet, ORC, CSV, JSON)
- `be/src/runtime/` - Runtime services (memory management, load balancing, stream processing)
- `be/src/connector/` - External data source connectors (Hive, Iceberg, Delta Lake)
- `be/src/service/` - RPC services and BE coordination
- `be/src/common/` - Shared utilities and common data structures

**Performance Focus:**
- Vectorized query execution
- Columnar storage format
- Memory-efficient algorithms
- SIMD optimizations where applicable

📋 **Note:** See `be/.cursorrules` for detailed backend component breakdown

### Frontend (fe/) - Java
SQL interface and query coordination layer:

**Core Components:**
- `fe/fe-core/src/main/java/com/starrocks/`
  - `sql/` - SQL parser, analyzer, and AST
  - `planner/` - Query planning and optimization (CBO)
  - `catalog/` - Metadata management (tables, partitions, statistics)
  - `scheduler/` - Query scheduling and execution coordination
  - `load/` - Data loading coordination (Broker Load, Stream Load, etc.)
  - `backup/` - Backup and restore functionality
  - `privilege/` - Authentication and authorization
  - `qe/` - Query execution coordination and session management
- `fe/fe-common/` - Common frontend utilities
- `fe/plugin-common/` - Plugin framework common components
- `fe/spark-dpp/` - Spark data preprocessing integration
- `fe/hive-udf/` - Hive UDF compatibility layer

**Key Responsibilities:**
- Parse and validate SQL statements
- Generate optimized query plans using Cost-Based Optimizer (CBO)
- Manage cluster metadata and coordination
- Handle user sessions and security

📋 **Note:** See `fe/.cursorrules` for detailed frontend component breakdown

### Java Extensions (java-extensions/) - Java
External connectivity and extensibility:

**Data Source Connectors:**
- `hive-reader/` - Apache Hive integration
- `iceberg-metadata-reader/` - Apache Iceberg support
- `hudi-reader/` - Apache Hudi integration
- `paimon-reader/` - Apache Paimon support
- `jdbc-bridge/` - JDBC connectivity for external databases
- `odps-reader/` - Alibaba ODPS integration

**Extension Framework:**
- `udf-extensions/` - User-Defined Function framework
- `common-runtime/` - Shared runtime for extensions
- `hadoop-ext/` - Hadoop ecosystem integration

📋 **Note:** See `java-extensions/.cursorrules` for detailed extensions breakdown

### Additional Important Directories

**Generated Sources (gensrc/):**
- `gensrc/proto/` - Protocol buffer definitions
- `gensrc/thrift/` - Thrift interface definitions
- `gensrc/script/` - Code generation scripts

**Testing Framework (test/):**
- `test/sql/` - SQL test cases organized by functionality
- `test/common/` - Common test utilities
- `test/lib/` - Test libraries and helpers

**Tools and Utilities:**
- `tools/` - Diagnostic tools, benchmarks, and utilities
- `bin/` - Binary executables and scripts
- `conf/` - Configuration files and templates
- `build-support/` - Build system support files
- `docker/` - Docker build configurations

**Other Key Directories:**
- `thirdparty/` - External dependencies and patches
- `fs_brokers/` - File system broker implementations
- `webroot/` - Web UI static files
- `format-sdk/` - Format SDK for data interchange

## Coding Guidelines

### C++ (Backend)
```cpp
// Use modern C++ features (C++17/C++20)
// Follow Google C++ Style Guide conventions
// Use RAII for resource management
// Prefer smart pointers over raw pointers
// Use const-correctness

// Example: Vectorized processing pattern
Status ColumnProcessor::process_batch(const ChunkPtr& chunk) {
    const auto& column = chunk->get_column_by_name("column_name");
    auto result_column = std::make_shared<Column>();
    
    // Vectorized operation on entire column
    for (size_t i = 0; i < chunk->num_rows(); ++i) {
        // Process element
    }
    
    return Status::OK();
}
```

### Java (Frontend)
```java
// Follow Java coding conventions
// Use dependency injection where appropriate  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StarRocks/starrocks](https://github.com/StarRocks/starrocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
