---
trigger: always_on
description: This file provides guidance to LLMs working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLMs working with code in this repository.

## Project Overview

Libro is a terminal-based reading tracker written in Python that stores data in a local SQLite database. It is a pure CLI tool, interactive input uses `prompt_toolkit` for field prompts with validation and autocomplete. Output is formatted with `rich` Python library.

## Development Commands

### Running

```bash
# Run the application
uv run libro              # Show books read this year
uv run libro add          # Add a book + review
uv run libro report       # Same as default
uv run libro --db ./libro.db report  # Use specific database file
```

### Using Justfile

```bash
just run          # Run the app
just test         # Run pytest tests
just lint         # Check with ruff
just lint-fix     # Fix lint issues
just type-check   # Run mypy
just ci           # Run all checks (lint, type-check, test)
just install      # Install dependencies with uv sync
just dev-install  # Install dev dependencies
just build        # Build the package
```

### Testing and Quality

```bash
uv run python -m pytest tests/ -v
uv run python -m ruff check src/libro/
uv run python -m mypy --package libro
```

## Code Architecture

### Project Structure

```
libro/
├── src/libro/
│   ├── main.py           # Entry point, command routing via match/case
│   ├── config.py         # Argument parsing and database location logic
│   ├── models.py         # Dataclasses: Book, Review, BookReview, ReadingList, ReadingListBook
│   └── actions/
│       ├── db.py         # Database init and migration
│       ├── show.py       # Display functions (rich tables)
│       ├── report.py     # Report views (author stats, yearly charts)
│       ├── modify.py     # Add/edit books and reviews (prompt_toolkit input)
│       ├── lists.py      # Reading list management
│       └── importer.py   # CSV import (Goodreads format and generic CSV)
└── tests/
```

### Key Technologies

- **CLI Parsing**: argparse with subparsers
- **Interactive Input**: prompt_toolkit (PromptSession with validators and completers)
- **Terminal Output**: rich (tables, progress bars, styled text)
- **Database**: SQLite3 (stdlib)
- **Package Manager**: uv

## Data Model

### Books vs Reviews

The application separates **books** from **reviews** to support:
- Books without reviews (unread books on a reading list)
- Multiple reviews per book (re-reads)

**Tables:**
1. `books` - Book metadata (title, author, pub_year, pages, genre)
2. `reviews` - Reading records (book_id, date_read, rating, review)
3. `reading_lists` - Curated book collections
4. `reading_list_books` - Many-to-many join table (with priority and added_date)

**Models** (all dataclasses in `models.py`):
- `Book` - Single book entity with `insert()`, `get_by_id()`, `find_by_title_author()`
- `Review` - Single review entity with `insert()`
- `BookReview` - Combined book + review, most commonly used for display; has `get_by_id()`
- `ReadingList` - List entity with full CRUD methods
- `ReadingListBook` - Association with `get_books_in_list()`, `get_list_stats()`

### Database Schema

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    pub_year INTEGER,
    pages INTEGER,
    genre TEXT
);

CREATE TABLE reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    book_id INTEGER,
    date_read DATE,
    rating INTEGER,
    review TEXT,
    FOREIGN KEY(book_id) REFERENCES books(id)
);

CREATE TABLE reading_lists (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    created_date DATE DEFAULT CURRENT_DATE
);

CREATE TABLE reading_list_books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    list_id INTEGER NOT NULL,
    book_id INTEGER NOT NULL,
    added_date DATE DEFAULT CURRENT_DATE,
    priority INTEGER DEFAULT 0,
    FOREIGN KEY(list_id) REFERENCES reading_lists(id) ON DELETE CASCADE,
    FOREIGN KEY(book_id) REFERENCES books(id) ON DELETE CASCADE,
    UNIQUE(list_id, book_id)
);
```

## Application Flow

### Entry Point

`main.py` calls `init_args()` from `config.py`, connects to the database, runs `migrate_db()`, then routes to the correct action via `match args["command"]`.

### Database Location Priority

1. `--db` command-line flag
2. `libro.db` in current directory
3. `LIBRO_DB` environment variable
4. Platform-specific data directory (via `appdirs`):
   - Linux: `~/.local/share/Libro/mkaz/libro.db`
   - macOS: `~/Library/Application Support/Libro/mkaz/libro.db`
   - Windows: `%APPDATA%\mkaz\Libro\libro.db`

## CLI Commands

```bash
# Default - show books read this year
libro
libro report

# Year chart of books read
libro report --chart

# Author stats (all authors with counts)
libro report --author
# Books by specific author
libro report --author "Stephen King"

# Show book/review detail by review ID
libro report <id>

# Filter by year
libro report --year 2024

# Add book + review (interactive prompts)
libro add

# Book management
libro book                    # Show recent books
libro book <id>               # Show specific book
libro book add                # Add book only (no review)
libro book edit <id>          # Edit book details

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkaz/libro](https://github.com/mkaz/libro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
