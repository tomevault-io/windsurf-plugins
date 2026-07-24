---
trigger: always_on
description: SynapseML is an open-source library providing scalable machine learning pipelines
---

# SynapseML Copilot Instructions

SynapseML is an open-source library providing scalable machine learning pipelines
for Apache Spark. It wraps algorithms (LightGBM, VW, Azure AI Services, ONNX, OpenCV)
as SparkML-compatible `PipelineStage`s with auto-generated Python bindings.

## Architecture

### Module Map

| Module | Directory | Purpose |
|--------|-----------|---------|
| **core** | `core/` | Foundational transformers, featurizers, IO, codegen, automl, causal inference |
| **cognitive** | `cognitive/` | Azure AI Services wrappers (OpenAI, Vision, Speech, Text, etc.) |
| **lightgbm** | `lightgbm/` | LightGBM classifier/regressor/ranker for Spark |
| **vw** | `vw/` | Vowpal Wabbit integration |
| **deep-learning** | `deep-learning/` | ONNX Runtime inference |
| **opencv** | `opencv/` | Image transformations via OpenCV |

All modules depend on `core`. `deep-learning` also depends on `opencv`.

### Directory Layout (same pattern in every module)

```
{module}/
├── src/
│   ├── main/
│   │   ├── scala/com/microsoft/azure/synapse/ml/{package}/
│   │   │   ├── MyTransformer.scala          ← primary source code
│   │   │   └── MyTransformerParams.scala    ← parameter traits (optional)
│   │   └── python/synapse/ml/{package}/
│   │       └── MyTransformer.py             ← hand-written Python (if needed)
│   └── test/
│       ├── scala/com/microsoft/azure/synapse/ml/{package}/
│       │   └── MyTransformerSuite.scala     ← ScalaTest tests
│       └── python/synapsemltest/{package}/
│           └── test_my_transformer.py       ← Python tests
└── target/
    └── scala-2.12/generated/src/python/     ← AUTO-GENERATED (never edit)
```

## Critical: The Code Generation Pipeline

**SynapseML auto-generates Python wrappers from Scala code.** This is the most
important thing to understand.

### How It Works

1. A Scala class mixes in the `Wrappable` trait
2. Running `sbt codegen` calls `makePyFile()` which generates a Python class
3. Generated files go to `target/scala-2.12/generated/src/python/synapse/ml/`
4. Generated files use underscore prefix: `_ClassName.py`
5. Hand-written Python in `src/main/python/` can extend the generated class

### What This Means for You

- **To add or change a feature**: Edit the **Scala** code. The Python wrapper
  regenerates automatically.
- **Never edit files in `target/`**: They are overwritten on every build.
- **Hand-written Python** (`src/main/python/`) is only for cases where the
  generated wrapper needs manual overrides or additional logic.

### Example: Generated vs Hand-Written Python

Generated (DO NOT EDIT): `target/.../synapse/ml/isolationforest/_IsolationForestModel.py`

Hand-written override (OK to edit): `core/src/main/python/synapse/ml/isolationforest/IsolationForestModel.py`
```python
from synapse.ml.isolationforest._IsolationForestModel import _IsolationForestModel

class IsolationForestModel(_IsolationForestModel):
    def getInnerModel(self):
        return self._java_obj.getInnerModel()
```

## Scala Patterns

### Transformer/Estimator Pattern

Every SynapseML stage follows this pattern:

```scala
// Copyright (C) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License. See LICENSE in project root for information.

package com.microsoft.azure.synapse.ml.stages

import com.microsoft.azure.synapse.ml.codegen.Wrappable
import com.microsoft.azure.synapse.ml.logging.{FeatureNames, SynapseMLLogging}
import org.apache.spark.ml.Transformer
import org.apache.spark.ml.param._
import org.apache.spark.ml.util._
import org.apache.spark.sql.types._
import org.apache.spark.sql.{DataFrame, Dataset}

object DropColumns extends DefaultParamsReadable[DropColumns]

class DropColumns(val uid: String)
    extends Transformer with Wrappable with DefaultParamsWritable with SynapseMLLogging {
  logClass(FeatureNames.Core)

  def this() = this(Identifiable.randomUID("DropColumns"))

  val cols: StringArrayParam =
    new StringArrayParam(this, "cols", "Comma separated list of column names")

  def getCols: Array[String] = $(cols)
  def setCols(value: Array[String]): this.type = set(cols, value)

  override def transform(dataset: Dataset[_]): DataFrame = {
    logTransform[DataFrame]({
      dataset.toDF().drop(getCols: _*)
    }, dataset.columns.length)
  }

  def transformSchema(schema: StructType): StructType = {
    val droppedCols = getCols.toSet
    StructType(schema.fields.filter(f => !droppedCols(f.name)))
  }

  def copy(extra: ParamMap): DropColumns = defaultCopy(extra)
}
```

### Key Conventions

- **Companion object**: Always add `extends DefaultParamsReadable[ClassName]`
  for model serialization.
- **`Wrappable` trait**: Required for Python code generation. Without it, no
  Python wrapper is created.
- **`SynapseMLLogging` trait**: Required on all transformers/estimators. Call
  `logClass(FeatureNames.X)` in the constructor and wrap `transform`/`fit`
  with `logTransform`/`logFit`.
- **Parameter traits**: For complex stages, define params in a separate trait
  (e.g., `trait MyParams extends Wrappable with HasInputCol`) and mix it into
  the class. This is the SynapseML composition pattern.
- **`uid` parameter**: Every stage must accept `uid: String` and provide a
  no-arg constructor that generates a random UID.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/SynapseML](https://github.com/microsoft/SynapseML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
