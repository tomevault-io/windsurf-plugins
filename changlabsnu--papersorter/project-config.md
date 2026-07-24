---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PaperSorter is an academic paper recommendation system built in Python that uses machine learning to filter RSS feeds and predict user interest in research articles. The system fetches articles from RSS/Atom feeds, generates embeddings using OpenAI-compatible APIs (like Solar LLM), and uses XGBoost regression to predict user interest levels.

## Architecture

The system consists of several key components:

### Core Modules
- **FeedDatabase** (`feed_database.py`): PostgreSQL-based storage for article metadata, user labels, and predictions
- **EmbeddingDatabase** (`embedding_database.py`): PostgreSQL-based storage for article embedding vectors using pgvector extension
- **FeedPredictor** (`services/feed_prediction.py`): XGBoost model training and prediction logic
- **BroadcastChannels** (`broadcast_channels.py`): Channel management for notifications
- **Data** (`data/`): Database schema and configuration
  - `schema.py`: Python schema definitions for database tables

### Tasks (`tasks/`)
CLI commands implemented as argparse-based commands:
- `init.py`: Initialize database schema
- `update.py`: Fetch new articles, generate embeddings, and queue items for broadcast
- `import.py`: Import articles from external sources (currently supports PubMed)
  - `pubmed` subcommand: Downloads recent PubMed update files and imports with sampling
- `train.py`: Train XGBoost model on labeled data (requires --name for database registration)
- `predict.py`: Generate embeddings and predictions for articles
- `broadcast.py`: Process broadcast queue and send notifications to Slack
- `serve.py`: Entry point for web interface (delegates to web package)
- `test.py`: Test various system components
- `models.py`: Comprehensive model management (list, show, activate, delete, export, import)
- `labeling.py`: Create and manage labeling sessions for training data collection
- `embeddings.py`: Manage embeddings table and HNSW indices (clear, reset, status, index on/off)

### Web Interface (`web/`)
Modular Flask application:
- `app.py`: Flask application factory
- `main.py`: Main route handlers for feed list and labeling
- `wsgi.py`: WSGI entry point for production deployment
- **auth/**: Authentication module with OAuth integration
  - `models.py`: User model for Flask-Login
  - `decorators.py`: Authentication decorators (admin_required)
  - `routes.py`: Login/logout/OAuth callback routes (Google, GitHub, ORCID)
- **api/**: RESTful API endpoints organized by domain
  - `feeds.py`: Feed operations (list, star, feedback, similar articles)
  - `search.py`: Text search, AI summarization, scholarly database integration
  - `settings.py`: Admin settings for channels, users, models, events
  - `user.py`: User preferences and poster generation
- **models/**: Data models
  - `scholarly_article.py`: Abstract base for scholarly articles
  - `semantic_scholar.py`: Semantic Scholar paper representation
- **utils/**: Shared utilities
  - `database.py`: Database helper functions and search query management
- **jobs/**: Background job processors
  - `poster.py`: AI-powered infographic poster generation

### Providers (`providers/`)
Feed and scholarly database provider implementations:
- `base.py`: Abstract base class for feed providers
- `factory.py`: Provider factory for creating provider instances
- `rss.py`: RSS/Atom feed provider
- `scholarly_database.py`: Abstract base for scholarly databases
- `semantic_scholar.py`: Semantic Scholar API integration
- `openalex.py`: OpenAlex API integration

### Utils (`utils/`)
Utility modules:
- `pubmed_sync.py`: PubMed FTP sync and XML parsing functionality
- `broadcast_hours.py`: Broadcast scheduling utilities
- `email.py`: Email notification utilities

### Entry Points
- `__main__.py`: Dynamic CLI command loader that imports all tasks from `tasks/__init__.py`
- `__init__.py`: Package initialization
- `__version__.py`: Version information
- `log.py`: Logging configuration

## Common Commands

### Installation
```bash
pip install -e .
```

### Recommended Initial Setup Workflow (New Users)
```bash
# Stage 1: Initial Model Training (Similarity-based)
# 1. Initialize database
papersorter init

# 2. Import PubMed data with relevant ISSNs (target ~10,000 articles)
papersorter import pubmed --issn 1476-4687 --issn 0036-8075 --files 20

# 3. Generate embeddings for all articles
papersorter predict --all

# 4. Start web interface
papersorter serve --skip-authentication user@example.com

# 5. Use semantic search to find and mark 5-10 diverse papers as "Interested"

# 6. Create labeling session based on similarity
papersorter labeling create --sample-size 200

# 7. Complete labeling session at /labeling

# 8. Train initial model
papersorter train --name "Initial Model v1"

# 9. Generate predictions
papersorter predict

# Stage 2: Model Refinement (Prediction-based) - Highly Recommended
# 10. Create second labeling session based on predictions
papersorter labeling create --base-model 1 --sample-size 1000

# 11. Complete second labeling session

# 12. Train refined model
papersorter train --name "Production Model v1"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChangLabSNU/PaperSorter](https://github.com/ChangLabSNU/PaperSorter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
