---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### Quick Build (Skip Tests)
```bash
./gradlew --parallel clean build -x test
# Or use the Makefile shortcut:
make
```

### Full Build (With Tests)

Most probably cannot run on laptop.

```bash
./gradlew syncSmalldata    # Download test data from S3
./gradlew build
```

### Run H2O Locally
```bash
java -jar build/h2o.jar
# Access UI at http://localhost:54321
```

## Testing

### Java Tests
H2O uses a custom multi-node testing framework (not standard JUnit runners):

- **Single-node cluster tests**: `./gradlew :h2o-algos:testSingleNode`
- **Multi-node cluster tests**: `./gradlew :h2o-algos:testMultiNode`
- **Single JVM tests**: `./gradlew :h2o-algos:testSingleNodeOneProc`

Test files are located in:
- `h2o-core/src/test/java/` - Platform tests
- `h2o-algos/src/test/java/` - Algorithm tests (e.g., `hex/tree/gbm/GBMTest.java`)

**Running a single test class**: Use the `test.single` property:
```bash
./gradlew :h2o-algos:testSingleNode -Dtest.single=GBMTest
```

## Module Architecture

### Core Dependencies
```
h2o-genmodel (standalone POJO/MOJO scoring)
    ↓
h2o-core (distributed computing engine, DKV, REST API framework)
    ↓
h2o-algos (ML algorithms: GBM, GLM, DL, RF, etc.)
    ↓
h2o-automl (AutoML functionality)
    ↓
h2o-app (assembly: aggregates core + algos + web UI)
```

### Key Modules
- **h2o-core**: Distributed key-value store (DKV), REST API infrastructure, Frame/Vec/Chunk data structures, MRTask framework
- **h2o-algos**: Machine learning algorithms (all extend `hex.ModelBuilder`)
- **h2o-web**: Flow web UI (Node.js-based, compiled into resources)
- **h2o-genmodel**: Standalone model scoring (no H2O runtime dependencies)
- **h2o-bindings**: Generates Python/R client bindings from REST schemas
- **h2o-persist-{hdfs,s3,gcs}**: Storage backends for distributed file systems

### Extension Modules
- **h2o-ext-xgboost**: XGBoost integration
- **h2o-ext-target-encoder**: Target encoding for categorical features
- **h2o-automl**: Automated machine learning

## Distributed Architecture

### Key Concepts

**DKV (Distributed Key-Value Store)**:
- Every object has a home node determined by consistent hashing of its `Key`
- Access via `DKV.put(key, value)` and `DKV.get(key)`
- The cloud "locks" via Paxos before first DKV write to prevent mid-computation node joins

**Vec/Chunk Data Distribution**:
- `Vec`: Distributed column of data (conceptually like a database column)
- `Chunk`: Contiguous subset of a Vec (typically 1K-1M rows)
- All Vecs in a `Frame` share a `VectorGroup` ensuring chunk alignment
- Same-numbered chunks across different Vecs have identical row ranges for efficient row-wise iteration

**MRTask (Map/Reduce)**:
- Extends `MRTask` and override `map(Chunk c)` and optionally `reduce(MRTask mrt)`
- Call `.doAll(frame)` or `.dfork(frame)` to execute
- Computation moves to data (not vice versa)
- Results reduce up a tree back to the initiating node

**Iced Serialization**:
- All distributed objects extend `Iced<T>` for auto-generated serialization
- `Keyed<T>` extends Iced and adds DKV key management
- Schemas extend Iced and provide versioned REST API DTOs

### Node Communication
- UDP for heartbeats and small messages
- TCP for bulk data transfer
- Nodes form a peer-to-peer cluster (no master node for data distribution)

## REST API Structure

### Handler-Route-Schema Pattern
1. **Routes** (`water.api.Route`): Map HTTP endpoints to handler methods
2. **Handlers** (`water.api.Handler`): Process requests with signature `(int version, Schema schema)`
3. **Schemas** (`water.api.Schema`): Versioned DTOs that translate between API and internal Iced objects
4. **RequestServer**: Central routing engine

### Algorithm Registration
Algorithms auto-register REST endpoints at startup:
- Each algorithm's constructor with `startup_once=true` creates a singleton prototype
- `RegisterAlgos.java` instantiates all algorithms during H2O initialization
- Each algorithm gets standardized endpoints: `/3/ModelBuilders/<algo>`, `/3/Grid`, etc.

### Adding New REST Endpoints
1. Create Schema class extending `water.api.Schema`
2. Create Handler class extending `water.api.Handler`
3. Register route in handler via `@Route` annotation or programmatically
4. Schema fields with `@API` annotation become public API parameters

## Algorithm Implementation

### ModelBuilder Pattern
All algorithms extend `hex.ModelBuilder<M, P, O>`:
- `M`: Model class (extends `hex.Model`)
- `P`: Parameters class (extends `hex.Model.Parameters`)
- `O`: Output class (extends `hex.Model.Output`)

### Key Methods to Override
- `init()`: Validate parameters, check data compatibility
- `trainModelImpl()`: Core training logic (runs on worker nodes)
- `compute2()`: Orchestrates distributed training via MRTask

### Example Structure
```java
public class MyAlgo extends ModelBuilder<MyAlgoModel, MyAlgoModel.MyAlgoParameters, MyAlgoModel.MyAlgoOutput> {
  @Override public void init(boolean expensive) {
    super.init(expensive);
    // Validate parameters
  }

  @Override public void trainModelImpl() {
    // Core training logic
  }
}
```

### Model Scoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h2oai/h2o-3](https://github.com/h2oai/h2o-3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
