---
trigger: always_on
description: Apache SkyWalking Python agent for distributed tracing, metrics, logging, and profiling.
---

# SkyWalking Python Agent

Apache SkyWalking Python agent for distributed tracing, metrics, logging, and profiling.

## Project Structure

```
skywalking/              # Main agent package
  __init__.py            # Component, Layer, Kind enums
  config.py              # All config via SW_AGENT_* env vars
  agent/                 # SkyWalkingAgent singleton, queue management, reporter threads
  trace/                 # context.py, span.py, carrier.py, segment.py, tags.py
  plugins/               # ~37 auto-instrumentation plugins (sw_*.py)
  meter/                 # Counter, gauge, histogram, PVM metrics
  log/                   # Structured logging with trace context
  profile/               # Thread/greenlet profiling
  client/                # gRPC/HTTP/Kafka protocol clients (sync + async)
  bootstrap/             # CLI (sw-python), sitecustomize loader, uwsgi hook
  utils/                 # Filters, comparators, atomic refs
  decorators.py          # @trace, @runnable for manual instrumentation
sw_python/               # CLI entry point module
tests/
  unit/                  # Standard pytest unit tests
  plugin/                # Docker-based integration tests per plugin
    base.py              # TestPluginBase with validate() method
    conftest.py          # docker_compose fixture (testcontainers)
    docker-compose.base.yml  # Mock collector + agent base services
    Dockerfile.plugin    # Agent image for plugin testing
    data/                # Data layer plugins (redis, mongo, mysql, etc.)
    http/                # HTTP client plugins (requests, urllib3, httpx, etc.)
    web/                 # Web framework plugins (flask, django, fastapi, etc.)
  e2e/                   # End-to-end tests with SkyWalking infra-e2e
  orchestrator.py        # get_test_vector() for multi-version testing
tools/
  plugin_doc_gen.py      # Auto-generate docs/en/setup/Plugins.md
  config_doc_gen.py      # Auto-generate Configuration.md
```

## Python Version Support

- **Current (master):** Python 3.8 - 3.11 (tested in CI), declared >=3.8 <=3.13
- **In-progress (PR #374):** Dropping 3.8, adding 3.12 + 3.13 to CI matrix
- **Upcoming:** Python 3.14 support needed

## Build & Development

```bash
make env          # Setup Poetry environment with all extras
make install      # Install with all optional dependencies
make lint         # Run flake8 + pylint
make fix          # Auto-fix style (unify, flynt)
make test         # Full test suite (unit + plugin docker tests)
make doc-gen      # Regenerate Plugins.md + Configuration.md
make package      # Build distribution wheel
make gen          # Generate gRPC protobuf code
```

Uses Poetry for dependency management. Config in `pyproject.toml`.

## Plugin Configuration Options (config.py)

Plugin-specific settings (all via `SW_` env vars):
- `agent_disable_plugins`: comma-separated regex patterns to skip plugins
- `plugin_http_http_params_length_threshold`: max chars for HTTP params (default 1024)
- `plugin_http_ignore_method`: comma-delimited HTTP methods to ignore
- `plugin_sql_parameters_max_length`: max SQL param length (default 0 = disabled)
- `plugin_flask_collect_http_params`, `plugin_django_collect_http_params`, `plugin_fastapi_collect_http_params`, `plugin_sanic_collect_http_params`, `plugin_bottle_collect_http_params`: collect HTTP params per framework
- `plugin_pymongo_trace_parameters` / `plugin_pymongo_parameters_max_length`: MongoDB filter tracing
- `plugin_elasticsearch_trace_dsl`: trace ES DSL
- `plugin_celery_parameters_length`: max Celery param length (default 512)
- `plugin_grpc_ignored_methods`: comma-delimited gRPC methods to ignore

Filter functions: `config.ignore_http_method_check(method)`, `config.ignore_grpc_method_check(method)`

## Context & Carrier API Details

### get_context() Signatures
```python
context.new_entry_span(op: str, carrier: Optional[Carrier] = None, inherit: Optional[Component] = None) -> Span
context.new_exit_span(op: str, peer: str, component: Optional[Component] = None, inherit: Optional[Component] = None) -> Span
context.new_local_span(op: str) -> Span
context.active_span  # property: current topmost active span
context.put_correlation(key, value)  # max 3 elements, 128 chars each
context.get_correlation(key) -> str
```

### Carrier Format
- `Carrier` class with key `sw8`, encodes 8 parts: sample, trace_id, segment_id, span_id, service, instance, endpoint, client_address (all base64 except span_id)
- `SW8CorrelationCarrier` subclass with key `sw8-correlation` for cross-process correlation context
- Iterate carrier items: `for item in carrier: headers[item.key] = item.val`
- `carrier.is_valid` / `carrier.is_suppressed` for validation

### URL Filter Utility
```python
from skywalking.utils.filter import sw_urlparse, sw_traceback
url_param = sw_urlparse(url)  # Returns parsed URL with credentials stripped from netloc
```

## Plugin Development

### Plugin Contract

Every plugin is a module file `skywalking/plugins/sw_<name>.py` with:

```python
# Required module-level attributes
link_vector = ['https://docs.example.com/']
support_matrix = {
    'package-name': {
        '>=3.7': ['1.0', '2.0']  # Python version -> tested lib versions
    }
}
note = """"""

# Required function
def install():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/skywalking-python](https://github.com/apache/skywalking-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
