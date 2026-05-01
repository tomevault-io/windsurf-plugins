---
trigger: always_on
description: A standalone C++20 / wxWidgets feed reader, successor to Emacs Elfeed.
---

# Notes for AI agents working on Elfeed2

A standalone C++20 / wxWidgets feed reader, successor to Emacs Elfeed.
Single-binary GUI app, no plugins. Scope is intentionally a bit
broader than classic Elfeed — built-in podcast / yt-dlp downloads
manager, inline image cache, etc.

The maintainer is Chris Wellons (skeeto, author of classic Elfeed and
dcmake). Concrete prose > corporate prose. He notices when a comment
explains the *what* instead of the *why*.

## Build & run

```sh
cmake -B build
cmake --build build --target elfeed2
build/elfeed2.app/Contents/MacOS/elfeed2          # macOS
build/elfeed2.exe                                 # Windows
build/elfeed2                                     # Linux
```

CLI options: `--db PATH` and `--config PATH` (see README). Use them
when stress-testing so the production database stays untouched. The
single-instance lock is per-DB so test instances run alongside the
real one.

`-DDEPS=LOCAL` switches from FetchContent-pinned versions to system
libraries (for distro packagers). `cmake/Toolchain-Mingw64.cmake`
cross-builds a self-contained Windows .exe.

`tests/fakefeeds.py` serves N synthetic Atom feeds on localhost with
configurable latency and failure injection. Use this for any
"generate lots of fetch / log / download activity" testing — the
maintainer reads real feeds with this app and isn't going to slam
real publishers for testing.

## Source layout

- **`src/app.cpp` / `src/app.hpp`** — wxApp entry, OnInit, single-
  instance check, CLI parse, elfeed_init / shutdown lifecycle.
- **`src/elfeed.hpp`** — central Elfeed state struct + all public
  function declarations. Deliberately does NOT include wx headers
  (only forward declares wx pointer types) so non-UI modules don't
  pull in the wx world. Anything wx-flavored stored here uses
  primitives (uint32_t color, std::string for paths, etc).
- **`src/main_frame.{hpp,cpp}`** — the wxFrame + wxAUI orchestrator.
  Owns all the panels.
- **`src/entry_list.{hpp,cpp}`** — virtual-list wxDataViewCtrl for
  entries. Includes the TabularTextRenderer for the date column.
- **`src/entry_detail.{hpp,cpp}`** — wxHtmlWindow preview pane.
- **`src/feeds_panel.{hpp,cpp}`** — feeds list with right-click
  context menu and `↳` indicator for canonical-URL redirects.
- **`src/log_panel.{hpp,cpp}`** — log table with filter checkboxes,
  Copy/Export/Clear context menu.
- **`src/downloads_panel.{hpp,cpp}`** — download queue with
  wxDataViewProgressRenderer per row.
- **`src/db.cpp`** — all SQLite. Schema lives in `schema_sql` at top.
- **`src/feed.cpp`** — pugixml parser for Atom/RSS/RDF.
- **`src/filter.cpp`** — filter DSL (`+tag -tag @age =feed ~feed
  !title #limit bareTitle`). `=` and `~` are case-insensitive
  substring (not regex — explicit choice).
- **`src/fetch.cpp`** — fetch worker pool, posts results to inbox.
- **`src/download.cpp`** — download manager. Subprocess (yt-dlp /
  curl via wxProcess) AND HTTP-direct (cpp-httplib via std::thread)
  paths.
- **`src/image_cache.{hpp,cpp}`** — preview pane image cache. SQLite
  blob storage + LRU eviction; URLs in entry HTML rewritten to
  `data:` URIs.
- **`src/data_uri_handler.{hpp,cpp}`** — wxFileSystemHandler for
  `data:` URIs (wx 3.2 has none built in).
- **`src/http.hpp`** + `http_posix.cpp` / `http_win.cpp` — HTTP
  client. cpp-httplib + mbedTLS (or OpenSSL with `-DDEPS=LOCAL`)
  on POSIX; WinHTTP on Windows.
- **`src/util.{hpp,cpp}`** — paths, time formatting, filename
  sanitization, MIME helpers, wxDataView column persistence + sort
  helpers.
- **`src/config.cpp`** — line-oriented `ssh_config`-style config
  parser.
- **`src/elfeed_import.cpp`** — one-way importer for classic
  Elfeed's `index` s-expression DB.

## Architecture notes

### Threading model

UI thread owns the database, all wxWidgets state, and `app->log` /
`app->feeds` / `app->entries` / `app->downloads` / etc. Workers
push to **inbox vectors** under per-resource mutexes, then call
`app_wake_ui(app)` which posts a wxThreadEvent. `MainFrame::on_wake`
drains inboxes (fetch results, image cache results) and triggers
appropriate refreshes.

The pattern: workers don't touch DB or UI directly. They produce
data, the UI thread consumes it.

Notable exception: image-cache writes were per-wake (one BEGIN/INSERT
/COMMIT per cell update), now coalesced via timer. Log persistence
is similarly **timer-coalesced (5s) to avoid one-fsync-per-wake**;
the same pattern is worth applying anywhere you find yourself
reaching for synchronous DB writes from a high-frequency callback.

### Database

- Tables: `feed`, `entry`, `entry_tag`, `entry_author`,
  `entry_enclosure`, `ui_state`, `image_cache`, `log_entry`.
- Author / enclosure are **relational** (not JSON blobs). Tags too.
- Tag preservation across refetch: the upsert path checks
  `EXISTS(...)` first; if the entry already exists, no tag inserts
  happen. Matches classic Elfeed's merge semantics. Without this
  precheck every refetch silently restores `unread`.
- **Date is NOT updated on refetch.** The parser falls back to
  `time(now)` for entries without a published date; if we updated
  on every refetch, undated entries would perpetually bubble to the
  top. First-sight wins.
- `ui_state` is the kitchen-sink key/value table for everything

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skeeto/elfeed2](https://github.com/skeeto/elfeed2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
