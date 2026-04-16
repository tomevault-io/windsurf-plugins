---
trigger: always_on
description: 1. **Use Pydantic to the highest degree possible**: Prefer Pydantic `BaseModel` over dictionaries, dataclasses, or untyped data structures
---

# Cognite Databricks Integration - Coding Guidelines

## Core Principles

1. **Use Pydantic to the highest degree possible**: Prefer Pydantic `BaseModel` over dictionaries, dataclasses, or untyped data structures
2. **Use PySpark when possible**: PySpark DataTypes are the source of truth for all type conversions
3. **Follow pygen-main style**: Code should look like it was written by the same developer as [pygen-main](https://github.com/cognitedata/pygen)
4. **Testing and release process**: Follow `release.md` in the workspace root for test expectations and release workflow guidance
5. **Review automation feedback**: Check Gemini Code Assist comments and GitHub PR checks for relevant PRs

## Type Safety & Data Structures

### Pydantic Models (PREFERRED)
- Use `pydantic.BaseModel` for all complex data structures
- Use `pydantic.Field` for field metadata and defaults
- Avoid `dict[str, Any]` - use typed Pydantic models instead
- Use `Field(default_factory=list)` for mutable defaults

```python
# ✅ Good - Pydantic model
from pydantic import BaseModel, Field

class UDTFConfig(BaseModel):
    """Configuration for a UDTF."""
    udtf_name: str = Field(..., description="UDTF function name")
    parameters: list[str] = Field(default_factory=list)
    
    @property
    def by_name(self) -> dict[str, str]:
        """Convenience property for dict-like access."""
        return {p: p for p in self.parameters}

# ❌ Bad - untyped dictionary
def get_config() -> dict[str, Any]:
    return {"udtf_name": "foo", "parameters": []}
```

### Type Hints (REQUIRED)
- All functions, methods, and class attributes must have type hints
- Avoid `Any` - use specific types or `TYPE_CHECKING` for complex imports
- Use `|` for union types (Python 3.10+): `str | None` not `Optional[str]`
- Use `from __future__ import annotations` for forward references

```python
# ✅ Good
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from cognite.client.data_classes.data_modeling import View

def process_view(view: View) -> str | None:
    """Process a view and return result."""
    return view.external_id if view else None

# ❌ Bad
def process_view(view):
    return view.external_id if view else None
```

## PySpark as Source of Truth

### Type Conversions
- **ALWAYS** use `TypeConverter` for all type conversions
- PySpark DataTypes are the intermediate representation
- Never duplicate type conversion logic - use `TypeConverter` methods

```python
# ✅ Good - Use TypeConverter
from cognite.databricks.type_converter import TypeConverter
from pyspark.sql.types import StringType, LongType

# Convert CDF property to PySpark, then to SQL
spark_type = TypeConverter.cdf_to_spark(property_type, is_array=False)
sql_type, type_name = TypeConverter.spark_to_sql_type_info(spark_type)
type_json = TypeConverter.spark_to_type_json(spark_type, "field_name", nullable=True)

# ❌ Bad - Manual type mapping
if isinstance(property_type, dm.Int32):
    sql_type = "INT"
    type_name = ColumnTypeName.INT
```

### PySpark DataTypes
- Use PySpark types (`StringType()`, `LongType()`, `StructType()`, etc.) for schemas
- Build schemas using `StructType([StructField(...)])` 
- Use `StructField.json()` for generating type JSON strings

```python
# ✅ Good
from pyspark.sql.types import StructType, StructField, StringType, LongType

schema = StructType([
    StructField("name", StringType(), nullable=True),
    StructField("count", LongType(), nullable=False),
])

# ❌ Bad - string-based schema
schema = "TABLE(name STRING, count INT)"
```

## Code Style ([pygen-main](https://github.com/cognitedata/pygen) alignment)

### Imports
- Group: standard library, third-party, local application
- Sort alphabetically within groups
- Use `TYPE_CHECKING` for type-only imports
- Use absolute imports

```python
# ✅ Good
from __future__ import annotations

import json
from pathlib import Path
from typing import TYPE_CHECKING

from cognite.client import CogniteClient
from pydantic import BaseModel, Field
from pyspark.sql.types import StringType, StructType

from cognite.databricks.type_converter import TypeConverter

if TYPE_CHECKING:
    from cognite.client.data_classes.data_modeling import View
```

### Naming Conventions
- Variables/functions: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Classes: `PascalCase`
- Private members: `_single_leading_underscore`
- Modules: `snake_case`

### Docstrings
- Start with concise one-line description
- Use `Args:` and `Returns:` sections for complex functions
- Keep descriptions brief and factual
- Omit obvious parameter descriptions

```python
# ✅ Good
def convert_type(property_type: object, is_array: bool = False) -> DataType:
    """Convert CDF property type to PySpark DataType.
    
    Args:
        property_type: CDF property type (e.g., dm.Text, dm.Int32)
        is_array: If True, wrap the base type in ArrayType
        
    Returns:
        PySpark DataType object
    """
    return TypeConverter.cdf_to_spark(property_type, is_array=is_array)

# ❌ Bad - too verbose
def convert_type(property_type: object, is_array: bool = False) -> DataType:
    """
    This function converts a CDF property type to a PySpark DataType.
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cognitedata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
