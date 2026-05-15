---
trigger: always_on
description: markdocify is an enterprise-grade CLI tool that comprehensively scrapes documentation websites and converts them into well-formatted, LLM-ready Markdown. Built with Go, it features concurrent scraping, intelligent content detection, and robust error handling.
---

# markdocify: Production-Ready Documentation Scraper

## 🏗️ Project Architecture

### Overview
markdocify is an enterprise-grade CLI tool that comprehensively scrapes documentation websites and converts them into well-formatted, LLM-ready Markdown. Built with Go, it features concurrent scraping, intelligent content detection, and robust error handling.

### Core Design Principles
- **Security-First**: SLSA compliance, code signing, vulnerability scanning
- **Production-Ready**: Context cancellation, graceful degradation, comprehensive logging
- **Developer-Friendly**: Zero-config operation, clear error messages, extensive documentation
- **Maintainable**: Clean architecture, comprehensive tests, automated CI/CD

## 📁 Project Structure

```
markdocify/
├── cmd/markdocify/              # CLI application entry point
│   └── main.go                  # Command-line interface, argument parsing
├── internal/                    # Private application packages
│   ├── aggregator/              # Document aggregation and output generation
│   │   ├── aggregator.go        # Page collection, TOC generation, file output
│   │   └── aggregator_test.go   # Comprehensive test suite
│   ├── config/                  # Configuration management
│   │   ├── config.go           # YAML config parsing, validation, defaults
│   │   └── config_test.go      # Configuration validation tests
│   ├── converter/               # HTML to Markdown conversion
│   │   └── converter.go        # Content sanitization, markdown generation
│   ├── scraper/                # Web scraping engine
│   │   ├── scraper.go          # Colly-based scraping, retry logic, content extraction
│   │   └── scraper_test.go     # Scraping functionality tests
│   └── types/                  # Shared data structures
│       └── types.go            # PageContent and other core types
├── configs/examples/           # Example configuration files
│   ├── nextjs-docs.yml        # Next.js documentation config
│   ├── react-docs.yml         # React documentation config
│   ├── stripe-docs.yml        # Stripe API documentation config
│   └── vercel-docs.yml        # Vercel documentation config
├── .github/workflows/          # CI/CD automation
│   ├── ci.yml                 # Testing, linting, security scanning
│   ├── release.yml            # Multi-platform builds, package distribution
│   └── security.yml           # Comprehensive security scanning
├── scripts/                   # Development and deployment scripts
│   └── verify-ci-setup.sh     # CI/CD verification script
├── test/                     # Test infrastructure
│   ├── e2e/                  # End-to-end tests
│   ├── integration/          # Integration tests
│   └── unit/                 # Unit tests
├── .goreleaser.yml           # Multi-platform release configuration
├── Dockerfile               # Container build configuration
├── Makefile                # Build automation
└── CI_CD_SETUP.md          # Complete CI/CD setup guide
```

## 🔧 Core Components

### 1. Configuration System (`internal/config/`)

**Features:**
- YAML-based configuration with sensible defaults
- Comprehensive input validation with clear error messages
- URL validation with scheme and host checking
- Regex pattern compilation and validation
- Support for multiple output formats and processing options

**Key Files:**
- `config.go`: Configuration structs, validation, URL checking
- `config_test.go`: Comprehensive test coverage including edge cases

**Example Configuration:**
```yaml
name: "React Documentation"
base_url: "https://react.dev/"
output_file: "react-complete-docs.md"

start_urls:
  - "https://react.dev/learn"
  - "https://react.dev/reference"

follow_patterns:
  - "^https://react\\.dev/(learn|reference)/.*"

processing:
  max_depth: 8
  concurrency: 3
  delay: 0.8
  preserve_code_blocks: true
  generate_toc: true
  sanitize_html: true
  scraping_timeout: "10m"

security:
  respect_robots: true
  max_file_size: "10MB"
  allowed_domains:
    - "react.dev"
```

### 2. Web Scraping Engine (`internal/scraper/`)

**Features:**
- Concurrent scraping with configurable workers and delays
- Exponential backoff retry mechanism with jitter
- Context-aware cancellation support
- Intelligent content extraction with site-specific patterns
- Privacy/legal URL filtering
- Domain validation with subdomain injection prevention
- Comprehensive structured logging

**Key Functions:**
- `RunWithContext()`: Context-aware scraping with timeout support
- `visitWithRetry()`: Retry logic with exponential backoff
- `extractContent()`: Intelligent content extraction
- `cleanTitle()`: Conservative title cleaning with pattern removal
- `isAllowedDomain()`: Secure domain validation

**Security Features:**
- Domain allowlist with exact/subdomain matching
- Automatic robots.txt compliance
- Rate limiting and respectful scraping
- Content sanitization

### 3. Content Processing (`internal/converter/`)

**Features:**
- HTML sanitization using bluemonday
- GitHub-flavored Markdown conversion
- Code block preservation
- Link resolution and cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vladkampov/markdocify](https://github.com/vladkampov/markdocify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
