---
trigger: always_on
description: - Always add debug logs & comments in the code for easier debug & readability
---

# Important rules you HAVE TO FOLLOW
- Always add debug logs & comments in the code for easier debug & readability
- Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase
- Do not make any changes, until you have 95% confidence that you know what to build. Ask me follow up questions until you have that confidence.
- NEVER use localStorage or sessionStorage in client code - store all state in memory or send to server
- This project will be built module by module which. Means. You must make sure you the modules are compatible with each other. When a player in a running game session is gonna play A card It doesn't matter whether it's human player or computer player or An online player from Somewhere else. It's always gonna call the same function and the card will be played.

# Project structure - START SIMPLE, EXPAND LATER
```
dig-card-game/                    # Your game "Dig"
├── server.js                     # WEEK 1: Single file server
├── package.json                  # WEEK 1: Dependencies
├── public/                       # WEEK 1: Client files
│   ├── index.html               # Lobby page
│   ├── game.html                # Game page
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js              # Client logic
│   ├── lang/                    # WEEK 1: Language files
│   │   ├── en.json              # English translations
│   │   ├── zh.json              # Chinese translations
│   │   └── i18n.js              # Language switching logic
│   ├── assets/                  # WEEK 2-3: Game assets
│   │   ├── sounds/              # Sound effects and music
│   │   │ 
│   │   ├── cards/               # Card images
│   │   │   └── standard-deck/   # 52 card images
│   │   └── avatars/             # AI character portraits
│   │       ├── beginner/        # Easy AI avatars
│   │       ├── intermediate/    # Medium AI avatars
│   │       └── expert/          # Hard AI avatars
└── models/                      # WEEK 2-3: Add when needed
    ├── Card.js                  # Basic card representation
    ├── Deck.js                  # Card deck management
    ├── Player.js                # Base player class
    ├── HumanPlayer.js           # Human player logic
    ├── ComputerPlayer.js        # Computer player logic
    ├── DigGameRules.js               # Your "Dig" game rules
    ├── AIEngine.js              # AI computing and probability calculations
    ├── AICharacters.js          # AI personality and difficulty definitions
    └── AudioManager.js          # Sound effects and music management
    └── Other necessary classes  # add more .js when you see fit

# ADD LATER (Week 4+) - Only when you need them:
# ├── routes/           # When you need organized API endpoints
# ├── socket/           # When you add Socket.IO for multiplayer  
# ├── database/         # When you need data persistence
# ├── utils/            # When you have repeated code to organize
# ├── config/           # When you need environment management
# └── tests/            # When you're ready to add testing
```

# Tech Stack
- **Backend**: Node.js + Express.js for HTTP server
- **Real-time**: Socket.IO for multiplayer communication
- **Database**: SQLite (local) → PostgreSQL (production on Railway)
- **Frontend**: Vanilla JavaScript (no frameworks initially)
- **Deployment**: Railway for hosting
- **Environment Management**: dotenv for configuration

# Architecture Patterns

## 1. Game Logic Architecture
- **Player Inheritance**: Base `Player` class with `HumanPlayer` and `ComputerPlayer` subclasses
- **Flexible Player Count**: Game engine supports 2-4 players, configurable per game type
- **Modular Game Rules**: Base `Game` class extended by `DigGame` for your specific rules
- **AI Engine**: Separate class handles card counting, probability calculations, and AI decision making
- **Session-Based**: No persistent user data - everything tied to temporary room codes
- **Event-Driven**: All game actions trigger events that update all connected clients
- **Separation of Concerns**: Game logic separate from network communication
- **Sessions**: You will see me use the word session a lot. This a session is a simulator of A real-world Table where players can come to this table and play Many many games and a session ends When the players are gone.


## 2. Client-Server Communication
- **HTTP Routes**: Minimal - just room creation/joining, no user authentication
- **WebSocket Events**: Real-time game actions (play card, chat, game updates, probability updates)
- **Room-Based Sessions**: Players join via room codes, no accounts needed
- **State Synchronization**: Server is source of truth, clients reflect server state
- **Auto-Cleanup**: Rooms automatically deleted when empty

## 3. Database Design
```sql
-- Minimal tables - no user accounts, session-based only
active_game_rooms (
  id, 
  room_code, 
  max_players, 
  current_players, 
  game_type,
  created_at, 
  last_activity,
  game_state_snapshot -- for reconnection handling
)
-- No persistent user data - everything discarded when room empty
```

# Node.js/JavaScript Specific Rules

## 0. General Coding
- Always prefer simple, readable solutions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Heloise28) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
