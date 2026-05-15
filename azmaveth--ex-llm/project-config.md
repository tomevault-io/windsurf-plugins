---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Version Management

### When to Bump Versions
- **Patch version (0.x.Y)**: Bug fixes, documentation updates, minor improvements
- **Minor version (0.X.0)**: New features, non-breaking API changes, new provider adapters
- **Major version (X.0.0)**: Breaking API changes (after 1.0.0 release)

### Version Update Checklist
1. Update version in `mix.exs`
2. Update CHANGELOG.md with:
   - Version number and date
   - Added/Changed/Fixed/Removed sections
   - **BREAKING:** prefix for any breaking changes
3. Commit with message: `chore: bump version to X.Y.Z`

### CHANGELOG Format
```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features or providers

### Changed
- Changes in existing functionality
- **BREAKING:** API changes that break compatibility

### Fixed
- Bug fixes

### Removed
- Removed features
- **BREAKING:** Removed APIs
```

## Feature Status

### ✅ Stable Features (Production Ready)
- **Core Chat API**: `ExLLM.chat/3` - Basic chat functionality with all providers
- **Streaming API**: `ExLLM.stream/4` - Real-time response streaming
- **Session Management**: `ExLLM.Core.Session` - Conversation persistence and state
- **Provider Support**: All major providers (OpenAI, Anthropic, Gemini, Ollama, etc.)
- **Function Calling**: Tool use and function calling across providers
- **Cost Tracking**: Token usage and cost calculation
- **Authentication**: API key management and OAuth2 (Gemini)
- **Configuration**: YAML-based model and provider configuration
- **Error Handling**: Comprehensive error handling and recovery
- **Test Caching**: Advanced response caching for 25x faster integration tests

### 🚧 Incomplete Features (Under Development)
- **Context Management**: `ExLLM.Core.Context.truncate_messages/5` - Automatic message truncation for token limits
- **Model Capabilities API**: `ExLLM.Infrastructure.Config.ModelConfig.get_model_config/1` - Programmatic model metadata access
- **Configuration Validation**: Runtime configuration validation utilities

### 📋 Testing Status
- **Core Functionality**: 100% tested and working (8/8 tests pass)
- **Comprehensive Suite**: 80% tested and working (12/15 tests pass)
- **Manual Testing**: All user-facing features verified working end-to-end

**Note**: The incomplete features are advanced/internal APIs that don't affect normal usage. All user-facing functionality in the example app works perfectly.

## Commands

### Development
```bash
# Run the application
iex -S mix

# Install dependencies
mix deps.get
mix deps.compile
```

### Testing

ExLLM includes a comprehensive testing system with intelligent caching and 24 specialized Mix aliases.

#### Basic Testing
```bash
# Run all tests (fast - uses cache when available)
mix test

# Run a specific test file
mix test test/ex_llm_test.exs

# Run tests at a specific line
mix test test/ex_llm_test.exs:42

# Run tests with coverage
mix test --cover
```

#### Streamlined Testing (12 Mix Aliases)
```bash
# === CORE TESTING STRATEGY ===
mix test.fast           # Fast development tests (excludes API calls and slow tests)
mix test.unit           # Unit tests only (pure logic, no external dependencies)  
mix test.integration    # Integration tests (live APIs by default, requires API keys)
mix test.live           # Force live API tests (bypasses cache, refreshes if enabled)
mix test.all            # All tests including slow/comprehensive suites

# === PROVIDER-SPECIFIC TESTING ===
mix test.anthropic      # Anthropic Claude tests
mix test.openai         # OpenAI GPT tests
mix test.gemini         # Google Gemini tests  
mix test.local          # Local providers (Ollama, LM Studio)
mix test.bumblebee      # Bumblebee tests (requires opt-in due to large downloads)

# === SPECIALIZED TESTING ===
mix test.oauth2         # OAuth2 authentication tests
mix test.ci             # CI/CD pipeline tests (excludes flaky/slow)

# === CACHE MANAGEMENT ===
mix cache.clear         # Clear test cache
mix cache.status        # Check cache status
```

#### Test Caching (25x Speed Improvement)

**Integration tests run against live APIs by default.** Test caching is disabled by default and must be explicitly enabled.

```bash
# DEFAULT BEHAVIOR: Integration tests hit live APIs
mix test --include integration     # Calls live provider APIs

# ENABLE CACHING: Use cached responses when available
export EX_LLM_TEST_CACHE_ENABLED=true
mix test --include integration     # Uses cache if fresh, otherwise excludes tests

# FORCE LIVE: Always use live APIs regardless of cache settings  
MIX_RUN_LIVE=true mix test --include integration

# Manage test cache
mix ex_llm.cache stats
mix ex_llm.cache clean --older-than 7d
mix ex_llm.cache clear
mix ex_llm.cache show anthropic

# Enable cache debugging
export EX_LLM_LOG_LEVEL=debug
```

#### Running Tests with Tags
```bash
# Include specific test types
mix test --include live_api
mix test --include requires_api_key
mix test --include oauth2

# Run only specific tags
mix test --only provider:anthropic
mix test --only streaming
mix test --only integration

# Exclude problematic tests
mix test --exclude requires_service
mix test --exclude slow
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azmaveth/ex_llm](https://github.com/azmaveth/ex_llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
