---
trigger: always_on
description: Senior-level Python development practices for jira-mcp-snowflake project with performance and scalability focus
---


# Senior Python Development Rules for jira-mcp-snowflake

## Senior Engineer Mindset

**Think like a senior software engineer**: Always consider performance, scalability, maintainability, and long-term implications of every code change. Write code that your future self and teammates will thank you for.

## Package Management - Always Use UV

This project uses [UV](https://docs.astral.sh/uv/) for Python package management. **ALWAYS** use `uv` commands instead of `pip` or other package managers:

- **Install dependencies**: `uv sync --dev` (includes dev dependencies)
- **Run Python scripts**: `uv run <script>` 
- **Run tests**: `uv run pytest`
- **Run linting**: `uv run flake8`
- **Add new dependencies**: `uv add <package>`
- **Add dev dependencies**: `uv add --dev <package>`

Reference the project configuration in [pyproject.toml](mdc:pyproject.toml) for dependency management.

## Python Style Guidelines (PEP 8 + Senior Best Practices)

### Code Style Standards
- **Follow PEP 8** religiously with these project-specific extensions:
- **Line length**: 120 characters max (configured in flake8)
- **Imports**: Group imports (stdlib, third-party, local) with blank lines between groups
- **Type hints**: MANDATORY for all function signatures, class attributes, and complex variables
- **Docstrings**: Use Google-style docstrings for all public functions, classes, and modules
- **Variable naming**: Use descriptive names that explain intent, not just what they contain

### Code Organization
```python
# Good: Descriptive, intention-revealing names
def calculate_jira_issue_metrics_for_timeframe(start_date: datetime, end_date: datetime) -> Dict[str, int]:
    """Calculate comprehensive JIRA issue metrics for the specified timeframe."""
    pass

# Bad: Abbreviated, unclear names
def calc_metrics(start, end):
    pass
```

### Type Hints and Documentation
```python
from typing import Dict, List, Optional, Union, Any
from datetime import datetime

def process_jira_issues(
    issues: List[Dict[str, Any]], 
    filter_criteria: Optional[Dict[str, Union[str, int]]] = None
) -> Dict[str, List[Dict[str, Any]]]:
    """
    Process JIRA issues with optional filtering.
    
    Args:
        issues: List of JIRA issue dictionaries from Snowflake
        filter_criteria: Optional filtering parameters
        
    Returns:
        Dictionary grouped by issue status with processed issue data
        
    Raises:
        ValueError: If issues list is empty or malformed
    """
    pass
```

## Performance & Scalability Guidelines

### Database Operations
- **Connection pooling**: Always use connection pools for database operations
- **Query optimization**: Use LIMIT clauses, proper indexing, and avoid N+1 queries
- **Batch operations**: Process data in batches, not one-by-one
- **Async operations**: Use `asyncio` for I/O-bound operations when possible

```python
# Good: Batch processing with connection pooling
async def fetch_issues_batch(issue_keys: List[str], batch_size: int = 100) -> List[Dict]:
    """Fetch issues in batches to avoid memory issues and improve performance."""
    results = []
    for i in range(0, len(issue_keys), batch_size):
        batch = issue_keys[i:i + batch_size]
        batch_results = await fetch_issues_from_db(batch)
        results.extend(batch_results)
    return results

# Bad: Individual queries
def fetch_issues_one_by_one(issue_keys: List[str]) -> List[Dict]:
    return [fetch_single_issue(key) for key in issue_keys]  # N+1 problem
```

### Memory Management
- **Generators**: Use generators for large datasets to avoid loading everything into memory
- **Context managers**: Always use context managers for resource management
- **Lazy evaluation**: Defer expensive computations until actually needed

```python
# Good: Generator for memory efficiency
def process_large_dataset(query_results: Iterator[Dict]) -> Iterator[Dict]:
    """Process large datasets without loading everything into memory."""
    for row in query_results:
        yield transform_row(row)

# Bad: Loading everything into memory
def process_all_at_once(query_results: List[Dict]) -> List[Dict]:
    return [transform_row(row) for row in query_results]  # Memory intensive
```

### Caching Strategy
- **Implement intelligent caching** for expensive operations
- **Cache invalidation**: Always have a clear cache invalidation strategy
- **TTL-based caching**: Use time-based expiration for data that changes

```python
from functools import lru_cache
from typing import Dict, Any
import time

@lru_cache(maxsize=128)
def get_project_metadata(project_key: str) -> Dict[str, Any]:
    """Cache project metadata as it rarely changes."""
    return fetch_project_from_db(project_key)

# For time-sensitive data, implement TTL caching
class TTLCache:
    def __init__(self, ttl_seconds: int = 300):
        self.cache = {}
        self.ttl = ttl_seconds
    
    def get_with_ttl(self, key: str, fetch_func) -> Any:
        now = time.time()
        if key in self.cache:
            value, timestamp = self.cache[key]
            if now - timestamp < self.ttl:
                return value
        
        value = fetch_func()
        self.cache[key] = (value, now)
        return value
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-community-ai-tools/jira-mcp-snowflake](https://github.com/redhat-community-ai-tools/jira-mcp-snowflake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
