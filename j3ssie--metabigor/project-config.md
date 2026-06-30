---
trigger: always_on
description: **Project**: Metabigor - OSINT intelligence tool without API key hassle
---

# Metabigor - CLAUDE.md

**Project**: Metabigor - OSINT intelligence tool without API key hassle
**Version**: v2.1.0
**Language**: Go 1.24.0
**Author**: [@j3ssie](https://twitter.com/j3ssie)
**License**: MIT

## Project Overview

Metabigor is a command-line OSINT (Open Source Intelligence) tool designed to perform network reconnaissance and intelligence gathering without requiring API keys. It's part of the Osmedeus Engine ecosystem and focuses on seven core capabilities:

1. **Network Discovery** (`net`) - Find IP ranges (CIDRs) from ASN, organization, domain, or IP
2. **Certificate Transparency** (`cert`) - Discover subdomains via crt.sh certificate logs
3. **IP Enrichment** (`ip`) - Get port/service/vulnerability data via Shodan InternetDB (free)
4. **GitHub Code Search** (`github`) - Find secrets and credentials in public repos via grep.app
5. **IP Clustering** (`ipc`) - Group IPs by ASN for infrastructure mapping
6. **Related Domains** (`related`) - Discover related domains via cert logs, WHOIS, analytics
7. **CDN/WAF Detection** (`cdn`) - Identify if IPs are behind CDN or WAF providers

## Architecture

### Directory Structure

```
metabigor/
├── cmd/metabigor/          # Main application entry point
├── internal/               # Internal packages (not importable by external projects)
│   ├── asndb/             # ASN database management (local CSV lookups)
│   ├── cert/              # Certificate transparency search (crt.sh)
│   ├── cli/               # Cobra CLI commands and subcommands
│   ├── core/              # Core constants and configuration
│   ├── countrydb/         # Country database management
│   ├── gitsearch/         # GitHub code search via grep.app
│   ├── httpclient/        # HTTP client utilities (retryable, Chrome-based)
│   ├── ipinfo/            # IP enrichment (Shodan InternetDB) and clustering
│   ├── netdiscovery/      # Network discovery (static DB + dynamic sources)
│   ├── options/           # Global CLI options and configuration
│   ├── output/            # Output formatting (JSON, CSV, flat) and logging
│   ├── related/           # Related domain discovery (crt, WHOIS, analytics)
│   └── runner/            # Core execution runner and input processing
├── public/                # Embedded databases (ASN, country CSV files)
└── test/                  # End-to-end test scripts
```

### Key Design Patterns

- **Internal-only packages**: All logic is in `internal/` to prevent external imports
- **Cobra CLI framework**: Each command is a separate file in `internal/cli/`
- **Runner pattern**: `internal/runner` processes input (stdin, flags, files) and routes to handlers
- **Output abstraction**: `internal/output` provides consistent formatting across all commands
- **Embedded databases**: `public/` contains CSV databases embedded via `//go:embed` for offline use

### Data Flow

1. User input → CLI command (`internal/cli/`)
2. CLI initializes runner → `internal/runner/runner.go`
3. Runner processes input sources (stdin, `-i`, `-I`, `--input`)
4. Runner calls module-specific handler (`cert`, `net`, `ip`, etc.)
5. Handler queries data sources (local DB, APIs, web scraping)
6. Results formatted via `internal/output/writer.go`
7. Output to stdout or file (`-o` flag)

## Key Technical Details

### Database Management

- **ASN Database**: `~/.metabigor/ip-asn-combined.csv` (2M+ entries)
  - Downloaded via `metabigor update`
  - Source: https://github.com/iplocate/ip-address-databases
  - Used by `net` and `ipc` commands for offline ASN lookups

- **Country Database**: `~/.metabigor/ip-country-combined.csv`
  - Used for geolocation enrichment
  - Same source as ASN database

### HTTP Client Strategy

- **Retryable HTTP**: Uses `hashicorp/go-retryablehttp` for resilient API calls
- **Chrome CDP**: Uses `chromedp` for JavaScript-heavy sites (grep.app, builtwith.com)
- **Rate limiting**: Concurrent execution controlled via `-c` flag (default: 10)

### Data Sources

- **crt.sh**: Certificate transparency logs (cert, related commands)
- **Shodan InternetDB**: Free IP enrichment API (no key required)
- **grep.app**: GitHub code search
- **bgp.he.net**: Live BGP routing data (dynamic network discovery)
- **viewdns.info**: Reverse WHOIS lookups
- **builtwith.com**: Analytics tracking correlation (Google Analytics, GTM)
- **projectdiscovery/cdncheck**: CDN/WAF detection library

## Development Guidelines

### Building

```bash
make build              # Build and install to $GOPATH/bin
make install            # Install directly via go install
make test               # Run unit tests with race detection
make e2e                # Run end-to-end tests
make build-all          # Cross-compile for all platforms
```

### Code Style

- **No external imports**: Keep all logic in `internal/`
- **Error handling**: Always check errors; use `output.Error()` for user-facing messages
- **Logging**: Use `output` package methods (`Info`, `Error`, `Debug`, `Success`)
- **Silent mode**: Respect `-q/--quiet` flag - no progress messages, errors only
- **Input flexibility**: Always support stdin, `-i`, `-I` file, and `--input` flag

### Version Management

- Version is defined in `internal/core/constants.go`
- Build metadata (commit, date) injected via ldflags in Makefile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j3ssie/metabigor](https://github.com/j3ssie/metabigor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
