---
trigger: always_on
description: This app is a Node.js/Express application with a React frontend that monitors changes to files containing chat logs from various agents (Claude Code, OpenCode, etc.). It processes these logs via parser configs to generate messages for text-to-speech playback.
---

# agent-tts

This app is a Node.js/Express application with a React frontend that monitors changes to files containing chat logs from various agents (Claude Code, OpenCode, etc.). It processes these logs via parser configs to generate messages for text-to-speech playback.

## Architecture

The application runs as a unified service on a single port (default: 3456), serving both the API and the frontend. It consists of:

- **Backend**: Express server with WebSocket support for real-time updates
- **Frontend**: React SPA with Tailwind CSS for styling
- **Database**: SQLite for persistent storage of logs, file tracking, and favorites
- **TTS Services**: Multiple providers (ElevenLabs, OpenAI, Kokoro) for text-to-speech generation
- **Audio Player**: Dedicated service for audio playback with proper process management

## Configuration

Configuration files are JavaScript/TypeScript files with default exports. Users can extend configurations by importing other config files and using spread operators.

Default configuration location: `~/.config/agent-tts/config.{js,ts}`

Configuration features:

- Hot-reload support with file watching
- Error reporting for syntax issues
- TypeScript support via `ts-blank-space` for type erasure
- Profile-based configuration for different agents

## File Locations (XDG Base Directory Specification)

The application follows the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/latest/) for organizing user files:

- **Configuration**: `~/.config/agent-tts/` (or `$XDG_CONFIG_HOME/agent-tts/`)
  - `config.{js,ts}` - Main configuration file
  - `images/` - Custom avatar images
- **State/Database**: `~/.local/state/agent-tts/` (or `$XDG_STATE_HOME/agent-tts/`)
  - `agent-tts.db` - SQLite database
  - `logs/` - Application logs
  - `backups/` - Database backups
- **Cache**: `~/.cache/agent-tts/` (or `$XDG_CACHE_HOME/agent-tts/`)
  - `audio/YYYY-MM-DD/` - Generated audio files

## Database

Data is stored in a SQLite database. The default location: `~/.local/state/agent-tts/agent-tts.db`. Use the `sqlite3` CLI tool to manipulate the database.

Here is the schema:

### file_states

Tracks the last file size of appendable logs like Claude Code

- filepath TEXT PRIMARY KEY,
- last_modified INTEGER NOT NULL,
- file_size INTEGER NOT NULL,
- last_processed_offset INTEGER NOT NULL,
- updated_at INTEGER DEFAULT (strftime('%s', 'now')),
- profile TEXT DEFAULT 'default'

### tts_queue

Stores user and assistant messages, including state

- id INTEGER PRIMARY KEY AUTOINCREMENT,
- timestamp INTEGER NOT NULL,
- filename TEXT NOT NULL,
- profile TEXT NOT NULL,
- original_text TEXT NOT NULL,
- filtered_text TEXT NOT NULL,
- state TEXT CHECK (state IN ('queued', 'playing', 'played', 'error', 'user')) NOT NULL,
- api_response_status INTEGER,
- api_response_message TEXT,
- processing_time INTEGER,
- created_at INTEGER DEFAULT (strftime('%s', 'now')),
- is_favorite INTEGER DEFAULT 0,
- cwd TEXT,
- role TEXT CHECK (role IN ('user', 'assistant'))

## File Monitoring

The system monitors specified log files for changes:

1. On startup, reads all files specified in watch config
2. Maintains last modified date and file size in SQLite
3. When changes detected, reads new content from last offset
4. Sends content to profile-specific parser
5. Processes messages through filter chain
6. Queues filtered text for TTS playback

Features:

- One database row per monitored file
- Queue-based processing (one change at a time)
- Offset-based reading for efficiency

## Text-to-Speech

TTS implementation with multiple providers:

- **Providers**: ElevenLabs, OpenAI, and Kokoro (local) support
- **Audio Storage**: Permanent audio files saved to `~/.cache/agent-tts/audio/YYYY-MM-DD/profile-timestamp.mp3`
- **Audio Serving**: Audio files served via `/audio` route for browser and remote playback
- **Audio Replay**: Cached audio files are reused when replaying messages
- **Playback Modes**:
  - **Server-side**: Automatic playback on local machine when new messages arrive
  - **Browser-side**: Manual playback in web UI using HTML5 Audio API
  - **Remote**: Audio URLs accessible for mobile apps and remote clients
- **AudioPlayer Service**: Centralized audio playback with proper process management
- **Stoppable Playback**: Works for both new TTS generation and replayed audio files
- **Queue-based Processing**: Prevents audio overlap with proper queue management
- **Database Logging**: All TTS entries tracked with:
  - Timestamp and profile
  - Original and filtered text
  - Status (queued, playing, played, error)
  - Favorite status for memorable moments
  - API response details
  - Processing time
  - Audio URL for playback

## API Endpoints

- `POST /api/tts/stop` - Stop current playback
- `POST /api/tts/pause` - Pause playback
- `POST /api/tts/resume` - Resume playback
- `POST /api/tts/skip` - Skip current item
- `GET /api/profiles` - Get all profiles
- `PUT /api/profiles/:id` - Enable/disable profile
- `GET /api/logs` - Get log entries with pagination support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiliman/agent-tts](https://github.com/kiliman/agent-tts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
