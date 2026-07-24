---
trigger: always_on
description: **Pydantic Logfire** is an observability platform built on OpenTelemetry. This repository contains the Python SDK for Logfire and documentation. The server application for recording and displaying data is closed source.
---

**Pydantic Logfire** is an observability platform built on OpenTelemetry. This repository contains the Python SDK for Logfire and documentation. The server application for recording and displaying data is closed source.

Key aspects:
- Opinionated wrapper around OpenTelemetry (traces, metrics, logs)
- Extensive integrations with popular Python packages
- SQL-based querying of telemetry data

# Code Quality

Pre-commit automatically runs ruff and pyright, but you can also run `make format/lint/typecheck` to run them explicitly, particularly to check files that haven't been changed.

# Documentation

Docs are rendered and deployed through the `pydantic/unified-docs` pipeline. Do not use MkDocs checks in this repository.

When linking between pages in this repository, use source-relative `.md` links. Published routes can
differ from source paths because `pydantic/unified-docs` remaps some sections. When adding or moving
a docs section, check whether unified-docs remaps it and verify the page and any anchor in a rendered
preview. Fix a missing remap in unified-docs instead of hard-coding a public route here, and never
include the deployment-specific `/docs` prefix.

## Writing standard

**The full documentation style guide is [`dev-docs/documentation-style-guide.md`](dev-docs/documentation-style-guide.md)** — page templates, the terminology glossary, the pre-publish checklist, the anti-pattern catalog, and the rules for AI-assisted authoring. Read it before writing or substantially editing a docs page.

Every page in the public docs (`docs/`) is held to one standard:

> If an expert in some other field who has just started building with AI tools wouldn't know what it means, we spell out the acronym, we explain the term in place, or we rewrite the sentence to be human friendly.

This is about _introducing_ terms, not avoiding them — give the real word plus a plain-language hand-hold at first use. Docstrings and other in-code text follow normal API-reference conventions and are out of scope.

# Core Structure

```
logfire/
├── __init__.py              # Public API via DEFAULT_LOGFIRE_INSTANCE
├── _internal/               # Internal implementation
│   ├── main.py              # Logfire and LogfireSpan classes
│   ├── config.py            # LogfireConfig, configuration setup
│   ├── config_params.py     # Environment variable and config file handling
│   ├── tracer.py            # ProxyTracerProvider, tracer wrapping
│   ├── metrics.py           # ProxyMeterProvider, metrics handling
│   ├── exporters/           # OTLP, console, test exporters and processors
│   ├── integrations/        # Framework-specific instrumentation
│   ├── auto_trace/          # AST rewriting for auto-instrumentation
│   └── ...
├── integrations/            # Public integration APIs
└── experimental/            # Experimental features

logfire-api/                 # No-op shim package for libraries
tests/                       # Test suite
docs/                        # Documentation source for unified docs
```

# Testing

Tests that create spans should follow this pattern:

```python
from inline_snapshot import snapshot
from logfire.testing import TestExporter
import logfire

def test_my_thing(exporter: TestExporter):
    # create spans, e.g:
    with logfire.span("a span"):
        ...

    assert exporter.exported_spans_as_dict(parse_json_attributes=True) == snapshot()
```

Then run `uv run pytest -k test_my_thing --inline-snapshot=fix` to automatically fill in `snapshot()` with a list of dicts and check that the results are sane.
If the output changes, running again will automatically update the snapshot in the code.
`TestExporter` normalizes common things. If some remaining fields are non-deterministic (e.g., IDs, timestamps), use `dirty_equals` matchers, e.g:

```python
from dirty_equals import IsStr
from inline_snapshot import snapshot

assert ... == snapshot({
    'name': 'foo',
    'random_id': IsStr(),
})
```

Use `@pytest.mark.anyio` for async tests.

Some tests are decorated with `@pytest.mark.vcr()` and use `pytest-recording` to record HTTP interactions. Existing VCR cassette files should suffice. When creating a new test like this, run `uv run pytest -k test_my_thing --inline-snapshot=fix --record-mode=rewrite`.

Tests should use user-facing APIs as much as possible. Minimize mocking and reaching into internals.

Avoid constructing `LogfireConfig` or `Logfire` instances unless absolutely necessary. Use `logfire.configure()` instead, typically with the `config_kwargs` fixture, and even then only if the default configuration done for each test doesn't already suffice.

There's no need to write `lf = logfire.configure(...); lf.foo()`. Write `logfire.configure(...); logfire.foo()`. There's also generally no need to explicitly call `logfire.shutdown()`.

# logfire-api


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pydantic/logfire](https://github.com/pydantic/logfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
