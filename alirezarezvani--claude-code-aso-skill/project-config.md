---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **Claude Code skill repository** containing the "app-store-optimization" skill. This skill provides comprehensive ASO capabilities for mobile app developers and marketers to optimize app presence on Apple App Store and Google Play Store.

## Skill Installation

### User-Level Installation (Available in All Projects)
```bash
cp -r app-store-optimization ~/.claude/skills/
```

### Project-Level Installation
```bash
cp -r app-store-optimization /path/to/project/.claude/skills/
```

### Verification
```bash
ls ~/.claude/skills/app-store-optimization/
# Should show: SKILL.md, 8 Python modules, sample files, documentation
```

## Architecture

### Core Modules (8 Python Scripts)

**keyword_analyzer.py** (13KB)
- Analyzes keywords for search volume, competition, relevance
- Key: `analyze_keyword()`, `compare_keywords()`, `find_long_tail_opportunities()`

**metadata_optimizer.py** (20KB)
- Optimizes titles, descriptions, keyword fields with platform-specific limits
- Key: `optimize_title()`, `optimize_description()`, `validate_character_limits()`

**competitor_analyzer.py** (21KB)
- Analyzes competitor ASO strategies, identifies gaps
- Key: `analyze_competitor()`, `compare_competitors()`, `identify_gaps()`

**aso_scorer.py** (19KB)
- Calculates 0-100 ASO health score across metadata, ratings, keywords, conversion
- Key: `calculate_overall_score()`, `generate_recommendations()`

**ab_test_planner.py** (23KB)
- Plans and tracks A/B tests with statistical significance
- Key: `design_test()`, `calculate_sample_size()`, `calculate_significance()`

**localization_helper.py** (22KB)
- Manages multi-language optimization, ROI analysis
- Key: `identify_target_markets()`, `translate_metadata()`, `calculate_localization_roi()`

**review_analyzer.py** (26KB)
- Analyzes user reviews for sentiment, issues, feature requests
- Key: `analyze_sentiment()`, `extract_common_themes()`, `generate_response_templates()`

**launch_checklist.py** (29KB)
- Generates pre-launch checklists, compliance validation, timing optimization
- Key: `generate_prelaunch_checklist()`, `optimize_launch_timing()`

### Platform Constraints (Critical)

**Apple App Store:**
- Title: 30 chars
- Subtitle: 30 chars
- Promotional Text: 170 chars
- Description: 4000 chars
- Keywords: 100 chars (comma-separated, no spaces)

**Google Play Store:**
- Title: 50 chars
- Short Description: 80 chars
- Full Description: 4000 chars
- No keyword field (extracted from title/description)

### Data Flow

Input (JSON) → Python Module → Analysis → Output (JSON)
- `sample_input.json`: Example request structure
- `expected_output.json`: Example response with keyword analysis, metadata recommendations

## Working with This Repository

### DO NOT Modify
- Python modules are production-ready (no external dependencies, Python 3.7+)
- SKILL.md is the skill specification file
- Character limit validation is critical for platform compliance

### Development Tasks
When asked to improve or extend this skill:

1. **Adding New Functions**: Add to existing modules maintaining consistency
2. **New Modules**: Create new .py file in `app-store-optimization/` with similar structure
3. **Testing**: Verify against platform character limits (Apple: 30/30/170/100/4000, Google: 50/80/4000)
4. **Documentation**: Update SKILL.md, README.md, HOW_TO_USE.md

### Testing Changes
```bash
cd app-store-optimization
python3 keyword_analyzer.py  # Basic syntax check
python3 metadata_optimizer.py
# Test character limit validation with sample data
```

## Common Use Cases

**Keyword Research:**
User provides app name, category, features → `keyword_analyzer.py` → prioritized keyword list

**Metadata Optimization:**
User provides app info → `metadata_optimizer.py` → platform-specific titles/descriptions with character validation

**ASO Health Check:**
User provides metrics (ratings, keyword rankings, conversion) → `aso_scorer.py` → 0-100 score + recommendations

**Pre-Launch:**
User provides launch date → `launch_checklist.py` → comprehensive checklist for both stores

## Critical Design Principles

1. **Zero External Dependencies**: Standard library only for portability
2. **Platform Compliance**: Always validate character limits
3. **Dual Platform**: Support both Apple App Store and Google Play Store
4. **Actionable Outputs**: Provide specific recommendations, not generic advice
5. **Statistical Rigor**: A/B testing requires proper sample size and significance calculations

## Key Differentiators

- **AI Keyword Focus**: Skill emphasizes finding "AI prioritization" opportunities (low competition, high relevance)
- **Long-Tail Strategy**: Identifies 3-4 word phrases with lower competition
- **Character Limit Enforcement**: Prevents non-compliant metadata submissions
- **No Paid Acquisition**: Focuses exclusively on organic ASO (no Apple Search Ads, Google Ads)

## Version Info

**Current Version**: 1.0.0 (November 7, 2025)
**Platform Requirements**: Current as of November 2025
**Python**: 3.7+
**Target Users**: App developers, app marketers, indie developers, ASO specialists

---

## ASO Agent System (NEW)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alirezarezvani/claude-code-aso-skill](https://github.com/alirezarezvani/claude-code-aso-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
