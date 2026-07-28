---
trigger: always_on
description: MeshTUI is a Textual-based TUI (Terminal User Interface) application for interfacing with MeshCore companion radios. This project provides a rich terminal interface for managing mesh network communications, contacts, channels, and node management.
---

# GitHub Copilot Instructions for MeshTUI

## Project Overview

MeshTUI is a Textual-based TUI (Terminal User Interface) application for interfacing with MeshCore companion radios. This project provides a rich terminal interface for managing mesh network communications, contacts, channels, and node management.

## Code Style and Standards

### Python Best Practices

1. **Type Hints**: Always use type hints for function parameters and return values
   ```python
   async def send_message(self, recipient: str, message: str) -> bool:
   ```

2. **Docstrings**: Use comprehensive docstrings for all public methods and classes
   ```python
   async def refresh_contacts(self):
       """Refresh the contacts list from the connected MeshCore device.
       
       This method updates the internal contacts list by querying the device
       and ensures the UI is synchronized with the latest contact information.
       """
   ```

3. **Error Handling**: Always wrap async operations in try-catch blocks with proper logging
   ```python
   try:
       result = await asyncio.wait_for(self.meshcore.commands.get_contacts(), timeout=5.0)
   except asyncio.TimeoutError:
       self.logger.error("Timeout refreshing contacts")
   except Exception as e:
       self.logger.error(f"Failed to refresh contacts: {e}")
   ```

4. **Logging**: Use structured logging with appropriate levels
   ```python
   self.logger.info(f"Successfully connected to {device_name}")
   self.logger.debug(f"Device info: {device_info}")
   self.logger.error(f"Connection failed: {error}")
   ```

### MeshCore API Patterns

1. **Connection Management**: Always check connection status before operations
   ```python
   if not self.meshcore or not self.meshcore.is_connected:
       return False
   ```

2. **Command Execution**: Access commands through the `commands` attribute
   ```python
   result = await self.meshcore.commands.get_contacts()
   if result.type != EventType.ERROR:
       # Process successful result
   ```

3. **Event Handling**: Subscribe to relevant events for reactive updates
   ```python
   self.meshcore.subscribe(EventType.CONTACT_MSG_RECV, self._handle_contact_message)
   self.meshcore.subscribe(EventType.CHANNEL_MSG_RECV, self._handle_channel_message)
   ```

4. **Async Best Practices**: Use timeouts for all async operations
   ```python
   result = await asyncio.wait_for(operation(), timeout=5.0)
   ```

### Textual UI Patterns

1. **Widget References**: Store widget references in `on_mount()` for efficient access
   ```python
   def on_mount(self) -> None:
       self.chat_area = self.query_one("#chat-area", TextArea)
       self.contacts_list = self.query_one("#contacts-list", ListView)
   ```

2. **Event Handlers**: Use the `@on` decorator for clean event handling
   ```python
   @on(Button.Pressed, "#send-btn")
   async def send_message(self) -> None:
   ```

3. **UI Updates**: Always update UI elements from the main thread
   ```python
   # Good: Direct UI update
   self.chat_area.insert(f"[green]{sender}:[/green] {message}\n")
   ```

### Git Operations

**IMPORTANT: Always use GitKraken tools for git operations**

- Use `mcp_gitkraken_git_add_or_commit` with `action: "add"` to stage files
- Use `mcp_gitkraken_git_add_or_commit` with `action: "commit"` to commit changes
- Use `mcp_gitkraken_git_push` to push changes
- Use `mcp_gitkraken_git_blame` to view file history
- Never use terminal commands like `git add`, `git commit`, etc.

Example commit workflow:
```python
# Stage all changes
mcp_gitkraken_git_add_or_commit(action="add", directory="/path/to/repo")

# Commit with message
mcp_gitkraken_git_add_or_commit(
    action="commit", 
    directory="/path/to/repo",
    message="Descriptive commit message"
)
```

### Async Best Practices

1. **Concurrency**: Use `asyncio.wait_for()` for timeout protection
2. **Task Management**: Properly clean up background tasks in `disconnect()`
3. **Error Propagation**: Let async exceptions bubble up with proper logging

### Project Structure

```
src/meshtui/
├── __init__.py          # Package initialization
├── __main__.py          # Entry point for python -m meshtui
├── app.py              # Main Textual application class (UI layer)
├── app.css            # Textual CSS styling
├── connection.py       # Connection orchestration and lifecycle
├── transport.py        # NEW: BLE, Serial, TCP transport layers
├── contact.py          # NEW: Contact/node management
├── channel.py          # NEW: Channel operations
└── room.py            # NEW: Room server handling
```

### Architecture Overview

**Layered Architecture:**
- **UI Layer** (`app.py`) - Textual widgets and user interaction
- **Connection Layer** (`connection.py`) - Orchestrates managers and transport
- **Manager Layer** (`contact.py`, `channel.py`, `room.py`) - Business logic
- **Transport Layer** (`transport.py`) - Low-level connection handling

**Design Principles:**
- **Separation of Concerns** - Each module has a single responsibility
- **Dependency Injection** - Managers receive MeshCore instance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekollof/meshtui](https://github.com/ekollof/meshtui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
