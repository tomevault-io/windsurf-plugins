---
trigger: always_on
description: Cursor rules for PySpark ETL development with code style, joins, window functions, map operations, and Iceberg patterns.
---

You are an expert in PySpark, Spark SQL, Apache Iceberg, and production data engineering. You write performant, idiomatic ETL code that is testable, readable, and safe for cumulative/snapshot tables.

Follow these rules when generating or reviewing PySpark code.

# PySpark ETL Best Practices

## 1. Project Structure

### ETL class scaffold

Create a base class that manages the SparkSession lifecycle. Accept an optional `spark_session` parameter so tests can inject a local session. Use an abstract method for the job logic.

```python
from abc import ABC, abstractmethod
from pyspark.sql import SparkSession

class BaseETL(ABC):
    def __init__(self, config, app_name="ETL Job", spark_session=None):
        self.spark = spark_session or SparkSession.builder.appName(app_name).getOrCreate()
        self.config = config
        self.logger = logging.getLogger(self.__class__.__name__)

    @abstractmethod
    def run_job(self): ...

    def stop(self):
        self.spark.stop()
```

### Config — use a factory function

Keep the dataclass as pure data and put CLI parsing in a standalone factory function. This makes configs easy to construct in tests without touching `sys.argv`.

```python
@dataclass
class MyConfig:
    read_date: int = 20200101

def create_config() -> MyConfig:
    parser = argparse.ArgumentParser()
    parser.add_argument("--read_date", type=int, default=20200101)
    args = parser.parse_args()
    return MyConfig(read_date=args.read_date)
```

### Pipeline composition with `.transform()`

Keep `run_job` as orchestration. Each step is a named method.

```python
events = self.read_source().transform(self.enrich).transform(self.merge_with_existing)
```

### Use a shared reader for partition-aware reads

Build a generic reader utility that handles partition mechanics (date filters, hour ranges, latest-partition lookups). Don't create one-off reader classes per table — keep domain-specific filters in the ETL where they're visible.

```python
class PartitionedReader:
    @staticmethod
    def read_latest(spark, table_name, partition_col):
        row = spark.read.table(table_name).agg(F.max(partition_col)).first()
        if row is None or row[0] is None:
            return spark.createDataFrame([], spark.read.table(table_name).schema)
        return spark.read.table(table_name).filter(F.col(partition_col) == row[0])

    @staticmethod
    def read_by_date(spark, table_name, partition_col, date_value):
        return spark.read.table(table_name).filter(F.col(partition_col) == date_value)

# Reader handles partitioning
events = PartitionedReader.read_by_date(spark, "catalog.my_table", "event_date", 20260319)

# Business filters stay in the ETL
events = events.filter(F.col("event_type").isin("login", "purchase"))
```

### Shared merge utilities

For simple outer-join-with-coalesce merges, build a reusable merge function that handles aliasing, join key coalescing, and per-column defaults. Use `map_zip_with` when you need per-key conflict resolution (timestamp-aware merges).

## 2. Code Style

### Use `F.col()` — always use the `F.` prefix

Import functions as `import pyspark.sql.functions as F` and use `F.col()`, `F.when()`, `F.lit()`, etc. throughout. This makes PySpark expressions immediately recognizable and greppable.

Avoid `df.colA` attribute access — it binds the column to a specific DataFrame variable, which breaks after joins or when the variable is reassigned. Use `F.col()` with `.alias()` on the DataFrame if disambiguation is needed.

```python
# BAD — binds column to a specific DataFrame variable, breaks after joins
df.select(F.lower(df1.colA), F.upper(df2.colB))

# GOOD
df.select(F.lower(F.col('colA')), F.upper(F.col('colB')))
```

### Extract complex conditions into named variables

Limit logic inside `.filter()` or `F.when()` to 3 expressions. Extract the rest.

```python
# BAD — redundant logic hidden in nested parentheses
F.when((F.col('status') == 'Delivered') | (((F.datediff('date_a', 'date_b') < 0) & ...)), 'Active')

# GOOD
is_delivered = (F.col('status') == 'Delivered')
date_passed = (F.datediff(F.col('date_a'), F.col('date_b')) < 0)
has_registration = (F.col('registration').rlike('.+'))
F.when(is_delivered | (date_passed & has_registration), 'Active')
```

### Prefer `select` over `withColumn` chains

`select` specifies the output schema in one pass. `withColumn` chains create intermediate DataFrames and can degrade performance — each call triggers a new projection in the query plan.

```python
# BAD — 3 intermediate DataFrames
df = df.withColumn("a", F.col("a").cast("double"))
df = df.withColumn("b", F.upper(F.col("b")))
df = df.withColumn("c", F.lit(1))

# GOOD — 1 DataFrame, explicit schema contract
df = df.select(
    F.col("a").cast("double"),
    F.upper(F.col("b")).alias("b"),
    F.lit(1).alias("c"),
)
```

### Use `alias` over `withColumnRenamed`

```python
# BAD
df.select('key', 'comments').withColumnRenamed('comments', 'num_comments')

# GOOD
df.select('key', F.col('comments').alias('num_comments'))
```

### Chaining limits

Max 5 statements per chain. Separate by operation type (select/filter vs withColumn vs join).

```python
# BAD — mixed concerns in one chain
df = (df.select('a', 'b', 'key')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
