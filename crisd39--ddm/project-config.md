---
trigger: always_on
description: Qt6 C++17 backend server (DDM - Display Data Manager) for a naval radar tactical display system. Processes commands from QML frontend, binary data from hardware concentrator (DCL), and interactive console commands. Uses UDP/LocalIPC for transport.
---

# DesformatConcentrator - AI Coding Agent Instructions

## Project Overview
Qt6 C++17 backend server (DDM - Display Data Manager) for a naval radar tactical display system. Processes commands from QML frontend, binary data from hardware concentrator (DCL), and interactive console commands. Uses UDP/LocalIPC for transport.

## Architecture - The Big Picture

### Three Input Sources, One State
```
Frontend (QML) ──JSON via ITransport──┐
Console (stdin) ──CommandDispatcher───┼──► CommandContext (shared state)
DCL Hardware ────Binary via ITransport┘     • std::deque<CursorEntity> cursors
                                             • std::deque<Track> tracks
                                             • double centerX, centerY
```

**Critical insight**: `CommandContext` (in `src/model/commandContext.h`) is the single source of truth. All controllers modify it, encoder reads from it. Thread-safe via Qt event loop - no mutexes needed.

### Message Routing Pattern
`MessageRouter` (`src/controller/messagerouter.cpp`) is the entry point for ALL network messages:
- Starts with `{` → `JsonCommandHandler` (frontend commands)
- High 0xFF density → `DclConcController` (binary hardware data)

**Key architectural decision**: Detection is O(1) for JSON (90% of traffic), so routing adds negligible overhead.

### JSON Command Processing Pipeline
```
JsonCommandHandler::processJsonCommand()
  ├─► Parse JSON → extract "command" + "args"
  ├─► Route via m_commandMap (QMap<QString, std::function>)
  │    └─► Example: "create_line" → LineCommandHandler::createLine()
  ├─► Handler validates (JsonValidator), executes, returns QByteArray
  └─► Send response via ITransport::send()
```

**Pattern to extend**: Add to `JsonCommandHandler::initializeCommandMap()`:
```cpp
m_commandMap["new_command"] = [this](const QJsonObject& args) {
    return m_newHandler->executeCommand(args);
};
```

### Console Command Pattern
Separate from JSON pipeline. Uses Command Pattern with `ICommand` interface:
```cpp
class MyCommand : public ICommand {
    QString getName() const override { return "mycommand"; }
    CommandResult execute(const CommandInvocation& inv, CommandContext& ctx) const override {
        // Direct manipulation of ctx
        ctx.emplaceCursorFront(...);
        ctx.out << "Success\n";
        return {true, ""};
    }
};
// Register in main.cpp:
registry->registerCommand(QSharedPointer<ICommand>(new MyCommand()));
```

## Critical Data Types & Conventions

### qfloat16 for Network Precision
`CursorEntity` uses `QPair<qfloat16, qfloat16>` for coordinates and `qfloat16` for angles/lengths. **Why**: Matches hardware protocol precision (16-bit floats reduce bandwidth). Convert from double→qfloat16 on input, maintain as qfloat16 in entities.

### std::deque for Entity Storage
`CommandContext` uses `std::deque<CursorEntity>` and `std::deque<Track>`, not `QList`. **Why**: Front insertion is O(1) (`emplaceCursorFront`), and no reallocation invalidates iterators during iteration.

### QSharedPointer for Commands
`CommandRegistry` stores `QSharedPointer<ICommand>`, not raw pointers. Qt's shared pointer with signal/slot integration.

### Header-Only Classes
`CommandRegistry` is header-only (in `.h` file). Pattern used for simple template-like classes.

## Transport Layer (ITransport Abstraction)

Two implementations switchable via `Configuration::instance().useLocalIpc`:
- **LocalIPC** (`LocalIpcClient`): QLocalSocket, for same-machine frontend (default: `useLocalIpc = true`)
- **UDP** (`UdpClientAdapter` wrapping `clientSocket`): For network-distributed systems

**Factory pattern**: `makeTransport(TransportKind::Udp/LocalIpc, opts, parent)` in `transportFactory.cpp`.

**Key**: Controllers depend on `ITransport*` interface, never concrete types. Signals: `messageReceived(QByteArray)`, methods: `send(QByteArray)`, `start()`.

## JSON Response Architecture (SOLID Applied)

### Separation of Concerns
- **JsonValidator** (`src/controller/json/validators/`): Validates input fields, returns `ValidationResult` struct
- **LineCommandHandler**: Business logic (create/delete cursors)
- **JsonSerializer**: Entity → JSON (`serializeLine(CursorEntity)`)
- **JsonResponseBuilder**: Constructs standardized responses (`buildSuccessResponse()`, `buildErrorResponse()`)

**Never** mix validation/serialization/response building in handler. Each class has one job.

### Standard Response Format
```json
{
  "status": "success" | "error",
  "command": "create_line",
  "args": { /* command-specific data */ }
}
```

Error responses include `error_code`, `message`, optional `details` object. See `JsonResponseBuilder` methods.

## Binary Protocol (DCL Concentrator)

`ConcDecoder` (`src/model/decoders/concDecoder.cpp`) decodes packed binary from hardware. Emits signals like:
- `newOverlay(QString)` - overlay mode changed
- `newQEK(QString)` - QEK button state
- `newRange(double)` - range value
- `newRollingBall(QPair<int,int>)` - trackball deltas

**Encoding**: `encoderLPD` (`lpdEncoder.h/cpp`) builds outgoing binary messages from `CommandContext` every 40ms (timer in `main.cpp`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CrisD39) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
