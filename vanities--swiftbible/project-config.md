---
trigger: always_on
description: > Universal agent instructions. `CLAUDE.md` is a compatibility symlink for Claude Code.
---

# AGENTS.md - SwiftBible Repository Documentation

> Universal agent instructions. `CLAUDE.md` is a compatibility symlink for Claude Code.
## Repository Overview

SwiftBible is an iOS application written in Swift that provides access to biblical texts, apocrypha, and pseudepigraphic literature. The app supports multiple text collections with rich navigation, search, and summary features.

## Project Structure

### Core iOS App (`/swiftbible/`)
```
swiftbible/
├── Models/
│   ├── Testament.swift           # Enums for text categories (old, new, apocrypha, enoch)
│   └── Version.swift             # Enum for Bible translations (kjv, asv, web)
├── Services/
│   └── BibleService.swift        # Data fetching services with caching per version
├── ViewModels/
│   └── AppViewModel.swift        # Global state including selectedVersion
├── Views/Bible/
│   └── BibleView.swift          # Main UI for browsing texts
├── Text/
│   ├── bible.json               # KJV Old/New Testament data
│   ├── asv.json                 # ASV Old/New Testament data
│   ├── web.json                 # WEB Old/New Testament data
│   ├── apocrypha.json           # Deuterocanonical books
│   ├── enoch.json               # Book of Enoch (5 sections)
│   ├── summaries.swift          # Detailed verse summaries
│   └── ChapterSummaries.swift   # Brief chapter summaries
```

### Python Parsers (`/python_parser/`)
```
python_parser/
├── parse_kjv.py                 # KJV Bible parser (from kjv.txt)
├── parse_asv.py                 # ASV parser (from Zefania XML)
├── parse_web.py                 # WEB parser (from USFX XML)
├── parse_apocrypha.py           # Apocrypha parser
├── parse_book_of_enoch.py       # Book of Enoch parser
├── book_of_enoch.txt            # Raw Enoch text
├── book_of_enoch_info.txt       # Scholarly information about Enoch
└── generate_verse_info.py       # Verse metadata generator
```

## Bible Translations

The app supports multiple Bible translations, all public domain:

### Supported Versions
| Version | Full Name | Year | Characteristics |
|---------|-----------|------|-----------------|
| KJV | King James Version | 1611 | Classic, formal language |
| ASV | American Standard Version | 1901 | Highly literal, scholarly |
| WEB | World English Bible | 2000 | Modern English, based on ASV |

### Version Implementation
```swift
enum Version: String, Codable, CaseIterable {
    case kjv
    case asv
    case web

    var displayName: String { ... }
    var shortName: String { ... }
    var filename: String { ... }  // Maps to JSON file
}
```

### Version Switching
- User selects version in Settings
- `AppViewModel.selectedVersion` stores the current version
- Views use computed properties that derive data from `BibleService.fetchBook(version:)`
- Changes propagate automatically via SwiftUI's @Observable tracking

## Data Architecture

### JSON Structure
All texts follow a consistent JSON structure:
```json
[
  {
    "name": "Book Name",
    "description": "Book description",
    "chapters": [
      {
        "number": 1,
        "paragraphs": [
          {
            "startingVerse": 1,
            "text": "Verse text with inline references like 1:2..."
          }
        ]
      }
    ]
  }
]
```

### Testament Categories
- `old` - Old Testament books
- `new` - New Testament books
- `apocrypha` - Deuterocanonical books
- `enoch` - Book of Enoch sections

## Book of Enoch Integration

### Background
The Book of Enoch is a pseudepigraphic work composed of 5 distinct sections spanning 108 chapters. It was excluded from the biblical canon but provides important historical and theological context.

### The Five Sections
1. **The Book of the Watchers** (Chapters 1-36)
   - Fall of the Watchers (angels) and their offspring (Nephilim)
   - Enoch's heavenly journeys and visions
   - Geography of punishment and blessing

2. **The Book of Parables** (Chapters 37-71)
   - Three parables featuring the "Son of Man"
   - Throne of glory visions
   - Resurrection and final judgment themes

3. **The Astronomical Book** (Chapters 72-82)
   - 364-day solar calendar revealed by angel Uriel
   - Cosmic order and celestial movements
   - Calendar ensuring proper festival timing

4. **The Book of Dream Visions** (Chapters 83-90)
   - Animal Apocalypse - symbolic history from Adam to Maccabees
   - Two major visions: flood and world history

5. **The Epistle of Enoch** (Chapters 91-108)
   - Apocalypse of Weeks (10 periods of history)
   - Woes against sinners, encouragement for righteous
   - Birth of Noah narrative

### Parser Implementation

#### Key Features
- **Sequential chapter numbering**: Maintains original 1-108 numbering across sections
- **Inline verse handling**: Converts "2 living" patterns to "1:2 living" format
- **Text cleaning**: Removes formatting artifacts while preserving references
- **Scholarly organization**: Creates 5 separate books matching academic divisions

#### Parser Logic (`parse_book_of_enoch.py`)
```python
# Section definitions
ENOCH_SECTIONS = [
    {"name": "The Book of the Watchers", "start_chapter": 1, "end_chapter": 36},
    {"name": "The Book of Parables", "start_chapter": 37, "end_chapter": 71},
    # ... etc
]

# Text parsing with inline verse handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vanities/swiftbible](https://github.com/vanities/swiftbible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
