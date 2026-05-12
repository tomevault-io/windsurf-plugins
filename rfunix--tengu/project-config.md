---
trigger: always_on
description: This file is the primary guide for AI assistants (Claude Code and others) working on
---

# CLAUDE.md — Tengu MCP Server

This file is the primary guide for AI assistants (Claude Code and others) working on
the Tengu codebase. Read it in full before making any changes.

---

## Project Overview

**Tengu** is a Model Context Protocol (MCP) server that exposes professional
pentesting tools to AI assistants through a clean, secure interface.

| Property        | Value                                             |
|----------------|---------------------------------------------------|
| Framework       | FastMCP 2.0+                                      |
| Python          | 3.12+                                             |
| Package manager | `uv` (full path: `/Users/rfunix/.local/bin/uv`)   |
| Validation      | Pydantic v2                                       |
| Logging         | structlog (JSON, structured)                      |
| Entry point     | `src/tengu/server.py` → `FastMCP("Tengu")`        |
| Config file     | `tengu.toml` at project root                      |
| Test suite      | 2643+ tests, 0 lint errors |
| Tools           | 80 MCP tools                                      |
| Resources       | 20 MCP resources                                  |
| Prompts         | 35 MCP prompts                                    |

---

## Architecture

### Request Flow

Every tool invocation follows this mandatory pipeline — no exceptions:

```
Claude (MCP Client)
       │
       ▼
  FastMCP layer          ← receives JSON-RPC call
       │
       ▼
1. sanitizer             ← tengu/security/sanitizer.py
   - Validate input formats (IP, hostname, CIDR, URL, port spec, etc.)
   - Strip/reject shell metacharacters: [;&|`$<>(){}\[\]!\\\'\"\\r\\n]
   - Raise InvalidInputError on any violation
       │
       ▼
2. allowlist             ← tengu/security/allowlist.py
   - Extract hostname from target
   - Check against blocked_hosts (always wins)
   - Check against allowed_hosts (must match if non-empty)
   - Raise TargetNotAllowedError on denial
       │
       ▼
3. rate_limiter          ← tengu/security/rate_limiter.py
   - Sliding window (60s) per tool
   - Concurrent slot limit per tool
   - Raise RateLimitError if either limit is exceeded
       │
       ▼
4. audit logger          ← tengu/security/audit.py
   - Write JSON audit record to logs/tengu-audit.log
   - Redact sensitive parameters (password, token, key, etc.)
   - Log tool name, target, params, result, duration
       │
       ▼
5. executor              ← tengu/executor/process.py
   - asyncio.create_subprocess_exec (NEVER shell=True)
   - Resolve tool path via shutil.which
   - Enforce timeout
   - Return (stdout, stderr, returncode)
       │
       ▼
  Output parser          ← per-tool parsing function
  (XML, JSON, JSONL, plain text)
       │
       ▼
  Return dict to Claude
```

### Module Map

```
src/tengu/
├── server.py              # FastMCP instance + tool/resource/prompt registration
├── config.py              # TenguConfig Pydantic model, load_config(), get_config()
├── types.py               # Shared Pydantic models (Host, Port, Finding, ScanResult, ...)
├── exceptions.py          # Custom exception hierarchy (TenguError subclasses)
│
├── security/
│   ├── sanitizer.py       # sanitize_target, sanitize_url, sanitize_port_spec,
│   │                      # sanitize_repo_url, sanitize_docker_image, sanitize_proxy_url
│   ├── allowlist.py       # TargetAllowlist, make_allowlist_from_config()
│   ├── rate_limiter.py    # SlidingWindowRateLimiter, rate_limited context manager
│   └── audit.py           # AuditLogger, get_audit_logger(), _redact_sensitive()
│
├── executor/
│   ├── process.py         # run_command(), stream_command() — NO shell=True
│   ├── registry.py        # check_all(), check_tool_async(), resolve_tool_path()
│   └── base.py            # (future: base executor class)
│
├── stealth/               # Stealth layer — Tor/proxy, UA rotation, timing jitter
│   ├── layer.py           # StealthLayer singleton, inject_proxy_flags()
│   ├── config.py          # StealthConfig Pydantic model
│   ├── timing.py          # Jitter utilities (random sleep ranges)
│   ├── user_agents.py     # UA rotation pool
│   └── http_client.py     # create_http_client() — httpx with proxy + UA injection
│
├── tools/
│   ├── pipeline.py        # tool_pipeline() — reusable security pipeline helper
│   ├── utility.py         # check_tools, validate_target
│   ├── recon/             # nmap, masscan, subfinder, dns, whois, amass, dnsrecon,
│   │                      # subjack, gowitness, httrack,
│   │                      # katana, httpx_probe, snmpwalk, rustscan
│   ├── web/               # nuclei, nikto, ffuf, headers, cors, ssl_tls, gobuster,
│   │                      # wpscan, testssl, wafw00f, feroxbuster
│   ├── osint/             # theharvester, shodan, whatweb, dnstwist
│   ├── injection/         # sqlmap, xss, commix, crlfuzz
│   ├── exploit/           # metasploit (msf_search, msf_module_info, msf_run_module,
│   │                      # msf_sessions_list, msf_session_cmd), searchsploit
│   ├── bruteforce/        # hydra, hash_tools (hash_crack, hash_identify), cewl
│   ├── proxy/             # zap (zap_spider, zap_active_scan, zap_get_alerts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rfunix/tengu](https://github.com/rfunix/tengu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
