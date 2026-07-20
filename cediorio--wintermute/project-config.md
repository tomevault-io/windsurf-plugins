---
trigger: always_on
description: This document outlines the test-driven development approach for building Wintermute.
---

# Wintermute Development Plan

This document outlines the test-driven development approach for building Wintermute.

## Development Phases

### Phase 1: Foundation (Models & Core Services)
**Goal**: Establish core data models and service interfaces

#### 1.1 Data Models
**Tests to Write**:
- `tests/test_models/test_persona.py`
  - ✓ Persona creation with valid data
  - ✓ Persona validation (required fields)
  - ✓ Persona serialization to/from JSON
  - ✓ Default values for optional fields
  
- `tests/test_models/test_message.py`
  - ✓ Message creation (user/assistant roles)
  - ✓ Message timestamp generation
  - ✓ Message formatting for display
  - ✓ Message metadata handling

**Implementation**:
- Create `Persona` class with fields: id, name, description, system_prompt, temperature, traits
- Create `Message` class with fields: role, content, timestamp, metadata
- Add validation using Pydantic
- Implement JSON serialization

#### 1.2 Configuration Management
**Tests to Write**:
- `tests/test_utils/test_config.py`
  - ✓ Load configuration from .env
  - ✓ Validate required settings
  - ✓ Default values for optional settings
  - ✓ Override with environment variables

**Implementation**:
- Create `Config` class using pydantic-settings
- Support .env file loading
- Validate URLs and API keys
- Provide sensible defaults

#### 1.3 Ollama Client
**Tests to Write**:
- `tests/test_services/test_ollama_client.py`
  - ✓ Initialize client with config
  - ✓ Check connection status
  - ✓ Generate response from prompt
  - ✓ Handle streaming responses
  - ✓ Error handling (connection failed, timeout)
  - ✓ Custom parameters (temperature, etc.)

**Implementation**:
- Create `OllamaClient` class
- Use httpx for async HTTP requests
- Implement `generate()` method
- Implement `stream()` method for real-time responses
- Add connection health check
- Handle errors gracefully

#### 1.4 OpenMemory Client
**Tests to Write**:
- `tests/test_services/test_memory_client.py`
  - ✓ Initialize client with config
  - ✓ Store memory item
  - ✓ Query memories by text
  - ✓ Retrieve user summary
  - ✓ Delete specific memories
  - ✓ Get memory stats
  - ✓ Error handling (API down, invalid key)

**Implementation**:
- Create `MemoryClient` class
- Implement CRUD operations for memories
- Add query method with filters
- Support user isolation (user_id)
- Implement connection health check
- Cache frequently accessed data

#### 1.5 Persona Manager
**Tests to Write**:
- `tests/test_services/test_persona_manager.py`
  - ✓ Load personas from directory
  - ✓ Get persona by ID
  - ✓ List all personas
  - ✓ Switch active persona
  - ✓ Validate persona data
  - ✓ Handle missing persona files
  - ✓ Reload personas dynamically

**Implementation**:
- Create `PersonaManager` class
- Load JSON persona files
- Validate persona schemas
- Track active persona
- Support hot-reloading

### Phase 2: UI Components
**Goal**: Build individual UI components with Textual

#### 2.1 Status Pane
**Tests to Write**:
- `tests/test_ui/test_status_pane.py`
  - ✓ Render initial status
  - ✓ Update connection status
  - ✓ Display memory count
  - ✓ Show model information
  - ✓ Handle status changes
  - ✓ Color coding (connected=green, disconnected=red)

**Implementation**:
- Create `StatusPane` widget (Static)
- Display connection indicators
- Show real-time stats
- Auto-refresh every 5 seconds
- Use Rich markup for styling

#### 2.2 Persona Pane
**Tests to Write**:
- `tests/test_ui/test_persona_pane.py`
  - ✓ Render persona list
  - ✓ Highlight active persona
  - ✓ Handle persona selection
  - ✓ Display persona descriptions on hover
  - ✓ Keyboard navigation
  - ✓ Update when personas change

**Implementation**:
- Create `PersonaPane` widget (ListView)
- Display all available personas
- Highlight current selection
- Emit events on selection change
- Add tooltips for descriptions
- Support keyboard shortcuts (1-9 for quick select)

#### 2.3 Chat Pane
**Tests to Write**:
- `tests/test_ui/test_chat_pane.py`
  - ✓ Render message history
  - ✓ Display user messages
  - ✓ Display assistant messages
  - ✓ Auto-scroll to bottom
  - ✓ Handle input submission
  - ✓ Show typing indicator
  - ✓ Handle long messages (wrapping)
  - ✓ Timestamp display

**Implementation**:
- Create `ChatPane` widget (Container)
- Use `VerticalScroll` for message history
- Add `Input` widget for user input
- Format messages with Rich styling
- Implement auto-scrolling
- Add typing indicator animation
- Handle markdown in messages

### Phase 3: Integration
**Goal**: Connect all components into working application

#### 3.1 Main Application
**Tests to Write**:
- `tests/test_app.py`
  - ✓ App initialization
  - ✓ Layout composition
  - ✓ Service initialization
  - ✓ Event routing
  - ✓ Graceful shutdown
  - ✓ Error recovery

**Implementation**:
- Create main `WintermuteApp` class
- Compose layout with Grid
- Initialize all services
- Set up event handlers
- Implement message loop:
  1. User input → Ollama
  2. Retrieve relevant memories
  3. Generate response with context
  4. Store new memories
  5. Display response
- Add error boundaries
- Implement graceful shutdown

#### 3.2 Message Flow
**Tests to Write**:
- `tests/test_integration/test_message_flow.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cediorio/wintermute](https://github.com/cediorio/wintermute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
