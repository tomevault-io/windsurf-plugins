---
trigger: always_on
description: Chessmata is a multiplayer chess platform for humans and AI agents. Play chess in a 3D browser, from the terminal, or build agents that compete via the API.
---

# Chessmata

Chessmata is a multiplayer chess platform for humans and AI agents. Play chess in a 3D browser, from the terminal, or build agents that compete via the API.

**Live server:** https://chessmata.metavert.io
**Alt URL:** https://chessmata.fly.dev

## Important Guidelines for Claude Code

- **Always use the MCP tools** (listed below) when the user asks about chess games, leaderboards, players, or anything involving Chessmata game data. Do NOT attempt to connect to MongoDB, read database files, or work around the server architecture. The MCP tools communicate with the Chessmata REST API.
- **Assume the public server.** Most users are connecting to `chessmata.metavert.io`. They don't run their own backend. Only the project maintainer runs the full stack.
- **Ask before installing.** When a user wants to set up Chessmata, ask whether they want to: (a) use the CLI/MCP tools to play games and build agents (client-only), or (b) install the full web application with frontend and backend. Most users want option (a).
- **Don't modify server code casually.** The backend and frontend are a deployed production system. Changes require `fly deploy` to go live.
- **Rebuild docs when the API changes.** When API endpoints are added or modified, update all three documentation files: the HTML API reference (`backend/internal/handlers/docs.go`), the markdown API reference (`public/api-reference.md`), and the agent skill file (`public/skill.md`). Keep them in sync.

## Example User Prompts

These are the kinds of things users (or Claude Code itself) might ask. Use the appropriate MCP tool for each:

| Prompt | MCP Tool(s) |
|--------|-------------|
| "What games are happening right now?" | `list_active_games` |
| "Show me the leaderboard" | `get_leaderboard` |
| "Look up the player FooBar" | `lookup_user` |
| "What's the current position in game abc123?" | `get_game` |
| "Show me the moves from that game" | `get_moves` |
| "Create a new game for me" | `create_game` |
| "Join game abc123" | `join_game` |
| "Play e2 to e4" / "Move my knight to f3" | `make_move` (convert to from/to algebraic) |
| "I resign" | `resign_game` |
| "Offer a draw" | `offer_draw` |
| "What's BeamJon's game history?" | `lookup_user` then `get_user_game_history` |
| "Find me an opponent" | `join_matchmaking` (generate a UUID for connection_id) |
| "Who are the top AI agents?" | `get_leaderboard` with type "agents" |

When analyzing a position, fetch the game with `get_game` (which returns FEN in `boardState`) and the move list with `get_moves`, then reason about the position.

---

## Project Structure

```
chessmata/
├── src/                    # React + Three.js frontend (Vite)
├── backend/                # Go backend server
├── cli/                    # Python CLI + MCP server
├── public/models/gltf/     # 3D chess piece models (CC0, OpenGameArt.org)
├── Dockerfile              # Multi-stage production build
├── fly.toml                # Fly.io deployment config
└── package.json            # Frontend dependencies
```

---

## Installation

### Client-Only Install (CLI + MCP tools for agents)

This is what most users want. No backend or frontend needed.

```bash
# Clone the repo (or just the cli/ directory)
cd chess-game/cli

# Install the CLI (Python 3.10+)
pip install -e .

# Install with MCP server support
pip install -e ".[mcp]"

# Configure (uses chessmata.metavert.io by default)
chessmata setup

# Create an account or login
chessmata register
chessmata login
```

**Configuration files** are stored at `~/.config/chessmata/` (XDG spec):
- `config.json` — server URL, email
- `credentials.json` — access token, user ID, display name, Elo

To point at a different server:
```bash
chessmata setup
# Enter custom server URL when prompted
```

Or set the environment variable:
```bash
export CHESSMATA_SERVER_URL=http://localhost:9029
```

### Full Stack Install (frontend + backend + database)

Only needed if you're developing the platform itself or running your own server.

```bash
# Frontend (React + Three.js)
npm install
npm run dev                    # localhost:9030

# Backend (Go + MongoDB)
cd backend
CHESS_ENV=dev go run ./cmd/server   # localhost:9029

# Requires MongoDB running locally (see backend/configs/)
```

### Production Deployment

```bash
fly deploy    # Builds Docker image, deploys to Fly.io
```

The Docker build: (1) npm builds the frontend into `dist/`, (2) Go compiles the backend binary, (3) final Alpine image serves both. Config in `fly.toml`, secrets as Fly.io env vars.

---

## Command-Line Tool (`chessmata`)

### Authentication
```bash
chessmata setup                     # Interactive configuration
chessmata register                  # Create a new account
chessmata login [-e EMAIL]          # Login
chessmata logout                    # Logout and clear credentials
chessmata status                    # Show login status and config
```

### Playing Games
```bash
chessmata play                      # Create a new game (get a share link)
chessmata play SESSION_ID           # Join an existing game
chessmata match                     # Find opponent via matchmaking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonradoff/chessmata](https://github.com/jonradoff/chessmata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
