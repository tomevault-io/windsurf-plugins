---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gitai is a Python CLI tool that uses AI to generate Git commit messages following the Conventional Commits standard. It analyzes project changes across multiple programming languages, automatically detects the project type, and uses OpenAI, Groq, or Anthropic APIs to generate meaningful commit messages.

## Essential Commands

### Development Setup
```bash
# Create virtual environment
./createVirtualEnv.sh

# Install dependencies
pip install -r requirements.txt

# Activate virtual environment
source venv/bin/activate
```

### Building the Application
```bash
# Build executable with PyInstaller
./build.sh
# This creates dist/gitai executable and copies .env.example to dist/.env
```

### Running from Source
```bash
# Main application
python src/gitai/gitai.py <project_path> '<message>' [--push]

# Release notes generator
python src/gitai/releaser.py <old_tag> <new_version>
```

### Testing the Built Executable
```bash
# After running build.sh
dist/gitai . 'test commit message'
dist/gitai . 'test commit message' --push
```

## Architecture

### Core Components

**1. Main Application (`src/gitai/gitai.py`)**
- **CLI Interface**: Accepts project path, base message, and optional --push flag
- **Language Detection**: Identifies programming language by checking indicator files (package.json, requirements.txt, etc.) and file extensions
- **AI Integration**: Dynamically loads OpenAI, Groq, or Anthropic client based on PROVIDER env var
- **Git Operations**: Executes git commands via subprocess with proper error handling

**2. Release Notes Generator (`src/gitai/releaser.py`)**
- Analyzes commits since last tag using `git log`
- Categorizes commits into New Features, Bug Fixes, and Other Changes
- Generates markdown-formatted release notes via AI
- Outputs to `dist/release_<version>.md`

### Key Design Patterns

**Strategy Pattern**: Provider selection (OpenAI/Groq/Anthropic) determined at runtime based on environment configuration.

**Command Pattern**: Git operations encapsulated in `run_git_command()` with consistent error handling.

**Template Method**: Commit message generation follows strict template defined in prompt, ensuring Conventional Commits format.

### Git Workflow
The application follows this sequence:
1. Check for uncommitted changes
2. If changes exist: detect language → generate diff → call AI → commit
3. Execute `git pull` to sync with remote
4. If conflicts after pull: auto-generate conflict resolution commit
5. If `--push` flag: push to remote (only if branch is ahead)

### Colored Console Output
The application uses colorama for cross-platform colored terminal output:
- `print_header()`: Cyan, for headers (🚀)
- `print_success()`: Green, for success messages (✅)
- `print_info()`: Blue, for informational messages (ℹ️)
- `print_warning()`: Yellow, for warnings (⚠️)
- `print_error()`: Red, for errors (❌)
- `print_git_operation()`: Magenta, for git operations (🔄)
- `print_ai_message()`: Cyan, for AI-related messages (🤖)

Each language has an associated emoji displayed during detection.

## Configuration

### Environment Variables (`.env`)
Required variables that must be set:
```bash
PROVIDER=openai|groq|anthropic  # AI provider
API_KEY=<your_api_key>          # Provider's API key
MODEL=<model_name>              # e.g., gpt-4o, mixtral-8x7b-32768, claude-3-5-sonnet-20241022
LANGUAGE=en|pt                  # Output language for commit messages
```

The application loads `.env` from the same directory as the executable (`sys.executable`).

### Supported AI Providers
- **OpenAI**: `gpt-4o` (128K context)
- **Groq**: `mixtral-8x7b-32768` (32K context, free tier with rate limits)
- **Anthropic**: `claude-3-5-sonnet-20241022` (200K context)

All providers use same parameters:
- `temperature=0.5`
- `max_tokens=500` (gitai.py) or `1000` (releaser.py)
- `top_p=1.0`

## Language Detection

The `detect_project_language()` function uses a three-phase approach:

1. **Root Indicator Files**: Checks for language-specific files (package.json, requirements.txt, Cargo.toml, etc.)
2. **File Extensions**: Scans root directory for extension patterns (.csproj, .java, .py, etc.)
3. **Recursive Search**: Walks subdirectories to find language indicators if not found in root

Supported languages include: Node.js, Python, Java, Go, PHP, Ruby, Rust, C#, Swift, Dart, Scala, Haskell, Perl, R, C/C++, JavaScript, TypeScript, Kotlin.

## Conventional Commits Standard

The application enforces ONLY these prefixes:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `chore`: Maintenance or minor fixes that don't alter functionality

**Required Format**:
```
[prefix]: [concise description]

[detailed explanation of changes, reasons, and impact]

🤖 Commit generated with [Gitai](https://github.com/leandrosilvaferreira/gitai)
```

The prompt explicitly instructs AI to use this format with a mandatory empty line after the first line.

## Important Implementation Details

### Git Command Execution
- Uses `subprocess.run()` with `text=True` and `encoding='utf-8'`
- Captures both stdout and stderr

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leandrosilvaferreira/gitai](https://github.com/leandrosilvaferreira/gitai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
