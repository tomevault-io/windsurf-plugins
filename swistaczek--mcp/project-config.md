---
trigger: always_on
description: Generates accessible descriptions for images and GIFs using Gemini LLM.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal MCP servers collection built with FastMCP framework. The project uses Python 3.13+, mise for task management, and uv for dependency management.

## Development Commands

```bash
# Setup and Dependencies
mise run install              # Install project dependencies via uv

# Testing
uv run pytest tests/ -v       # Run full test suite (73 passed, 14 skipped expected)
uv run pytest tests/test_domains.py::TestTLDExtraction -v  # Run specific test class
uv run pytest tests/ -k "chinese" -v  # Run tests matching pattern
uv run pytest tests/ -m "not integration" -v  # Skip integration tests

# Development
mise tasks                    # List all available mise tasks
mise run dev                  # Run server in development mode
```

## Architecture

### MCP Server Pattern
Each server follows the FastMCP pattern:
1. Import FastMCP and create instance: `mcp = FastMCP("Server Name")`
2. Define tools using `@mcp.tool` decorator with Pydantic field validation
3. Use async functions for I/O operations (WHOIS queries, DNS lookups)
4. Return `ToolResult` with both human-readable text and structured JSON

### Domain Checker Architecture
Multi-layer domain availability verification with optional OVH browser-based validation.

**Verification Layers:**
1. **Primary**: WHOIS protocol queries (port 43, 10s timeout)
2. **Fallback**: DNS record lookup when WHOIS fails (potential false positives)
3. **Optional OVH Verification**: Browser automation to catch false positives

**Key Features:**
- **Batch processing**: Up to 50 domains per request
- **TLD support**: 40+ TLDs including compound (.com.cn) and Chinese IDN domains
- **Pattern matching**: TLD-specific "not found" patterns in `NOT_FOUND_PATTERNS` dict
- **False positive detection**: OVH verification catches DNS-based false positives
- **Graceful degradation**: OVH verification failures don't break core functionality

**Tool:**
`check_domains(domains, verify_available?)` - Check domain registration status
- `domains: list[str]` - Domain names to check (1-50)
- `verify_available: bool = False` - Enable OVH verification for available results

**OVH Verification Module** (`ovh_verifier.py`):
- Browser-based domain availability check using OVH's web interface
- Bypasses OVH API bot protection via Playwright automation
- Provides pricing information for available domains
- Catches false positives: domains reported available by DNS but actually registered
- **Detects aftermarket/premium domains**: Distinguishes between standard registration and secondary market

**Aftermarket Detection:**
OVH displays domain type labels that indicate aftermarket/resale domains:
- **"Premium"** - Premium domains sold at higher prices
- **"Sprzedaż przez stronę trzecią"** - Third-party sales (external marketplace)

The verifier parses these labels and marks such domains as NOT available for standard registration.

**Verification Flow:**
```python
1. WHOIS query for each domain
2. If WHOIS fails → DNS fallback (⚠️ potential false positives)
3. If verify_available=True and domain appears available:
   → OVH browser verification checks availability
   → Detects aftermarket labels (Premium, third-party)
   → Aftermarket domains marked as NOT available
   → False positives flagged if OVH says registered
```

**Response Structure (with OVH and Aftermarket):**
```json
{
  "results": {
    "catch.dev": {
      "registered": false,
      "available": false,
      "method": "dns",
      "reason": "AFTERMARKET: Domain on secondary market (Premium) - 1 384,70 zł",
      "aftermarket": true,
      "aftermarket_type": "Premium",
      "aftermarket_price": "1 384,70 zł",
      "ovh_verification": {
        "ovh_available": false,
        "ovh_verified": true,
        "ovh_price": "1 384,70 zł",
        "ovh_price_type": "premium",
        "ovh_is_aftermarket": true,
        "ovh_aftermarket_type": "Premium",
        "ovh_error": null
      }
    },
    "erne.dev": {
      "registered": false,
      "available": true,
      "method": "dns",
      "reason": "OVH CONFIRMED available (23,39 zł)",
      "ovh_confirmed": true,
      "standard_price": "23,39 zł",
      "ovh_verification": {
        "ovh_available": true,
        "ovh_verified": true,
        "ovh_price": "23,39 zł",
        "ovh_price_type": "standard",
        "ovh_is_aftermarket": false,
        "ovh_aftermarket_type": null,
        "ovh_error": null
      }
    }
  },
  "available_domains": ["erne.dev"],
  "aftermarket_domains": ["catch.dev"],
  "registered_domains": [],
  "failed_domains": [],
  "ovh_verification": {
    "enabled": true,
    "confirmed_available": ["erne.dev"],
    "aftermarket": ["catch.dev"],
    "false_positives": [],
    "verification_failed": [],
    "duration_seconds": 18.36
  }
}
```

**Domain Categories:**
- `available_domains` - Standard registration at normal prices
- `aftermarket_domains` - Secondary market (Premium/third-party) with high prices
- `registered_domains` - Already registered, not for sale
- `failed_domains` - Check failed (unsupported TLD, network error)

**Dependencies:**
- `playwright` - Browser automation (optional, for OVH verification)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swistaczek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
