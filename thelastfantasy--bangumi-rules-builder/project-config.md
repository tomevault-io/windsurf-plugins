---
trigger: always_on
description: A Rust application that automatically generates qBittorrent RSS download rules for anime seasons by scraping websites, processing data with AI, and integrating with Bangumi API.
---

# Smart Bangumi qBittorrent Rule Generator

A Rust application that automatically generates qBittorrent RSS download rules for anime seasons by scraping websites, processing data with AI, and integrating with Bangumi API.

## Overview

This tool automates the process of creating download rules for new anime seasons by:
1. **Web Scraping**: Extracts anime information from websites like kansou.me
2. **AI Processing**: Uses AI API (currently DeepSeek) to clean titles and generate search keywords
3. **Bangumi Integration**: Searches for official anime information via Bangumi API
4. **Rule Generation**: Creates qBittorrent RSS download rules with proper naming and filtering

## Architecture

### Core Components

#### Data Structures
- `Task`: Configuration for processing (description, site, root_path)
- `TableInfo`: Extracted HTML table with title and content
- `AnimeWork`: Raw anime data with titles, dates, and keywords
- `BangumiResult`: Processed anime data with official Bangumi information
- `Statistics`: Processing metrics and API usage tracking
- `QBRule`: qBittorrent RSS rule format with proper serialization

#### Main Workflow
1. **Configuration Loading** (`main.rs:112-130`)
   - Loads `tasks.json` for season configuration
   - Supports multiple sites (currently only "kansou")

2. **Web Scraping** (`process_kansou_site`)
   - Fetches HTML from kansou.me
   - Extracts tables with `extract_tables_with_titles()`
   - Parses anime works with `parse_table_works()`

3. **AI Processing** (`match_and_process_with_ai`)
   - Uses AI API (currently DeepSeek) for table selection and title cleaning
   - Generates search keywords in multiple languages
   - Processes works in batches to manage API limits

4. **Bangumi API Integration** (`search_bangumi_for_works`)
   - Searches Bangumi API with generated keywords
   - Uses weighted scoring system for better matching accuracy
   - Matches by air date, Japanese title, Chinese name, and aliases
   - Extracts Chinese names and aliases from infobox
   - **Improved Matching Logic** (since 2025-10-01):
     - Weighted scoring: Japanese title (50%), Chinese name (30%), aliases (20%)
     - Minimum score threshold of 0.5 to prevent false matches
     - Enhanced title matching with longest common substring algorithm
     - Date filtering handled by Bangumi API search parameters

5. **Rule Generation** (`generate_qb_rules`)
   - Creates qBittorrent RSS rules with proper regex patterns
   - Sets up download paths and categories
   - Generates season-based folder structure
   - **Duplicate Handling**: Automatically merges works with identical names
   - **Failure Tracking**: Captures and reports works that fail to generate rules

### Key Features

#### Date Processing
- Parses Japanese date formats like "2025/09/01(火)"
- Filters out undetermined dates
- Matches Bangumi air dates with fuzzy logic

#### Multi-language Support
- Japanese title variants (with/without middle dots)
- Chinese translations and aliases
- English titles for international releases

#### Smart Filtering
- Quality filters (1080p, 2160p, WebRip)
- Subtitle language detection (CHS, CHT, GB, BIG5)
- Exclusion patterns for previews and compilations

#### Detailed Reporting
- Comprehensive statistics with failure tracking
- Duplicate work detection and merging
- Rule generation failure analysis with reasons
- API usage metrics and token tracking

## Configuration

### tasks.json
```json
{
  "description": "2025年10月新番",
  "site": "kansou",
  "root_path": "E:\\Anime\\新番"
}
```

### Environment Variables
- `DEEPSEEK_API_KEY`: Required for AI processing

## Development

### Dependencies
- **serde/serde_json**: JSON serialization
- **reqwest**: HTTP client for API calls
- **scraper**: HTML parsing and extraction
- **tokio**: Async runtime
- **regex**: Pattern matching
- **chrono**: Date/time handling
- **log/env_logger/colored**: Logging system with colored output
- **indicatif**: Progress bars

### Planned Dependencies for AI Integration
- **rig-core**: Unified AI model interface for multiple providers (DeepSeek, OpenAI, Claude, Ollama, local models)
- **rig-openai** / **rig-anthropic**: Specific provider integrations (if needed)
- **llm** / **llm-chain**: Local model inference (alternative to rig-core for direct local model support)

### Building and Testing
```bash
# Build the project
cargo build

# Run tests
cargo test

# Check for warnings
cargo check

# Run specific test modules
cargo test test_workflow_with_mock_data
cargo test test_search_bangumi_with_keyword_logic
cargo test test_logger_functionality

# IMPORTANT: Do NOT automatically run the full program with `cargo run`
# The program performs web scraping and multiple API calls which can take 5-10 minutes
# Always ask the user to manually execute `cargo run` when needed
```

### Code Style
- Follow Rust naming conventions (snake_case for fields)
- Use `#[serde(rename)]` for external API compatibility
- Document public functions and structs
- Handle errors with `Result<_, Box<dyn std::error::Error>>`
- **Testing**: All tests should be created in the `tests` module within `main.rs`, not in separate files

### Git Branch Naming (Git Flow)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thelastfantasy/Bangumi-Rules-Builder](https://github.com/thelastfantasy/Bangumi-Rules-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
