---
trigger: always_on
description: - **Minimize Terminal Usage**: Use terminal commands sparingly to reduce complexity
---


# Python Coding Standards and Best Practices

## Core Principles

## Gihub Copilot Agent Usage

- **Minimize Terminal Usage**: Use terminal commands sparingly to reduce complexity
- **Windows Compatibility**: Avoid `&&` and `||` operators that may not work on all shells
- **No Directory Changes**: Avoid `cd` commands; you are always in the project root
- **terminal Usage**: run command `python3` instead of `python`, and this is the only command you should use

### Dependencies and Environment

- **Standard Library Only**: Use only Python standard library modules unless explicitly permitted
- **Self-Contained**: Code must run without third-party dependencies on Windows, macOS, and Linux
- **No External Dependencies**: Avoid pip packages to ensure maximum compatibility and easy deployment

### Python 2.7 and 3.x Compatibility

- **Primary Target**: Python 3.x is preferred
- **Legacy Support**: When Python 2.7 compatibility is required, use `six` library patterns
- **Forbidden Features**:
  - NO f-strings (not supported in Python 2.7)
  - NO `async`/`await` syntax
  - Avoid Python 3.6+ exclusive features when compatibility is needed
- **Unicode String Handling**:
    - **IMPORTANT**: Ruff formatter automatically converts `u"string"` to `"string"`
    - In Python 2, `u"\n"` and `"\n"` are different types (unicode vs bytes)
    - Prefer `# fmt: skip` on the exact line that must preserve a Unicode literal
    - Example:
        ```python
        unicode_string = u"hello\nworld"  # fmt: skip
        ```

## Project Architecture

### Directory Structure

```txt
ddns/                    # Main application code
├── provider/           # DNS provider implementations
│   ├── _base.py       # Abstract base classes (SimpleProvider, BaseProvider)
│   └── *.py           # Provider-specific implementations
├── util/              # Utility functions and classes
│   ├── http.py        # HTTP client functionality
│   ├── config.py      # Configuration management
│   └── *.py           # Other utilities
└── __init__.py        # Package initialization

tests/                   # Unit tests
├── base_test.py        # Shared test utilities and base classes
├── test_provider_*.py  # Provider-specific tests
└── README.md          # Testing documentation

doc/                     # Documentation
├── cli.md              # Command line interface documentation
├── docker.md           # Docker usage and deployment guide
├── env.md              # Environment variables reference
├── json.md             # JSON configuration format
├── dev/                # Developer documentation
│   └── provider.md     # Provider development guide
├── providers/          # Provider-specific documentation
│   ├── dnspod.md       # DNSPod configuration guide
│   ├── cloudflare.md   # Cloudflare configuration guide
│   └── *.md            # Other provider guides
└── img/                # Documentation images and diagrams
    ├── ddns.png        # Project logo and icons
    └── ddns.svg        # Vector graphics and diagrams
      
schema/                  # JSON schemas
├── v2.8.json           # Legacy configuration schema v2.8
├── v2.json             # Legacy configuration schema v2
└── v4.0.json           # Current configuration schema v4.0
```

### Provider Architecture

#### SimpleProvider (Basic DNS Provider)

**Purpose**: For DNS providers that only support simple record updates without querying existing records.

**Must Implement**:

- `set_record(domain, value, record_type="A", ttl=None, line=None, **extra)`
  - Updates or creates DNS records
  - Should handle both creation and update logic, if supported by the provider
  - Must return `True` on success, `False` on failure with appropriate error logging
  - Should never raise exceptions for API failures

**Optional**:

- `_validate()` - Custom authentication validation (has default implementation)

**Available Methods**:

- `_http(method, url, ...)` - HTTP/HTTPS requests with automatic error handling
- `_mask_sensitive_data(data)` - Log-safe data masking for security (supports URL-encoded data)

#### BaseProvider (Full CRUD DNS Provider - Recommended for Most Providers)

**Purpose**: For DNS providers supporting complete DNS record management with query capabilities.

**Must Implement**:

- `_query_zone_id(domain)` - Retrieves zone ID for a domain by calling domain info or list domains/zones API
- `_query_record(zone_id, subdomain, main_domain, record_type, line, extra)` - Finds existing DNS record by calling list records or query record API
- `_create_record(zone_id, subdomain, main_domain, value, record_type, ttl, line, extra)` - Creates new DNS record by calling create record API
- `_update_record(zone_id, old_record, value, record_type, ttl, line, extra)` - Updates existing DNS record by calling update record API

**Recommended Practices**:

- Implement a `_request()` method for signed/authenticated HTTP requests:
  - Should raise `Exception` or `RuntimeError` on blocking errors for fast failure
  - Should return `None` or appropriate default on recoverable errors (e.g., NotFound)
- Use `self.logger` for consistent logging throughout the provider

**Inherited Methods**:

- `_http()` - HTTP requests with authentication error handling (raises RuntimeError on 401/403)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NewFuture/DDNS](https://github.com/NewFuture/DDNS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
