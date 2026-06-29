---
trigger: always_on
description: Scripts in `scripts/` are loaded automatically at startup.
---

# GTProxy - Lua Scripting Agent Guidelines

## Overview

Scripts in `scripts/` are loaded automatically at startup.
The scripting engine uses sol2 for C++/Lua bindings.

## Global APIs

| API             | Purpose                                   |
|-----------------|-------------------------------------------|
| `logger`        | Logging functions with fmt formatting     |
| `event`         | Event system for packet/connection events |
| `send`          | Send packet structs to client or server   |
| `packet`        | Packet types, enums, and raw send helpers |
| `command`       | Register custom proxy commands            |
| `scheduler`     | Schedule timed tasks and callbacks        |
| `world`         | Access world and player state             |
| `item_database` | Access item database for item lookups     |

## Logger

```lua
logger.info("Message")
logger.debug("Debug info")
logger.warn("Warning message")
logger.error("Error message")

-- Formatting support (fmt-style)
logger.info("There are {} pending tasks", count)
logger.info("Player {} is at ({}, {})", player.name, pos.x, pos.y)
```

## Events

### Registering Event Handlers

```lua
event.on("server:SendToServer", function(ctx)
    if ctx:has_packet() then
        -- Both `ctx:parse()` and `ctx:parse_packet()` work identically
        local pkt = ctx:parse()
        -- Process decoded packet

        -- Modify packet fields directly (no tracking needed)
        if pkt.net_id then
            pkt.net_id = 999
        end

        -- Cancel and resend if modifying
        ctx:cancel()
        send.to_server(pkt)
    else
        local data = ctx:get_data()
        -- Process raw packet bytes
    end
end)
```

### Available Events

| Event                   | Description                    |
|-------------------------|--------------------------------|
| `"client:Connect"`      | Game client connected to proxy |
| `"client:Disconnect"`   | Game client disconnected       |
| `"server:Connect"`      | Proxy connected to game server |
| `"server:Disconnect"`   | Proxy disconnected from server |
| `"client:SendToClient"` | Packet from server to client   |
| `"server:SendToServer"` | Packet from client to server   |

## Command Registration

You can register commands from scripts in two ways. The `description` parameter is optional.

```lua
-- With description (name, description, callback)
command.register("say", "Echo a message", function(ctx)
    if #ctx.args < 1 then
        logger.error("Usage: /say <message>")
        return false
    end
    logger.info(table.concat(ctx.args, " "))
    return true
end)

-- Without description (name, callback) — a default description is used
command.register("echo", function(ctx)
    if #ctx.args < 1 then
        ctx:reply("Usage: /echo <message>")
        return false
    end

    local message = table.concat(ctx.args, " ")
    local log = LogPacket.new()
    log.msg = message
    send.to_client(log)
    return true
end)

-- Command context includes reply() function for direct responses
command.register("greet", "Greet the player", function(ctx)
    ctx:reply("Hello from the proxy!")
    return true
end)
```

### Command Management

```lua
-- List all registered commands
local all_commands = command.list()
for name, description in pairs(all_commands) do
    logger.info("Command: {} - {}", name, description or "No description")
end

-- Execute a command programmatically
command.execute("/say Hello world")

-- Get current command prefix
local prefix = command.get_prefix()
logger.info("Current prefix: {}", prefix)

-- Set new command prefix
command.set_prefix("#")
logger.info("Prefix changed to #")
```

### Command Context

| Field         | Type     | Description                                    |
|---------------|----------|------------------------------------------------|
| `ctx.args`    | table    | Parsed command arguments                       |
| `ctx.raw`     | string   | Raw input string                               |
| `ctx:reply()` | function | Send a message to the player (returns boolean) |

### Command Management

```lua
-- List all registered commands
local commands = command.list()
for name, desc in pairs(commands) do
    logger.info("{}: {}", name, desc)
end

-- Get command prefix
local prefix = command.get_prefix()
logger.info("Command prefix: {}", prefix)

-- Set command prefix
command.set_prefix("!")

-- Execute a command programmatically
local success = command.execute("/say Hello World")
if success then
    logger.info("Command executed successfully")
end
```

## Sending Packets

### Using Packet Structs

```lua
-- Send packet struct to client
local log = LogPacket.new()
log.msg = "Hello from Lua!"
send.to_client(log)

-- Send packet struct to server
local join = JoinRequestPacket.new()
join.world_name = "START"
send.to_server(join)
```

### Using Raw/Text Functions

```lua
-- Send raw bytes (table of integers 0-255)
packet.send_raw({ 0x03, 0x00, 0x00, 0x00 }, true)  -- to server
packet.send_raw({ 0x03, 0x00, 0x00, 0x00 }, false) -- to client

-- Send text packet (uses NET_MESSAGE_GAME_MESSAGE by default)
packet.send_text("action|respawn", true)

-- Send text packet with specific message type
packet.send_text("action|respawn", true, packet.NET_MESSAGE_GENERIC_TEXT)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZTzTopia/GTProxy](https://github.com/ZTzTopia/GTProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
