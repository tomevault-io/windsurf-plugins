---
trigger: always_on
description: **Tien Len TCP v3** is a networked card game application using JavaFX for the client UI and a TCP socket-based server. Tien Len is a Vietnamese card game. This is a full-stack multiplayer game with user authentication, real-time game sessions, and betting mechanics.
---

# Copilot Instructions for Tien Len TCP v3

## Project Overview
**Tien Len TCP v3** is a networked card game application using JavaFX for the client UI and a TCP socket-based server. Tien Len is a Vietnamese card game. This is a full-stack multiplayer game with user authentication, real-time game sessions, and betting mechanics.

- **Java Version**: 17
- **UI Framework**: JavaFX 21.0.3
- **Build Tool**: Maven
- **Database**: MySQL (connector included)
- **Port**: 12345 (server)

## Architecture

### Core Components

#### 1. **Server** (`src/main/java/tienlen/server/`)
- **ServerBackend**: Main server loop, manages connections, player registrations, and game sessions
- **GameSession**: Orchestrates single game instance - handles turns, move validation, betting, and game state
- **ClientHandler**: Handles individual client connections - receives messages, delegates to GameSession
- **UserManager**: Manages user authentication and account balances (MySQL integration)
- **PlayerInfo**: In-memory representation of online player state

**Key Pattern**: Thread-per-client model. Each client gets a `ClientHandler` running on its own thread, with synchronized access to shared `GameSession` state.

#### 2. **Client** (`src/main/java/tienlen/client/`)
- **TienLenClientUI**: JavaFX UI components - player hand, played cards area, player list, chat
- **ClientFX** or **ClientMain**: Entry point, establishes socket connection, spawns listener threads
- **ServerListener**: Reads incoming messages from server in a separate thread
- **AuthListener**: Handles login/registration flows

**Key Pattern**: Listener threads update UI via `Platform.runLater()` for thread-safety.

#### 3. **Game Logic** (`src/main/java/tienlen/logic/`)
- **GameLogic**: Card combo identification (single, pair, triple, four-of-a-kind, straights, three pairs, etc.)
- Validates move legality against last played move

#### 4. **Model** (`src/main/java/tienlen/model/`)
- **Card**: Rank (2=15, 3-10, J=11, Q=12, K=13, A=14) and Suit enums; implements `Comparable`
- **Deck**: Shuffled deck of 52 cards
- **Player**: Game state per player (hand, status, current session)
- **Message**: Simple action|data protocol format
- **Move**: Represents a card play with `ComboType` enum and validation
- **GameState**: Snapshot of game state (players, hands, turn, last move)

## Communication Protocol

**Format**: `ACTION|DATA` (pipe-delimited, see [Protocol.java](src/main/java/tienlen/utils/Protocol.java))

### Key Actions
- **Auth**: `LOGIN`, `LOGIN_SUCCESS`, `LOGIN_FAILED`, `REGISTER`, `REGISTER_SUCCESS`, `REGISTER_FAILED`
- **Session**: `GET_SESSIONS`, `SESSION_LIST`, `SELECT_SESSION`, `SESSION_INFO`, `JOIN`
- **Game**: `GAME_START`, `PLAY`, `PASS`, `HAND`, `PLAYER_LIST`, `NEWGAME`
- **Chat**: `CHAT`
- **Money**: `BET`, `BET_RESULT`, `BALANCE`, `PLAYER_INFO`, `GAME_RESULT`, `INSUFFICIENT_BALANCE`

### Encoding/Decoding
```java
// Encoding: Message -> String
Protocol.encode(new Message("PLAY", "3♥,5♦,7♠"))

// Decoding: String -> Message
Message msg = Protocol.decode("PLAY|3♥,5♦,7♠");
```

## Game Flow & State Management

1. **Initialization**: Player joins session → GameSession fills to 4 players → deals 13 cards each
2. **Turn Cycle**: 
   - Current player can: PLAY (valid cards), PASS
   - Move validated against `lastMove` via `GameLogic.identifyType()`
   - If 3+ players pass consecutively → reset `lastMove` (anyone can play)
3. **Win Condition**: One player empties hand first
4. **Betting**: Fixed `betAmount` per game; winner takes pot (stored in `GameSession.totalPot`)
5. **Balance**: MySQL `UserManager` tracks account balances

## Critical Patterns & Conventions

### Thread Safety
- `GameSession` uses synchronized methods for state mutations (addToPot, setGameRunning, etc.)
- `ServerBackend` uses `CopyOnWriteArrayList` and `Collections.synchronizedMap()`
- Client UI updates always via `Platform.runLater()` to avoid JavaFX thread exceptions

### Card Representation
- **String format**: `"3♥"`, `"10♣"`, `"A♠"`, `"2♦"` (suit symbols: ♠♥♦♣)
- **Rank values**: 2=15 (highest), 3-10=face value, J=11, Q=12, K=13, A=14
- **Comparable**: Cards sorted by rank; suit is secondary

### Message Routing
1. `ServerListener` (client) or `ClientHandler` (server) reads raw string from socket
2. `Protocol.decode()` → `Message` object
3. Handler processes `message.getAction()` and routes based on Protocol constants
4. Response sent via `Protocol.encode()` + `PrintWriter.println()`

### UI Threading (JavaFX)
```java
// WRONG: Direct UI update in server listener thread
chatArea.appendText(text);

// RIGHT: Defer to FX thread
Platform.runLater(() -> chatArea.appendText(text));
```

## Build & Run

### Maven
```bash
# Compile & package
mvn clean package

# Run client
mvn javafx:run -DmainClass=tienlen.client.ClientFX

# Run server
mvn exec:java -Dexec.mainClass=tienlen.server.ServerMain
```

### Bash Script
See [build.sh](build.sh) - sets up JavaFX module path (adjust SDK path as needed)

## Database Integration

- **UserManager.getInstance()**: Singleton pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nhatgi0210) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
