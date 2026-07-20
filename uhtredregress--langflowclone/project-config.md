---
trigger: always_on
description: Guidelines for backend development in Langflow, focusing on Python components, FastAPI services, and backend testing.
---



# Backend Development Guidelines

## Purpose
Guidelines for backend development in Langflow, focusing on Python components, FastAPI services, and backend testing.

---

## 1. Backend Environment Setup

### Prerequisites
- **Python Package Manager:** `uv` (>=0.4) for dependency management
- **Database:** SQLite for development, PostgreSQL for production
- **Development Tools:** `make` for build coordination

### Backend Service
```bash
make backend  # Start FastAPI backend on port 7860
```
- Auto-reloads on file changes
- Health check: http://localhost:7860/health
- Backend components: `src/backend/base/langflow/`

---

## 2. Component Development

### Component Structure
```
src/backend/base/langflow/components/
├── agents/           # Agent components
├── data/            # Data processing components
├── embeddings/      # Embedding components
├── input_output/    # Input/output components
├── models/          # Language model components
├── processing/      # Text processing components
├── prompts/         # Prompt components
├── tools/           # Tool components
└── vectorstores/    # Vector store components
```

### Adding New Components
1. **Location:** Add to appropriate subdirectory under `src/backend/base/langflow/components/`
2. **Import:** Update `__init__.py` with alphabetical imports:
   ```python
   from .my_component import MyComponent

   __all__ = [
       "ExistingComponent",
       "MyComponent",  # Add alphabetically
   ]
   ```
3. **Auto-restart:** Backend auto-restarts on save
4. **Browser refresh:** Refresh browser to see component changes

### Component Testing
- **Unit Tests:** `src/backend/tests/unit/components/`
- **Test Structure:** Mirror component directory structure
- **Test Base Classes:** Use `ComponentTestBaseWithClient` or `ComponentTestBaseWithoutClient`
- **Version Testing:** Provide `file_names_mapping` for backward compatibility

### Development Tips
- **Fast iteration:** Edit component in UI first, then save to source
- **Component updates:** Old components show "Updates Available" after backend restart
- **Testing:** Create comprehensive unit tests for all new components

---

## 3. Backend Code Quality

### Formatting (CRITICAL)
```bash
make format_backend  # Format Python code
```
**Important:** Run `make format_backend` _early and often_ (ideally before running linting or committing changes). It auto-corrects the majority of style issues, preventing lengthy manual fixes when lint errors surface later.

### Linting
```bash
make lint  # Run linting checks
```

### Testing
```bash
make unit_tests  # Run backend unit tests
```

### Pre-commit Workflow
1. **Run `make format_backend`** (FIRST - saves time on lint fixes)
2. Run `make lint`
3. Run `make unit_tests`
4. Commit changes

---

## 4. FastAPI Development

### API Structure
```
src/backend/base/langflow/api/
├── v1/              # API version 1
│   ├── chat.py      # Chat endpoints
│   ├── flows.py     # Flow management
│   ├── users.py     # User management
│   └── ...
└── v2/              # API version 2 (future)
```

### Testing APIs
- Use `client` fixture from `conftest.py`
- Test with `logged_in_headers` for authenticated endpoints
- Example:
  ```python
  async def test_flows_endpoint(client, logged_in_headers):
      response = await client.post(
          "api/v1/flows/",
          json=flow_data,
          headers=logged_in_headers
      )
      assert response.status_code == 201
  ```

---

## 5. Database Development

### Models Location
```
src/backend/base/langflow/services/database/models/
├── api_key/         # API key models
├── flow/            # Flow models
├── folder/          # Folder models
├── user/            # User models
└── ...
```

### Database Testing
- Use in-memory SQLite for tests
- Database tests may fail in batch runs - run individually if needed:
  ```bash
  uv run pytest src/backend/tests/unit/test_database.py
  ```

---

## 6. Async Development Patterns

### Component Async Methods
```python
async def run(self) -> MessageType:
    """Main component execution method."""
    # Use await for async operations
    result = await self.async_operation()
    return result

async def message_response(self) -> Message:
    """Return a Message object for chat components."""
    return Message(
        text=self.input_value,
        sender=self.sender,
        session_id=self.session_id,
    )
```

### Background Tasks
```python
import asyncio

async def process_in_background(self):
    """Process items without blocking."""
    # Use asyncio.create_task for background work
    task = asyncio.create_task(self.heavy_operation())

    # Ensure proper cleanup
    try:
        result = await task
        return result
    except asyncio.CancelledError:
        # Handle cancellation gracefully
        await self.cleanup()
        raise
```

### Queue Operations
```python
async def queue_processing(self):
    """Non-blocking queue operations."""
    queue = asyncio.Queue()

    # Non-blocking put
    queue.put_nowait(data)

    # Timeout-controlled get
    try:
        result = await asyncio.wait_for(queue.get(), timeout=5.0)
        return result
    except asyncio.TimeoutError:
        # Handle timeout appropriately

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UhtredRegress/LangFlowClone](https://github.com/UhtredRegress/LangFlowClone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
