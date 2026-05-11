---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`tmpltool` is a fast, single-binary command-line template rendering tool built in Rust. It uses MiniJinja (Jinja2-compatible) templates with environment variables and provides extensive custom functions for hash generation, filesystem operations, data parsing, and validation.

## Big Picture - What This Tool Does

### Purpose
`tmpltool` is a **configuration file generator** that renders Jinja2-style templates by combining:
- Environment variables
- File contents (JSON, YAML, TOML)
- System information (hostname, IP, OS)
- Generated values (UUIDs, random strings, hashes)

It's designed for **DevOps/SRE workflows** like generating Kubernetes manifests, Docker Compose files, application configs, and deployment scripts.

### Typical Use Cases
1. **Kubernetes manifests** - Generate YAML with environment-specific values, resource limits, labels
2. **Docker Compose** - Template service configurations with dynamic ports, hosts, secrets
3. **Application configs** - Create config files (JSON/YAML/TOML) from environment variables
4. **Build scripts** - Generate shell scripts with embedded build metadata (git hash, timestamp)
5. **CI/CD pipelines** - Produce deployment configs that vary by environment

### Function Categories (100+ functions)

| Category | Examples | Description |
|----------|----------|-------------|
| **Environment** | `get_env`, `filter_env` | Access and filter environment variables |
| **Hash & Crypto** | `md5`, `sha256`, `sha512`, `bcrypt`, `hmac_sha256` | Checksums, password hashing, signatures |
| **Encoding** | `base64_encode/decode`, `hex_encode/decode`, `escape_html/xml/shell` | Data encoding and escaping |
| **UUID & Random** | `uuid`, `random_string`, `generate_secret` | Generate unique IDs and secure random values |
| **Date/Time** | `now`, `format_date`, `parse_date`, `date_add`, `date_diff` | Timestamps, formatting, date math |
| **Filesystem** | `read_file`, `file_exists`, `glob`, `file_size`, `list_dir` | Read files, check existence, find files |
| **Path Manipulation** | `basename`, `dirname`, `join_path`, `file_extension` | Path operations (no security restrictions) |
| **Data Parsing** | `parse_json/yaml/toml`, `read_json_file`, `read_yaml_file` | Parse structured data |
| **Data Serialization** | `to_json`, `to_yaml`, `to_toml` | Convert objects to formatted strings |
| **Object Manipulation** | `object_merge`, `object_get/set`, `object_keys/values`, `object_pick/omit` | Deep merge, nested access, transformation |
| **Validation** | `is_email`, `is_url`, `is_ip`, `is_uuid`, `matches_regex` | Validate string formats |
| **String Manipulation** | `slugify`, `to_snake_case`, `indent`, `truncate`, `regex_replace` | Case conversion, formatting, regex |
| **Array Functions** | `array_sort_by`, `array_group_by`, `array_unique`, `array_filter_by`, `array_pluck` | Sort, group, filter, transform arrays |
| **Statistics** | `array_sum`, `array_avg`, `array_median`, `array_min/max` | Numeric calculations on arrays |
| **Math** | `min`, `max`, `round`, `ceil`, `floor`, `percentage`, `abs` | Mathematical operations |
| **Logic** | `default`, `coalesce`, `ternary`, `in_range` | Conditional logic helpers |
| **Predicates** | `array_any`, `array_all`, `array_contains`, `starts_with`, `ends_with` | Boolean checks |
| **System & Network** | `get_hostname`, `get_ip_address`, `resolve_dns`, `is_port_available`, `get_os` | System info, DNS, network |
| **CIDR/IP** | `cidr_contains`, `cidr_network`, `cidr_netmask`, `ip_to_int` | IP address and subnet operations |
| **Kubernetes** | `k8s_resource_request`, `k8s_label_safe`, `k8s_probe`, `k8s_secret_ref` | K8s-specific YAML helpers |
| **Web/URL** | `parse_url`, `build_url`, `query_string`, `basic_auth` | URL manipulation, HTTP auth |
| **Command Execution** | `exec`, `exec_raw` | Run shell commands (trust mode only) |
| **Debugging** | `debug`, `inspect`, `type_of`, `assert`, `warn`, `abort` | Development and error handling |

### Key Design Decisions
1. **Explicit environment access** - Use `get_env()` instead of auto-exposing all env vars (security)
2. **Dual syntax for filters** - Most functions work as both `func(arg=x)` and `x | func` (flexibility)
3. **Dual syntax for is-functions** - All `is_*` functions work as both `is_email(string=x)` and `{% if x is email %}` (readability)
4. **Security by default** - Filesystem restricted to CWD; use `--trust` for full access
5. **Output validation** - `--validate json/yaml/toml` ensures valid output format
6. **Single binary** - No runtime dependencies, easy to deploy in containers
7. **IDE integration** - `--ide-json` outputs all function metadata (83 functions) for editor autocomplete/docs

### CLI Quick Reference
```bash
# Basic usage
tmpltool template.tmpltool                    # Render to stdout
tmpltool template.tmpltool -o output.txt      # Render to file
echo '{{ now() }}' | tmpltool             # Pipe template from stdin

# With options
tmpltool --trust system.tmpltool              # Allow filesystem access outside CWD
tmpltool config.tmpltool --validate json      # Validate output is valid JSON

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bordeux/tmpltool](https://github.com/bordeux/tmpltool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
