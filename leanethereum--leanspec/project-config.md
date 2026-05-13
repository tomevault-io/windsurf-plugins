---
trigger: always_on
description: This is a Python repository for the Lean Ethereum Python specifications. It is set up as
---

# Working with leanSpec

## Repository Overview

This is a Python repository for the Lean Ethereum Python specifications. It is set up as
a single `uv` project containing the main specifications and various cryptographic
subspecifications that the Lean Ethereum protocol relies on.

## Key Directories

- `src/lean_spec/` - Main specifications for the Lean Ethereum protocol
- `src/lean_spec/subspecs/` - Supporting subspecifications for cryptographic primitives
- `tests/` - Specification tests
- `docs/` - MkDocs documentation source

## Development Workflow

### Running Tests

```bash
uv sync                           # Install dependencies
uv run pytest                     # Run unit tests
uv run fill --fork=lstar --clean -n auto                # Generate test vectors
uv run fill --fork=lstar --clean -n auto --scheme=prod  # Generate test vectors with production scheme
# Note: execution layer support is planned for future, infrastructure is ready
# for now, `--layer=consensus` is default and the only value used.
```

### Code Quality

```bash
uv run ruff format       # Format code
uv run ruff check --fix  # Lint and fix
uvx tox -e typecheck     # Type check
uvx tox -e all-checks    # All quality checks
uvx tox                  # Everything (checks + tests + docs)
```

### Common Tasks

- **Main specs**: `src/lean_spec/`
- **Subspecs**: `src/lean_spec/subspecs/{subspec}/`
- **Unit tests**: `tests/lean_spec/` (mirrors source structure)
- **Consensus spec tests**: `tests/consensus/` (generates test vectors)
- **Execution spec tests**: `tests/execution/` (future - infrastructure ready)

## Code Style

- Line length: 100 characters, type hints everywhere
- Google docstring style
- Test files/functions must start with `test_`
- **No example code in docstrings**: Do not include `Example:` sections with code blocks in docstrings. Keep documentation concise and focused on explaining *what* and *why*, not *how to use*. Unit tests serve as usage examples.
- **No section separator comments**: Never use banner-style separator comments (`# ====...`, `# ----...`, or similar). They add visual clutter with no value. Use blank lines to separate logical sections. If a section needs a heading, a single `#` comment line is enough.
- **No backtick references in comments or docstrings**: Never use RST/Markdown backticks (`` `` ``) to reference identifiers in Python comments or docstrings. This is source code, not rendered documentation. Backticks add visual noise and make comments harder to scan. Just write the name directly.
- **Never use RST-style double backticks**: If a backtick is used anyway (e.g. when quoting a literal value inline), use a single `` ` `` (Markdown), never `` `` `` (RST). Double backticks are banned everywhere — comments, docstrings, and any other prose embedded in Python source.
- **CRITICAL - Preserve existing documentation**: When refactoring or modifying code, NEVER remove or rewrite existing comments and docstrings unless they are directly invalidated by the code change. Removing documentation that still applies creates unnecessary noise in code review diffs and destroys context that was carefully written. Only modify documentation when:
  - The documented behavior has actually changed
  - The comment references code that no longer exists
  - The comment is factually wrong after your change

### Import Style

**All imports must be at the top of the file.** Never place imports inside functions, methods, or conditional blocks. This applies to both source code and tests. The **only** exception is genuine circular dependencies — in that case, import inside the function that needs the type (see the `TYPE_CHECKING` rule below).

Bad:
```python
def process(data):
    from lean_spec.subspecs.ssz import hash_tree_root
    return hash_tree_root(data)
```

Good:
```python
from lean_spec.subspecs.ssz import hash_tree_root

def process(data):
    return hash_tree_root(data)
```

**Avoid confusing import renames.** When an external library exports a name that conflicts with a local type, prefer restructuring over renaming.

Bad:
```python
from cryptography.hazmat.primitives.asymmetric.x25519 import (
    X25519PublicKey as CryptographyX25519PublicKey,
)
```

Good - import the module and use qualified access:
```python
from cryptography.hazmat.primitives.asymmetric import x25519

# Then use x25519.X25519PublicKey when needed
public_key = x25519.X25519PublicKey.from_public_bytes(data)
```

Good - move conflicting local types to a separate constants/types module:
```python
# In constants.py - no external dependencies that conflict
X25519PublicKey: TypeAlias = Bytes32

# In crypto.py - import from constants, use qualified access for external
from .constants import X25519PublicKey
from cryptography.hazmat.primitives.asymmetric import x25519
```

This keeps code readable and avoids mental overhead of tracking renamed imports.

**CRITICAL - Never use `TYPE_CHECKING`.** The `if TYPE_CHECKING:` pattern is banned entirely from this codebase. Do not import `TYPE_CHECKING` from `typing`, and do not place any imports behind `if TYPE_CHECKING:` guards. This pattern is fragile, hard to maintain, and causes subtle bugs — especially with Pydantic models.

Instead:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leanEthereum/leanSpec](https://github.com/leanEthereum/leanSpec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
