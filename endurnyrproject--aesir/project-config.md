---
trigger: always_on
description: This file provides guidelines for agentic coding agents operating in this repository.
---

# AGENTS.md

This file provides guidelines for agentic coding agents operating in this repository.

## The Project

Aesir is a Ragnarok Online Emulator written in Elixir, the objective is to build a stable, professional and well implemented server, following
the good Elixir code practices and correctness with the Ragnarok Online Mechanics.

## Project Structure

The project follows an Elixir umbrella application structure, organized into four main apps:

1. **commons** - Core shared functionality used by all server applications
   - Network handling (TCP sockets via Ranch)
   - Packet definitions and parsing
   - Database models and schemas
   - Authentication system
   - Session management (distributed via Memento)
   - Inter-server communication (via Phoenix.PubSub)

2. **account_server** - Handles user authentication and login
   - Login packet processing
   - Account validation
   - Server selection
   - Client version checking

3. **char_server** - Manages character-related operations
   - Character creation, listing, and selection
   - Character deletion
   - Character data management
   - Slot management
   - Server transfer to zone servers

4. **zone_server** - Implements in-game world functionality
   - Map system and navigation
   - Character movement and positioning
   - Entity management (players, NPCs, monsters)
   - Status effects and combat
   - Scripting system (using Lua)

### Key Architectural Components

- **Umbrella Structure**: Each server is a separate OTP application within an umbrella project
- **Distributed System**: Uses libcluster for node discovery and communication
- **Database**: Uses Ecto with Postgres for persistent storage
- **In-Memory Storage**: Uses Memento (Mnesia wrapper) for distributed session data
- **Inter-Server Communication**: Phoenix.PubSub for messaging between servers
- **Network Layer**: Ranch TCP implementation with custom packet handling
- **Scripting**: Lua integration for game logic and event scripting

## Packet System

Ragnarok Online uses a binary packet-based protocol. Key components:

1. **Packet Base Module**: `Aesir.Commons.Network.Packet` provides the foundation for all packet types
   - Defines common packet structure (2-byte header, variable/fixed size)
   - Provides parsing and building utilities
   - Contains binary encoding/decoding helpers

2. **Packet Registry**: `Aesir.Commons.Network.PacketRegistry` manages packet definitions
   - Maps packet IDs to their corresponding modules
   - Handles packet size information
   - Helps with packet validation

3. **Individual Packet Modules**: Each packet type has its own module that:
   - Defines the packet structure
   - Implements parsing from binary data
   - Implements building back to binary format
   - Documents the packet format and purpose

4. **Connection Handling**: `Aesir.Commons.Network.Connection` manages TCP socket communication
   - Receives and buffers incoming data
   - Parses complete packets
   - Routes packets to appropriate handlers
   - Manages connection state

### Example Packet Implementation Pattern

```elixir
defmodule MyServer.Packets.SomePacket do
  use Aesir.Commons.Network.Packet
  
  @packet_id 0x1234
  @packet_size 24  # or :variable for variable-sized packets
  
  defstruct [:field1, :field2, :field3]
  
  @impl true
  def packet_id, do: @packet_id
  
  @impl true
  def packet_size, do: @packet_size
  
  @impl true
  def parse(<<@packet_id::16-little, data::binary>>), do: parse(data)
  def parse(<<field1::32-little, field2::16-little, field3::binary-size(16)>>) do
    {:ok, %__MODULE__{
      field1: field1,
      field2: field2,
      field3: extract_string(field3)
    }}
  end
  def parse(_), do: {:error, :invalid_packet}
  
  @impl true
  def build(%__MODULE__{} = packet) do
    data = <<packet.field1::32-little, packet.field2::16-little, pack_string(packet.field3, 16)::binary>>
    build_packet(@packet_id, data)
  end
end
```

## Session Management

The session system manages player connections and state across servers:

1. **Session Creation**: Started in the account server during login
2. **Session Validation**: Used by char/zone servers to verify authentication
3. **Distributed Storage**: Uses Memento (Mnesia) for cluster-wide access
4. **Server Tracking**: Monitors which players are on which servers
5. **Heartbeat System**: Detects disconnected nodes and cleans up orphaned sessions

### Key Session Functions

- `SessionManager.create_session/2` - Creates a new session after successful login
- `SessionManager.validate_session/3` - Validates session credentials when changing servers
- `SessionManager.set_user_online/4` - Marks a user as online on a specific server
- `SessionManager.end_session/1` - Cleans up session data when a user logs out

## Database Models

The project uses Ecto for database interactions with two primary models:

1. **Account**: `Aesir.Commons.Models.Account`
   - User authentication data (username, password hash)
   - Account status (active, banned, etc.)
   - Security information

2. **Character**: `Aesir.Commons.Models.Character`
   - Character attributes and stats
   - Position and map information
   - Equipment and appearance data
   - Game state information

## Testing Approach


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EndurnyrProject/aesir](https://github.com/EndurnyrProject/aesir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
