---
trigger: always_on
description: Apple Music integration via AppleScript, UI automation, or MusicKit API
---


# Apple Music Integration

Guide for integrating with Apple Music. Three approaches: AppleScript (direct control), UI automation (catalog without API), and MusicKit API (cross-platform).

## When to Use

Invoke when users ask to:
- Manage playlists (create, add/remove tracks, list)
- Control playback (play, pause, skip, volume)
- Play an Apple Music URL (album, playlist, or song)
- Search catalog or library
- Add songs to library
- Access listening history or recommendations

## Critical Rule: Library-First Workflow

**You CANNOT add catalog songs directly to playlists.**

Songs must be in the user's library first:
- ❌ Catalog ID → Playlist (fails)
- ✅ Catalog ID → Library → Playlist (works)

**Why:** Playlists use library IDs (`i.abc123`), not catalog IDs (`1234567890`).

This applies to both AppleScript and API approaches.

---

# AppleScript (macOS)

Zero setup. Works immediately with the Music app.

**Run via Bash:**
```bash
osascript -e 'tell application "Music" to playpause'
osascript -e 'tell application "Music" to return name of current track'
```

**Multi-line scripts:**
```bash
osascript <<'EOF'
tell application "Music"
    set t to current track
    return {name of t, artist of t}
end tell
EOF
```

## Available Operations

| Category | Operations |
|----------|------------|
| **Playback** | play, pause, stop, resume, next track, previous track, fast forward, rewind, play URL |
| **Player State** | player position, player state, sound volume, mute, shuffle enabled/mode, song repeat |
| **Current Track** | name, artist, album, duration, time, rating, loved, disliked, genre, year, track number |
| **Library** | search, list tracks, get track properties, set ratings |
| **Playlists** | list, create, delete, rename, add tracks, remove tracks, get tracks |
| **AirPlay** | list devices, select device, current device |

## Track Properties (Read)

```applescript
tell application "Music"
    set t to current track
    -- Basic info
    name of t           -- "Hey Jude"
    artist of t         -- "The Beatles"
    album of t          -- "1 (Remastered)"
    album artist of t   -- "The Beatles"
    composer of t       -- "Lennon-McCartney"
    genre of t          -- "Rock"
    year of t           -- 1968

    -- Timing
    duration of t       -- 431.0 (seconds)
    time of t           -- "7:11" (formatted)
    start of t          -- start time in seconds
    finish of t         -- end time in seconds

    -- Track info
    track number of t   -- 21
    track count of t    -- 27
    disc number of t    -- 1
    disc count of t     -- 1

    -- Ratings
    rating of t         -- 0-100 (20 per star)
    loved of t          -- true/false
    disliked of t       -- true/false

    -- Playback
    played count of t   -- 42
    played date of t    -- date last played
    skipped count of t  -- 3
    skipped date of t   -- date last skipped

    -- IDs
    persistent ID of t  -- "ABC123DEF456"
    database ID of t    -- 12345
end tell
```

## Track Properties (Writable)

```applescript
tell application "Music"
    set t to current track
    set rating of t to 80          -- 4 stars
    set loved of t to true
    set disliked of t to false
    set name of t to "New Name"    -- rename track
    set genre of t to "Alternative"
    set year of t to 1995
end tell
```

## Player State Properties

```applescript
tell application "Music"
    player state          -- stopped, playing, paused, fast forwarding, rewinding
    player position       -- current position in seconds (read/write)
    sound volume          -- 0-100 (read/write)
    mute                  -- true/false (read/write)
    shuffle enabled       -- true/false (read/write)
    shuffle mode          -- songs, albums, groupings
    song repeat           -- off, one, all (read/write)
    current track         -- track object
    current playlist      -- playlist object
    current stream URL    -- URL if streaming
end tell
```

## Playback Commands

```applescript
tell application "Music"
    -- Play controls
    play                          -- play current selection
    pause
    stop
    resume
    playpause                     -- toggle play/pause
    next track
    previous track
    fast forward
    rewind

    -- Play specific content
    play (first track of library playlist 1 whose name contains "Hey Jude")
    play user playlist "Road Trip"

    -- Settings
    set player position to 60     -- seek to 1:00
    set sound volume to 50        -- 0-100
    set mute to true
    set shuffle enabled to true
    set song repeat to all        -- off, one, all
end tell
```

### Matching titles with typographic punctuation / accents

`whose name contains "..."` is **glyph-exact** — unlike Music's native `search`
command, it does *not* fold a straight apostrophe (U+0027) against the curly one
(U+2019) Music often stores, nor accents. So `whose name contains "That's a No
No"` misses a title stored as `That's a No No`. Two robust strategies:

```applescript
-- 1. Match the quote-free fragments instead of the apostrophe itself.
--    Works regardless of which apostrophe variant is stored or typed.
play (first track of library playlist 1 whose ¬
    (name contains "That" and name contains "s a No No"))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [777Timo/applemusic-mcp](https://github.com/777Timo/applemusic-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
