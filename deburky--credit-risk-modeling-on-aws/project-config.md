---
trigger: always_on
description: Rules for writing Python code with proper formatting and output standards for AWS ML workflows
---


## Code Quality and Formatting Tools

### Type Checking with mypy

- **NEVER use `# type: ignore`** without a specific error code (e.g., `# type: ignore[import-untyped]`)
- **NEVER use `# type: ignore` as a workaround** - fix the underlying type issue instead
- All functions must have type annotations for parameters and return values
- Use type narrowing with `isinstance()` instead of ignoring types
- For libraries without stubs, create local stubs in `stubs/` directory
- If mypy complains, it's usually right - fix the issue, don't silence it
- If stubs are not available, create local stubs in `typings/` directory
- If stubs are available, install them to `--dev` dependency group
- Use `TYPE_CHECKING` to avoid circular imports and reduce runtime overhead

#### Fixing Type Errors Without Type Ignore Comments

When mypy reports type errors, fix them properly instead of using `# type: ignore`:

- **`# type: ignore[no-any-return]`**: Use `cast()` for boto3/third-party returns: `return cast(Dict[str, Any], response)` or `return cast(str, status)`
- **`# type: ignore[arg-type]`**: Use `cast()` for arguments: `Model(env=cast(Any, env_vars))`
- **`# type: ignore[assignment]`**: Use `Any` with comment or `cast()`: `self.session: Any` or `cast(ExpectedType, value)`
- **`# type: ignore[attr-defined]`**: Use `getattr()` for dynamic attributes: `bind_method = getattr(MyClass, "bind", None)`
- **Optional narrowing**: Use assertions: `assert image_uri is not None` or `cast(str, image_uri)`

**Always import `cast` from `typing`**. Use `cast()` when you know the exact type but mypy can't infer it. Use `Any` with explanatory comments when types are truly dynamic or incompatible at type-checking time.

### Avoid sys.path Manipulation

- **NEVER use `sys.path.insert()` or `sys.path.append()`** to modify the Python import path
- This is an anti-pattern that makes code fragile, hard to test, and breaks proper package structure
- It creates implicit dependencies and makes imports unpredictable

#### Why It's Bad

```python
# BAD: sys.path manipulation
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent / "training"))
from sagemaker_pipeline import execute_pipeline_workflow
```

Problems:
- Implicit dependencies that aren't visible in the code
- Breaks when files are moved or restructured
- Makes testing difficult (imports depend on file location)
- Violates Python packaging best practices
- Can cause import conflicts and circular dependencies

#### Proper Solutions

**1. Use `__init__.py` for Package Exports**

Create `__init__.py` files to properly export functions from modules:

```python
# training/__init__.py
from sagemaker_pipeline import execute_pipeline_workflow

__all__ = ["execute_pipeline_workflow"]
```

Then import cleanly:

```python
# GOOD: Proper package import
from training import execute_pipeline_workflow
```

**2. Package Dependencies with Deployment**

For Lambda functions or other deployment scenarios, package the required modules:

```python
# setup_lambda.py - Package training module with Lambda
import zipfile
from pathlib import Path

training_dir = Path("training")
with zipfile.ZipFile("lambda.zip", "w") as zip_file:
    # Add Lambda function
    zip_file.write("lambda_function.py")
    # Add training module files
    for py_file in training_dir.glob("*.py"):
        arcname = f"training/{py_file.name}"
        zip_file.write(py_file, arcname)
```

**3. Use Relative Imports (Within Packages)**

For imports within the same package:

```python
# GOOD: Relative import within package
from .sagemaker_pipeline import execute_pipeline_workflow
```

**4. Install as Editable Package**

For development, install the project as an editable package:

```bash
# pyproject.toml or setup.py
pip install -e .
```

Then imports work naturally:

```python
from batch_scoring.training import execute_pipeline_workflow
```

### Syntax Validation with py_compile

- After finalizing code edits, run `py_compile` to verify syntax correctness:
```bash
  # Check single file
  python -m py_compile path/to/file.py
  
  # Check multiple files in a module
  python -m py_compile sagemaker_endpoints/**/*.py
  python -m py_compile batch_scoring/**/*.py
  python -m py_compile real_time_scoring/**/*.py
  python -m py_compile mlflow_on_aws/**/*.py
```
- `py_compile` catches syntax errors before runtime
- Run before committing to ensure all files are valid Python

### Docstring Validation with pydocstyle

- After writing or updating docstrings, run `pydocstyle` to verify they follow PEP 257 conventions:
```bash
  # Check single file
  pydocstyle path/to/file.py
  
  # Check multiple files in a module
  pydocstyle sagemaker_endpoints/**/*.py
  pydocstyle batch_scoring/**/*.py
  pydocstyle real_time_scoring/**/*.py
  pydocstyle mlflow_on_aws/**/*.py
  
  # Show only errors (no summary)
  pydocstyle --select=D path/to/file.py
```
- `pydocstyle` validates docstring formatting, presence, and style
- Run after writing docstrings to ensure consistency and compliance with PEP 257
- Common checks: missing docstrings, incorrect formatting, improper capitalization

#### Docstring Rules to Follow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deburky/credit-risk-modeling-on-aws](https://github.com/deburky/credit-risk-modeling-on-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
