---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Conversation Export Toolkit - extracts, analyzes, and indexes conversations from ChatGPT, Claude, and Google Gemini into searchable markdown files with YAML frontmatter and optional Obsidian vault generation.

## Commands

```bash
# =============================================================================
# UNIFIED IMPORT CLI (RECOMMENDED)
# =============================================================================
# Main entry point for all imports - orchestrates extraction, logging, and post-processing

# Full import from all sources (auto-detects default paths)
python3 scripts/run_import.py --all

# Incremental import (only new conversations, skip existing)
python3 scripts/run_import.py --all --incremental

# Import specific sources only
python3 scripts/run_import.py --chatgpt ChatGPT/conversations.json
python3 scripts/run_import.py --claude Claude/conversations.json
python3 scripts/run_import.py --gemini "Google/path/to/MyActivity.json"

# Full import with post-processing
python3 scripts/run_import.py --all --research-index --memories

# Preview what would happen (no changes made)
python3 scripts/run_import.py --all --dry-run

# Add notes to import session
python3 scripts/run_import.py --all --notes "Monthly backup import"

# =============================================================================
# POST-PROCESSING
# =============================================================================

# Build unified research index (scans output/*/conversations/)
python3 scripts/research_index.py

# Extract Claude memories to markdown
python3 scripts/memories_to_md.py

# Migrate old inline metadata to YAML frontmatter (idempotent)
python3 scripts/migrate_to_yaml.py

# Identify and rename Gemini files with non-descriptive titles
python3 scripts/gemini_meta_indexer.py              # Dry-run: preview changes
python3 scripts/gemini_meta_indexer.py --rename      # Apply renames
python3 scripts/gemini_meta_indexer.py --rename --json  # Also save JSON manifest

# Quality filter: score conversations and move low-value ones to output/_excluded/
python3 scripts/quality_filter.py --dry-run          # Preview scoring (no moves)
python3 scripts/quality_filter.py                    # Apply filter (moves excluded files)
python3 scripts/quality_filter.py --verbose          # Show per-file scoring details
python3 scripts/quality_filter.py --threshold 50     # Custom score threshold (default: 40)

# Biography extraction: extract biographical facts from conversations using AI
uv run scripts/biography_extractor.py --all-sources            # All sources (Claude backend)
uv run scripts/biography_extractor.py --all-sources --sample 20 # Test with 20 files
uv run scripts/biography_extractor.py --all-sources --export output/profile/data/extractions.json

# Biography extraction v2: multi-model backend support
uv run scripts/biography_extractor_v2.py --all-sources --provider claude
uv run scripts/biography_extractor_v2.py --all-sources --provider gemini --model gemini-2.0-flash
uv run scripts/biography_extractor_v2.py --all-sources --provider codex

# Profile generator: aggregate biography extractions into organized markdown
python3 scripts/profile_generator.py                          # From default extractions.json
python3 scripts/profile_generator.py --input output/profile/data/extractions.json
python3 scripts/profile_generator.py --output-dir output/profile

# =============================================================================
# VAULT BUILDING
# =============================================================================

# Build Obsidian vault for any date range (flexible successor to build_december_vault.py)
python3 scripts/build_vault.py --all                          # All conversations
python3 scripts/build_vault.py --month 2026-01                # Specific month
python3 scripts/build_vault.py --from 2025-12-01 --to 2025-12-31
python3 scripts/build_vault.py --from 2026-01-01              # From date to now
python3 scripts/build_vault.py --name "my-vault"              # Custom vault name

# =============================================================================
# INDIVIDUAL EXTRACTORS (for manual/fine-grained control)
# =============================================================================

# Extract ChatGPT or Claude conversations (auto-detects format)
python3 scripts/simple_extractor.py ChatGPT/conversations.json -o output/chatgpt-full
python3 scripts/simple_extractor.py Claude/conversations.json -o output/claude-full

# Extract Gemini activity (groups into sessions by 30-min gaps)
python3 scripts/gemini_extractor.py "Google/Η δραστηριότητά μου/Εφαρμογές Gemini/MyActivity.json" -o output/gemini
python3 scripts/gemini_extractor.py path/to/MyActivity.json -o output/gemini --no-grouping

# AI-powered analysis with caching (requires Claude CLI)
uv run scripts/conversation_summarizer.py ChatGPT/conversations.json --output-dir output/chatgpt-full --max 50
uv run scripts/conversation_summarizer.py path/to/conversations.json --no-cache

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/silver-gr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
