---
trigger: always_on
description: A **local-first mobile app** for tracking books you own, books you read, favorite quotes, page references, notes, ratings, and your own recorded audio clips.
---

# Claude.md

## Project: Personal Reading Vault
A **local-first mobile app** for tracking books you own, books you read, favorite quotes, page references, notes, ratings, and your own recorded audio clips.

This app is **for personal use only**.  
It does **not** sync to a server, does **not** use cloud storage, and does **not** require user accounts.

---

# 1. Product Vision

Build a private phone app that feels like a personal digital library:

- A front page that looks like a **bookshelf / library grid** of covers
- Tap a cover to open the book
- Inside each book, save:
  - quotes
  - page numbers
  - optional chapter references
  - notes
  - your own recorded audio clips
  - reading dates
  - rating
- Let the app later suggest **what book to read next from your own library**

This is not a social app, not a marketplace, and not a cloud product.  
It is a **personal reading journal + quote vault + TBR recommender**.

---

# 2. Non-Negotiable Product Constraints

## 2.1 Local-first only
All data stays on-device unless export/import is explicitly added later.

### This means:
- no Supabase
- no Firebase
- no cloud database
- no remote auth
- no remote file storage
- no server APIs required for core usage

## 2.2 Personal use
The app is intended for one user on one phone.

## 2.3 Manual content entry
The app should not depend on external services such as:
- Audible clip importing
- Kindle syncing
- Goodreads syncing
- Apple Books syncing

If those are ever explored, they must be optional and treated as future experiments, not core architecture.

## 2.4 Privacy-first
Since the app contains personal reading history, notes, and audio recordings:
- data should be stored locally
- audio files should remain local
- app should function fully offline
- no analytics required in MVP

---

# 3. Recommended Technical Direction

## 3.1 App type
A **mobile app** for iPhone and Android, with primary use on phone.

## 3.2 Recommended stack
Use **React Native with Expo** for the app shell and UI.

### Why
- fast to build
- excellent for phone-first UI
- easy local media handling
- easier iteration than native Swift/Kotlin for a solo project
- supports local storage and file access

## 3.3 Local persistence
Recommended local storage strategy:

- **SQLite** for structured app data
- **local file storage** for book covers and recorded audio
- lightweight key-value storage for preferences if needed

### Suggested libraries
- `expo-sqlite` or a SQLite wrapper
- `expo-file-system`
- `expo-audio` or current Expo audio recording/playback support
- `expo-image-picker` for cover import
- `expo-router` for navigation
- optional ORM/query layer:
  - Drizzle ORM with SQLite, or
  - direct SQL with a clean repository layer

## 3.4 Architecture style
Use a simple local-first architecture:

- UI layer
- feature/state layer
- repository/data layer
- SQLite database
- local media storage

Avoid overengineering. No microservices nonsense. It is a phone app, not a doomed corporate platform.

---

# 4. Core Product Model

The main objects are:

- **Book**
- **Moment**
- **Tag**
- **Reading Session** (future-friendly if rereads matter)

For MVP, the app can start with `Book` and `Moment`, then add more structured reading history later.

---

# 5. Core User Experience

## 5.1 Library screen
The home screen shows the user's books as a visual shelf/grid of covers.

### Goals
- feel like browsing a personal library
- make books easy to open
- give immediate visual context

### Each book card should show:
- cover
- title
- author
- optional status badge
- optional small indicators:
  - number of moments
  - rating
  - finished date

## 5.2 Book detail screen
When a user taps a book cover, they open the book page.

### Book page includes:
- large cover
- title
- author
- reading status
- date started
- date finished
- rating
- action buttons:
  - add moment
  - record audio
  - edit book

### Content sections
Recommended tabs:
- All
- Quotes
- Audio
- Favorites

## 5.3 Add moment flow
A moment is the core saved item.

A moment may contain:
- quote text
- page number
- chapter
- note
- favorite flag
- optional recorded audio clip
- optional tags

This is better than treating quote and audio as unrelated objects. The real thing being saved is a **moment from the book**.

---

# 6. Local Data Model

## 6.1 Books table

```sql
CREATE TABLE books (
  id TEXT PRIMARY KEY NOT NULL,
  title TEXT NOT NULL,
  author TEXT NOT NULL,
  cover_path TEXT,
  format TEXT,
  status TEXT NOT NULL DEFAULT 'not_started',
  date_started TEXT,
  date_finished TEXT,
  rating REAL,
  priority INTEGER DEFAULT 0,
  genre TEXT,
  description TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);
```

### Notes
- `cover_path` is a local file path, not a remote URL
- `rating` can support half-stars, e.g. `4.5`
- `status` values:
  - `not_started`
  - `currently_reading`
  - `finished`
  - `paused`
  - `abandoned`

## 6.2 Moments table

```sql
CREATE TABLE moments (
  id TEXT PRIMARY KEY NOT NULL,
  book_id TEXT NOT NULL,
  quote_text TEXT,
  page_number INTEGER,
  chapter TEXT,
  note_text TEXT,
  audio_path TEXT,
  audio_duration_seconds INTEGER,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marSanGal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
