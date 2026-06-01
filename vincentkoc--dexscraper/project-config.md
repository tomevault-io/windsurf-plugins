---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Python package for real-time cryptocurrency trading data from DexScreener's WebSocket API. The project has evolved from a single-file script into a full-featured package with multi-chain support, rich CLI interface, and enterprise-ready architecture supporting multiple blockchain networks including Solana, Ethereum, Base, BSC, and more.

## Commands

### Package Installation
```bash
pip install -e .[dev]  # Development install with all dependencies
pip install dexscraper  # PyPI installation (when published)
```

### Running the Application
```bash
# CLI interface
dexscraper interactive           # Rich interactive mode
dexscraper trending --chain solana --limit 10  # Simple trending pairs

# Legacy compatibility
python dex.py  # Still works, calls new package
```

### Testing
```bash
pytest tests/ -v                # Run all tests
pytest tests/ -v --cov=dexscraper --cov-report=html  # With coverage
pytest -m integration           # Integration tests only
pytest -m "not slow"           # Skip slow tests
```

### Code Quality
```bash
black dexscraper/ tests/        # Format code
isort dexscraper/ tests/        # Sort imports
flake8 dexscraper/              # Lint code
mypy dexscraper/                # Type check
pre-commit run --all-files      # Run all pre-commit hooks
```

### Security
```bash
bandit -r dexscraper/           # Security scan
safety check                    # Dependency vulnerability scan
```

## Architecture

### Package Structure
```
dexscraper/
├── __init__.py          # Package exports and version
├── scraper.py           # Main DexScraper class
├── config.py            # Configuration classes and enums
├── models.py            # Data models (TradingPair, TokenProfile, etc.)
├── protocol.py          # Binary protocol decoder (legacy)
├── enhanced_protocol.py # Enhanced protocol with OHLC support
├── cloudflare_bypass.py # Cloudflare bypass utilities
├── cli.py               # Rich-based CLI interface
├── logger.py            # Logging configuration
├── utils.py             # Utility functions
└── _version.py          # Auto-generated version (setuptools-scm)
```

### Core Components

**DexScraper Class (`scraper.py`)**:
- Multi-chain WebSocket management with automatic reconnection
- Configurable rate limiting and retry logic
- Binary protocol decoder for DexScreener's proprietary format
- Support for streaming and batch processing
- Optional Cloudflare bypass integration

**Configuration System (`config.py`)**:
- `ScrapingConfig`: Main configuration class with validation
- `PresetConfigs`: Ready-to-use configurations for common scenarios
- Enums for `Chain`, `Timeframe`, `RankBy`, `Order`, `DEX`
- `Filters` class for advanced filtering options

**Data Models (`models.py`)**:
- `TradingPair`: Complete trading pair information
- `TokenProfile`: Enhanced token metadata with social links
- `OHLCData`: MetaTrader-compatible candlestick data
- `ExtractedTokenBatch`: Batch processing results
- Export methods for JSON, CSV, and custom formats

**CLI Interface (`cli.py`)**:
- Rich-based interactive terminal interface
- Real-time data visualization with tables and progress bars
- Export functionality to multiple formats
- Configuration prompts and validation

**Key Functions**:
- `DexScraper.get_pairs()`: Batch pair retrieval
- `DexScraper.stream_pairs()`: Real-time streaming
- `decode_pair()`: Binary decoder for individual trading pairs
- `decode_enhanced_ohlc()`: OHLC data extraction
- `clean_string()`: String sanitization
- `CloudflareBypass.get_session()`: Bypass utilities

### Data Flow
1. **Configuration**: ScrapingConfig defines chains, filters, and ranking options
2. **Connection**: WebSocket connects with configurable parameters for any supported chain
3. **Protocol Handling**: Binary messages validated and decoded using enhanced protocol
4. **Data Processing**: Messages parsed in chunks with support for different data types:
   - Standard trading pairs (512-byte chunks)
   - OHLC data (candlestick format)
   - Token profiles (metadata)
5. **Output**: Clean, validated data in multiple formats (JSON, CSV, MetaTrader)
6. **Streaming**: Real-time updates via async generators or batch processing

### Binary Protocol Details
- **Message Types**: Support for multiple message formats (pairs, OHLC, profiles)
- **String Fields**: Length-prefixed (1-2 bytes length + UTF-8 data)
- **Numeric Data**: Packed as consecutive doubles with 8-byte alignment
- **Enhanced Protocol**: Additional support for:
  - OHLC candlestick data
  - Token profile metadata
  - Social media links and descriptions
- **Validation**: Built-in checks for string lengths, numeric ranges, and data integrity
- **Version Handling**: Multiple protocol versions with backward compatibility

### Testing Strategy
Comprehensive test coverage across all components:
- **Unit Tests**: Individual functions and classes
  - `test_scraper.py`: Core scraper functionality
  - `test_config.py`: Configuration validation
  - `test_models.py`: Data model serialization
  - `test_cli.py`: CLI interface testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentkoc/dexscraper](https://github.com/vincentkoc/dexscraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
