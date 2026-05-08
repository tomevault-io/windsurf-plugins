---
trigger: always_on
description: Nadzoring Python framework usage rules for CLI tool and library development
---


- Always maintain strict separation between CLI layer (`nadzoring/commands/`) and domain logic (`nadzoring/dns_lookup/`, `network_base/`, `security/`, `arp/`). CLI modules should only handle Click argument parsing, progress bars, and call domain functions—never implement business logic.
- Domain layer functions must never print to stdout/stderr or call Click functions. They should return structured data (TypedDict or dataclass) and let the CLI layer handle presentation.
- All public functions in domain modules must return structured errors rather than raising exceptions for expected failures. Always include an `"error"` field in returned dictionaries that is `None` on success and contains a human-readable string on failure.
- Never use `print()` for logging. Use the project's logging module (`nadzoring.logger`) with `get_logger(__name__)` and appropriate log levels (DEBUG, INFO, WARNING, ERROR).
- Always use `logger.exception()` inside except blocks—it automatically captures and includes the traceback.

## Error Handling Pattern

All domain functions must follow the structured error return pattern:

```python
from nadzoring.dns_lookup.types import DNSResult

def resolve_with_timer(domain: str, record_type: str = "A") -> DNSResult:
    result: DNSResult = {
        "domain": domain,
        "record_type": record_type,
        "records": [],
        "ttl": None,
        "error": None,
        "response_time": None,
    }

    try:
        # ... resolution logic ...
        result["records"] = resolved_records
        result["response_time"] = measured_time
    except dns.resolver.NXDOMAIN:
        result["error"] = "Domain does not exist"
    except dns.exception.Timeout:
        result["error"] = "Query timeout"
        logger.debug("DNS timeout for %s", domain)
    except Exception as exc:
        result["error"] = str(exc)
        logger.debug("Unexpected error: %s", exc)

    return result
```

Callers should always check the `"error"` field before using other data:

```python
result = resolve_with_timer("example.com")
if result["error"]:
    print(f"DNS error: {result['error']}")
else:
    for record in result["records"]:
        print(record)
```

## CLI Command Structure

- Every CLI command must use the `@common_cli_options` decorator from `nadzoring.utils.decorators` to ensure consistent behavior for `--verbose`, `--quiet`, `--no-color`, `--output`, and `--save`.
- Commands should accept Click arguments as tuples and iterate over them, typically using `tqdm` for progress bars (respecting the `quiet` flag).
- CLI commands must return a plain list of dictionaries (or a JSON-serializable structure) containing the raw results. Formatting and saving are handled by the decorator.

```python
from nadzoring.utils.decorators import common_cli_options
from tqdm import tqdm

@dns_group.command(name="resolve")
@common_cli_options(include_quiet=True)
@click.argument("domains", nargs=-1, required=True)
def resolve_command(domains: tuple[str, ...], *, quiet: bool) -> list[dict]:
    """Resolve DNS records for one or more domains."""
    results: list[dict] = []
    pbar = None if quiet else tqdm(total=len(domains), desc="Resolving", unit="domain")

    for domain in domains:
        result = resolve_dns(domain)  # domain function call
        results.append({"domain": domain, "records": result.get("records", [])})
        if pbar:
            pbar.update(1)

    if pbar:
        pbar.close()
    return results
```

## Type Annotations

- Use Python 3.12+ modern syntax everywhere:
  - `str | None` instead of `Optional[str]`
  - `list[str]` instead of `List[str]` from typing
  - `dict[str, Any]` instead of `Dict[str, Any]`
  - `tuple[int, ...]` for variable-length tuples
- Use `type` aliases for complex, repeated type signatures.
- Define `TypedDict` classes for all structured return types (see `nadzoring/dns_lookup/types.py` for examples).
- All function parameters and return values must be fully annotated, including `-> None`.
- Use `from __future__ import annotations` only when necessary for forward references.

```python
from typing import TypedDict, Literal

type RecordType = Literal["A", "AAAA", "MX", "NS", "TXT", "CNAME", "PTR"]

class DNSResult(TypedDict, total=False):
    domain: str
    record_type: str
    records: list[str]
    ttl: int | None
    error: str | None
    response_time: float | None
```

## Docstring Format

All public functions, classes, and modules must have Google-style docstrings with `Args:`, `Returns:`, and `Examples:` sections:

```python
def traceroute(
    target: str,
    *,
    max_hops: int = 30,
    timeout: float = 2.0,
) -> list[TraceHop]:
    """
    Perform a traceroute to the specified target host.

    Uses 'traceroute' (with 'tracepath' fallback) on Linux and 'tracert'
    on Windows. Results include per-hop RTT measurements.

    Args:
        target: Hostname or IP address to trace.
        max_hops: Maximum number of hops before stopping. Defaults to 30.
        timeout: Per-hop timeout in seconds. Defaults to 2.0.

    Returns:
        List of TraceHop objects. Unreachable hops have None values for
        host/ip and rtt_ms contains [None].

    Examples:
        >>> hops = traceroute("8.8.8.8", max_hops=10)
        >>> hops[0].hop
        1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexeev-prog/nadzoring](https://github.com/alexeev-prog/nadzoring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
