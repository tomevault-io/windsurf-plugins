---
trigger: always_on
description: **Claude Config Manager (ccm)** is a CLI tool written in Rust that manages multiple Claude Code configuration profiles. It enables users to switch between different AI providers that support Anthropic-compatible APIs and launch Claude Code with specific configurations.
---

# Claude Config Manager (ccm) - Documentation for AI Agents

## Project Overview

**Claude Config Manager (ccm)** is a CLI tool written in Rust that manages multiple Claude Code configuration profiles. It enables users to switch between different AI providers that support Anthropic-compatible APIs and launch Claude Code with specific configurations.

*Note: The README.md documentation mentions several providers (OpenAI, Deepseek, Kimi, GLM, Minimax) as known examples of compatible services, but this tool works with any provider offering Anthropic-compatible API endpoints.*

## Architecture

### Core Components

1. **CLI Interface** (`src/cli.rs`)
   - Uses `clap` derive macros for command parsing
   - 8 main commands: `add`, `list`/`ls`, `show`, `remove`/`rm`, `switch`/`swc`, `run`, `import-current`, `sync`
   - Supports environment variable injection via `--env` flags

2. **Profile Management** (`src/profile.rs`)
   - Interactive profile creation with validation
   - JSON-based profile storage with environment variables
   - Safety mechanisms (prevents removal of active profiles)
   - Current profile tracking
   - **Sync functionality** - compares and synchronizes current profile with active Claude settings

3. **Configuration** (`src/config.rs`)
   - XDG-compliant directory structure
   - Configurable Claude settings path via `CLAUDE_SETTINGS_PATH`
   - Automatic directory creation and path resolution

### File Structure
```
src/
├── main.rs      # Entry point and command routing
├── lib.rs       # Library root
├── cli.rs       # Command-line interface definitions
├── config.rs    # Path and configuration management
└── profile.rs   # Core profile functionality
```

## Key Features

### Profile Management
- **Interactive Creation**: Prompts for required and optional environment variables
- **Environment Variables**: Support for custom variables via `--env KEY=VALUE`
- **Validation**: Input validation and error handling
- **Security**: Prevents accidental deletion of active profiles

### Sync Feature
- **Automatic Synchronization**: `ccm sync` command compares current profile with actual `settings.json`
- **Bidirectional Update**: Updates ccm profile to match Claude settings when they diverge
- **Change Detection**: Uses JSON comparison to detect changes accurately

### Supported Environment Variables
- `ANTHROPIC_BASE_URL` (required) - API endpoint URL
- `ANTHROPIC_AUTH_TOKEN` (required) - Authentication token
- `ANTHROPIC_MODEL` (optional) - Model selection
- `API_TIMEOUT_MS` (optional) - Request timeout
- `ANTHROPIC_SMALL_FAST_MODEL` (optional) - Fast model alternative
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` (optional int) - Traffic control

## File Storage

### Default Paths (XDG-compliant)
- **Profiles Directory**: `~/.config/ccm/profiles/*.json`
- **Current Profile**: `~/.config/ccm/current`
- **Claude Settings**: `~/.claude/settings.json`

### Environment Variable Overrides
For testing or custom configurations, these directories can be overridden:
- **`CCM_CONFIG_DIR`**: Override the base directory for CCM configuration (default: `~/.config/ccm`)
  - Profiles will be stored in `$CCM_CONFIG_DIR/profiles/`
  - Current profile marker will be at `$CCM_CONFIG_DIR/current`
- **`CLAUDE_SETTINGS_PATH`**: Override the Claude settings file path (default: `~/.claude/settings.json`)

### Profile Format
```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.example.com/v1",
    "ANTHROPIC_AUTH_TOKEN": "sk-...",
    "ANTHROPIC_MODEL": "gpt-4",
    "API_TIMEOUT_MS": "300000"
  }
}
```

## Dependencies

### Core Dependencies
- `clap` (4.3) - Command-line parsing with derive macros
- `serde` (1.0) - Serialization framework
- `serde_json` (1.0) - JSON handling
- `dirs` (6.0) - Cross-platform directory access
- `anyhow` (1.0) - Error handling

## Build System

### Development Tools
- **Rust Toolchain**: 1.90.0
- **Task Runner**: `just` for build automation
- **CI/CD**: GitHub Actions with multi-platform builds

### Build System
DO NOT run build command by your own.This project uses [`just`](https://just.systems/) as the task runner for build automation. See the `justfile` for available commands.

## Common Usage Patterns

### Adding Profiles
```bash
# Interactive creation
ccm add my-profile

# With environment variables
ccm add my-profile --env CUSTOM_VAR=value
```

### Managing Profiles
```bash
# List all profiles
ccm list

# Switch to profile
ccm switch my-profile

# Show profile content
ccm show my-profile

# Remove profile
ccm remove my-profile
```

### Launching Claude
```bash
# Switch and run
ccm switch my-profile && ccm run

# Import current settings
ccm import-current backup-profile

# Sync current profile with Claude settings
ccm sync
```

## Extension Points

### Adding New Commands
1. Add command variant to `Commands` enum in `cli.rs`
2. Implement handler function in `profile.rs`
3. Add command routing in `main.rs`
4. Update CLI interface with any aliases in the clap derive macros

### Adding New Environment Variables
1. Update interactive prompts in `add_profile_interactive()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caibirdme/ccm](https://github.com/caibirdme/ccm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
