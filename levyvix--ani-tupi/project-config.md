---
trigger: always_on
description: **Best Practices for writing Python code:**
---

## General Rules and Guidelines

**Best Practices for writing Python code:**

### Design Principles
- Apply the DRY principle - Don't Repeat Yourself
- Prefer composition over inheritance for more maintainable code
- Write pure functions when possible (no side effects, same output for same input)
- Follow SOLID principles for maintainable object-oriented design
- Write tests first (TDD) or alongside code development
- Use dataclasses for data containers

### Handling Complexity
- Hide implementation details behind clean interfaces
- Create abstractions that eliminate complexity for users
- Encapsulate related data and behavior in cohesive classes
- Use interfaces or abstract base classes to define contracts
- Apply dependency injection for more flexible and testable code
- Favor simple solutions over complex or clever ones
- Design for the most common use case first
- Keep component coupling loose through well-defined interfaces

---

## Core Values

**Simplicity First**: Every feature should feel effortless to the user. Complex logic (incremental search, fuzzy matching, automatic syncing) runs invisibly.

**DRY Architecture**: Code is organized by *what it does*, not *where it lives*. If multiple scrapers implement `search()` → they're one pattern (plugin protocol). If multiple services fetch from APIs → they share a base class. If code repeats → extract it.

**Immutable Data Flow**: Data flows forward only. Services don't modify input; they return transformed copies. This makes debugging trivial: follow the data, not the mutations.

**Plugin Everything**: Scrapers, PDF readers, storage backends—all pluggable. New source? Create one file. Done.

**User Configuration Over Code**: Settings live in environment variables (via Pydantic config), not config files that break. Users can tune everything without touching code.

---

## CLI Usage Guide

### Basic Anime Playback

```bash
# Search and select anime interactively
ani-tupi -q "anime name"

# Search and jump to specific episode
ani-tupi -q "anime name" -e 5
```

### Advanced Options

```bash
# Select specific season (for anime with multiple seasons)
ani-tupi -q "anime name" -S 2

# Select specific season and episode
ani-tupi -q "anime name" -S 2 -e 5

# Continue from where you left off
ani-tupi -c

# Continue from specific episode (overrides history)
ani-tupi -c -e 5

# Continue from history but override to different season
ani-tupi -c -S 2

# List available sources
ani-tupi --list-sources

# Clear cache (all or specific anime)
ani-tupi --clear-cache
ani-tupi --clear-cache "anime name"
```

### Season and Episode Usage

- `-S 2` → Seleciona estação 2 (para anime com múltiplas estações)
- `-S 2 -e 5` → Estação 2, episódio 5 (pula menus)
- `-e 5` → Episódio 5 (navegação via menu para próximo/anterior disponível)
- `-e 1` → Início (episódio 1)
- `-e 100` → Erro se > total de episódios disponíveis

**Nota sobre estações**: Se um anime tem apenas uma estação, o menu de seleção é automaticamente pulado.

---

## Architecture Principles

### The Three-Tier System

1. **Commands** (CLI entry points) - Parse user intent
2. **Services** (business logic) - Coordinate plugins, cache, APIs, and persistence
3. **Plugins** (implementations) - Scrapers, readers, storage backends

Services orchestrate. They decide: "Should I search cache first?" "Should I sync to AniList?" "Which plugin should I use?"

Commands ask services questions. Services ask plugins for data. Plugins never ask anything—they're pure adapters.

**To extend**: Add a new feature? Build a service. Add a new data source? Build a plugin. Add a new command? Wire up a service call.

### Pattern: Centralized Configuration

All settings flow through `models/config.py` (Pydantic v2):

```python
from models.config import settings
cache_ttl = settings.cache_duration_hours
reader = settings.manga.pdf_reader
```

Why? Environment variables override defaults (`ANI_TUPI__CACHE__DURATION_HOURS=48`), no scattered `.env` files, type validation on boot, configuration is self-documenting.

### Pattern: Plugin Protocol (Not Inheritance)

Each plugin implements a structural type:

```python
class Scraper(Protocol):
    def search(self, query: str) -> list[AnimeMetadata]: ...
    def get_episodes(self, url: str) -> list[EpisodeData]: ...
```

Why protocol, not ABC?
- Scrapers auto-discover with duck typing
- No base class boilerplate
- Plugin loading is one loop: find `.py` files in `scrapers/plugins/`, import them, extract classes matching the protocol

### Pattern: Repository for Plugin Access

Don't import plugins directly. Use the repository:

```python
from services.repository import get_scrapers

for scraper in get_scrapers():
    results.extend(scraper.search(query))
```

Why? Scrapers are loaded dynamically. The repository tracks which ones exist, which ones are enabled.

### Pattern: Multi-Source Title Normalization

The repository automatically deduplicates anime results from multiple sources using intelligent title normalization. This means:

**Same anime, different title formats are merged:**
```
AnimesDigital: "Anime A: Revolucao Dublado"
AnimeOnlineCC: "Anime A - Revolucao Dublado"
AnimeFireTV:   "Anime A | Revolucao Dublado"

Result: Single entry "anime a revolucao dublado [animesdigital, animesonlinecc, animefiretv]"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levyvix/ani-tupi](https://github.com/levyvix/ani-tupi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
