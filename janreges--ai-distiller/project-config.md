---
trigger: always_on
description: AI Distiller is a high-performance CLI tool that extracts essential code structure from large codebases, making them digestible for LLMs by removing unnecessary details while preserving semantic information. Think of it as "code compression for AI context windows."
---

# AI Distiller - Claude Development Instructions

## Project Overview

AI Distiller is a high-performance CLI tool that extracts essential code structure from large codebases, making them digestible for LLMs by removing unnecessary details while preserving semantic information. Think of it as "code compression for AI context windows."

## Key Project Goals

1. **Single native binary** - No runtime dependencies, works everywhere
2. **Blazingly fast** - Process 10MB codebase in <2 seconds
3. **Multi-language support** - 15+ languages via tree-sitter WASM
4. **Flexible output** - Text, Markdown, JSON, JSONL, XML formats
5. **Granular control** - Strip exactly what you don't need

## CLI Interface Specification

The main CLI is `aid` (AI Distiller):

```bash
aid [path] [flags]

# Examples:
aid                                    # Process current directory
aid src/                              # Process src directory
aid main.py                           # Process single file
aid --comments=0,implementation   # Remove comments and implementations
aid --format json-structured --output api.json   # JSON output to file
aid --private=0 --protected=0 --internal=0 --stdout       # Print only public members to stdout

# Special git mode (activated when path is .git):
aid .git                              # Show full git history
aid .git --git-limit=50              # Show last 50 commits
aid .git --with-analysis-prompt      # Include AI analysis prompt for insights

# AI-powered analysis modes:
aid --ai-action=flow-for-deep-file-to-file-analysis     # Generate comprehensive task list
aid --ai-action=flow-for-multi-file-docs                # Generate documentation workflow
aid --ai-action=prompt-for-refactoring-suggestion       # Generate refactoring analysis prompt
aid --ai-action=prompt-for-complex-codebase-analysis    # Full codebase analysis with diagrams
aid --ai-action=prompt-for-security-analysis            # Security-focused analysis prompt
aid --ai-action=prompt-for-performance-analysis         # Performance optimization analysis
aid --ai-action=prompt-for-best-practices-analysis      # Best practices and code quality analysis
aid --ai-action=prompt-for-bug-hunting                  # Systematic bug hunting analysis
aid --ai-action=prompt-for-single-file-docs             # Single file documentation analysis
aid --ai-action=prompt-for-diagrams                     # Generate 10 beneficial Mermaid diagrams

# Legacy mode (deprecated):
aid --ai-analysis-task-list                    # Use --ai-action=flow-for-deep-file-to-file-analysis instead
```

### Important Flags

**NEW: Individual Filtering Flags (Recommended)**

The new flag system provides precise control over what to include:

**Visibility Flags** (control which members to show based on access level):
- `--public=0/1` (default: 1) - Include public members
- `--protected=0/1` (default: 0) - Include protected members
- `--internal=0/1` (default: 0) - Include internal/package-private members
- `--private=0/1` (default: 0) - Include private members

**Content Flags** (control what content to include):
- `--comments=0/1` (default: 0) - Include comments
- `--docstrings=0/1` (default: 1) - Include documentation comments
- `--implementation=0/1` (default: 0) - Include function/method bodies
- `--imports=0/1` (default: 1) - Include import statements
- `--annotations=0/1` (default: 1) - Include decorators/annotations
- `--fields=0/1` (default: 1) - Include class fields and properties
- `--methods=0/1` (default: 1) - Include methods and functions

**Group Filtering** (alternative syntax):
- `--include-only=public,protected,imports` - Include only these categories
- `--exclude-items=private,comments` - Exclude these categories

**File Pattern Filtering** (NEW - supports multiple syntaxes):
- `--include "*.go,*.py,*.ts"` - Comma-separated patterns
- `--include "*.go" --include "*.py"` - Multiple flags  
- `--exclude "*test*,*spec*,*.json"` - Exclude patterns
- `--exclude "*test*" --exclude "*.json"` - Multiple exclusions

**Examples:**
```bash
# Default: public APIs only
aid src/

# Include all visibility levels
aid src/ --public=1 --protected=1 --internal=1 --private=1

# Include implementation details
aid src/ --implementation=1

# Exclude comments but include everything else
aid src/ --exclude-items=comments

# Only public and protected members with imports
aid src/ --include-only=public,protected,imports

# Extract only method signatures (no fields/properties) - great for large codebases  
aid src/ --fields=0 --implementation=0

# Extract only data structures (no method noise)
aid models/ --methods=0

# Focus on public API methods only
aid services/ --fields=0 --private=0 --protected=0 --internal=0

# Performance control
aid large-project/ --workers=1        # Single-threaded for debugging
aid huge-codebase/ --workers=8        # Use 8 parallel workers
aid . --recursive=0                   # Only current directory, no subdirs
```

**Legacy Flag (Deprecated):**
- `--strip <items>` - Still works but deprecated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janreges/ai-distiller](https://github.com/janreges/ai-distiller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
