---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**LifeOS** is a personal GTD (Getting Things Done) system with Todoist integration. The system enables natural language task creation, automated project management, and comprehensive life tracking.

---

## Quick Commands

```bash
# Todoist Task Management
./lifeos 'your task description'    # Natural language task creation
./lifeos setup                       # Setup Todoist API Token
./lifeos test                        # Test Todoist connection
./lifeos setup-goals                 # Quick setup for fitness/career/English goals
./lifeos fitness                     # Send workout plan
./lifeos export-tasks                # Export all tasks to JSON
./lifeos list-tasks                  # List all tasks

# Anki Sync (Notion → Anki)
./lifeos setup-anki [PAGE_ID]        # Create Anki Cards database in Notion
./lifeos sync-anki                   # Sync Notion to Anki (.apkg)
./lifeos sync-anki --dry-run         # Test sync without changes (safe mode)

# Eudic Sync (欧路词典 → Notion → Anki)
./lifeos sync-eudic                  # Sync Eudic vocabulary to Notion
./lifeos test-eudic                  # Test Eudic API connection (dry-run)
./lifeos sync-eudic --limit 10       # Test with limited number of words

# Life Tracking (Logseq Integration)
./lifeos today                       # Initialize today's journal
./lifeos log work 'content' 8 '2h'  # Log activity with rating and duration
./lifeos report                      # Generate weekly report
./lifeos sync                        # Git sync Logseq

# AI Advisor
./lifeos analyze                     # Analyze life patterns from historical data
./lifeos plan                        # Generate optimized plans based on analysis
```

---

## Core Components

### 1. Todoist Manager (`scripts/todoist_manager.py`)
Full-featured Todoist API integration:
- Create/manage tasks, projects, and labels
- Batch task operations
- Data export for analysis
- Pre-configured fitness/career/English templates

**Key Features:**
- Direct REST API integration (no email required)
- Cross-platform sync (Windows, Mac, iPhone, Android)
- Full programmatic control via Python
- Habit tracking and recurring tasks

### 2. Personal Assistant (`scripts/personal_assistant.py`)
Natural language task processing:
- Parse user input into structured tasks
- Auto-categorize by project and priority
- Estimate task duration
- Send to Todoist via API

### 3. Goals Setup (`scripts/setup_goals.py`)
Quick initialization for three major goals:
- **Fitness (work-out)**: 6 workout tasks
- **Career (job-hunt)**: 8 job search tasks
- **English (speak-up)**: 10 learning tasks

### 4. Logseq Tracker (`scripts/logseq_tracker.py`)
Daily journaling and life tracking:
- Template-based journal initialization
- Activity logging with ratings
- Weekly report generation
- Git synchronization

### 5. AI Advisor (`scripts/ai_advisor.py`)
Pattern analysis and recommendations:
- Analyze historical data from Logseq
- Generate optimization suggestions
- Create personalized plans

### 6. Anki Sync Manager (`scripts/sync_notion_anki.py`)
Notion to Anki synchronization system:
- Query unsynced cards from Notion "Anki Cards" database
- Generate .apkg files using genanki
- Send to Telegram via Bot API
- Update Notion sync status automatically
- Support for multiple decks and tags
- Prevent duplicate cards with stable GUIDs

**Key Features:**
- Uses Notion API 2025-09-03 (latest version)
- GitHub Actions automation (daily at 8am Beijing time)
- Dry-run mode for testing
- Cross-platform Anki import (mobile & desktop)

### 7. Eudic Sync Manager (`scripts/sync_eudic_notion.py`)
Eudic (欧路词典) vocabulary synchronization to Notion:
- Fetch vocabulary from Eudic API automatically
- Parse word, phonetics, and definitions
- Add to Notion "Anki Cards" database
- Track sync state to avoid duplicates
- Batch and incremental sync support

**Key Features:**
- Official Eudic OpenAPI integration
- Auto-tags: "欧路", "vocabulary"
- Deck: Vocabulary (customizable)
- Supports --limit flag for testing
- GitHub Actions automation (runs before Anki sync)
- Format: Front (word) + Back (phonetic + definition)

---

## Configuration Files

- `config/todoist_config.json` - Todoist API settings and project mappings
- `config/assistant_profile.json` - Assistant behavior configuration
- `config/logseq_templates.json` - Logseq journal templates
- `config/anki_sync_config.json` - Anki sync behavior and deck settings
- `config/eudic_config.json` - Eudic (欧路词典) API token and sync settings
- `notion-kit/.env` - Notion and Telegram credentials

### Todoist Project Mapping

```json
{
  "fitness": "work-out",    // Workout and fitness
  "career": "job-hunt",     // Job searching
  "english": "speak-up",    // English learning
  "errands": "Errands",     // Daily errands and life tasks (shopping, cleaning, etc.)
  "inbox": "Inbox"          // Default inbox for uncategorized tasks
}
```

### Labels

- `urgent` - Urgent tasks (red)
- `important` - Important tasks (orange)
- `routine` - Daily routines (grey)
- `habit` - Habit formation (green)

---

## Project Structure

```
LifeOS/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huangf06) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
