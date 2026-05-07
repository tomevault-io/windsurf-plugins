---
trigger: always_on
description: **MANDATORY** perform the following actions:
---

# Claude Code Configuration for {{PROJECT_NAME}}

## At the Start of ANY Work Session

**MANDATORY** perform the following actions:

1. Read the **`.memory_bank/README.md`** file completely.
2. Follow the mandatory reading sequence instructions from this file:
   - **[Tech Stack](.memory_bank/tech_stack.md)**: Learn which technologies, libraries and versions we use
   - **[Coding Standards](.memory_bank/guides/coding_standards.md)**: Formatting rules, naming conventions and best practices
   - **[Current Tasks](.memory_bank/current_tasks.md)**: List of active tasks and current team focus
3. Follow links to relevant documents depending on task type:
   - For new features → study specification in `.memory_bank/specs/`
   - For bugs → study workflow `.memory_bank/workflows/bug_fix.md`
   - For technology questions → check `.memory_bank/tech_stack.md`

---

## About the Project: {{PROJECT_NAME}}

**{{PROJECT_NAME}}** - {{PROJECT_DESC}}

### Key Project Features:

#### 1. {{LANGUAGE}} Architecture
- Using **{{LANGUAGE}}** with full type annotations
- **Framework**: {{FRAMEWORK}}
- **Asynchronous architecture**: all I/O operations via async/await
- Command and callback query handlers in `bot/` module (or main application logic)

#### 2. AI/LLM Integration (if applicable)
- **OpenAI API (GPT-4)** for analysis and report generation
- **LangChain** for orchestrating AI agents
- All LLM calls must be wrapped in retry mechanisms
- Use structured outputs for parsing LLM responses

#### 3. Async/Await Patterns
**CRITICALLY IMPORTANT:**
- All I/O operations (HTTP requests, database queries, file operations) MUST be asynchronous
- Use `async def` and `await` for all functions with I/O
- For HTTP requests use **httpx**, NOT requests
- For database queries use async drivers (asyncpg for PostgreSQL)
- **FORBIDDEN** to block event loop with synchronous calls

Correct approach example:
```python
import httpx
from typing import Dict, Any

async def fetch_company_data(company_id: str) -> Dict[str, Any]:
    """Asynchronously load company data."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/companies/{company_id}")
        response.raise_for_status()
        return response.json()
```

#### 4. External API Integrations
All external integrations must:
- Be located in `integrations/` module
- Have clear interface (Pydantic models for request/response)
- Include error handling according to `.memory_bank/patterns/error_handling.md`
- Use retry mechanisms for unstable APIs
- Have fallback strategies when service is unavailable
- Log all requests for debugging

Integration structure example:
```python
# integrations/company_registry.py
from typing import Optional
from pydantic import BaseModel
import httpx

class CompanyInfo(BaseModel):
    """Company data model."""
    name: str
    inn: str
    registration_date: str
    status: str

class CompanyRegistryClient:
    """Client for working with company registry."""

    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.example.com"

    async def get_company(self, inn: str) -> Optional[CompanyInfo]:
        """Get company information by INN."""
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/companies/{inn}",
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
            if response.status_code == 404:
                return None
            response.raise_for_status()
            return CompanyInfo(**response.json())
```

#### 5. Application-Specific Patterns
**When working with the application:**
- Use handlers for commands (`/start`, `/help`, `/check`) if applicable
- Use callback_query handlers for inline buttons (Telegram bots)
- Use FSM (Finite State Machine) for complex dialogs if applicable
- Handle errors gracefully - always send understandable message to user
- Use typing indicators (`send_chat_action`) for long operations if applicable
- Limit message size (Telegram limit: 4096 characters) if applicable

Handler example:
```python
from aiogram import Router, types
from aiogram.filters import Command

router = Router()

@router.message(Command("check"))
async def handle_check_command(message: types.Message) -> None:
    """Handler for /check command to start verification."""
    await message.answer("Sending verification request...")
    # Processing logic
```

#### 6. Data Processing & Storage
- **PostgreSQL** for storing structured data (companies, reports, users)
- **Redis** for caching and task queues
- All database models must be in `data/models.py`
- Use migrations (Alembic) for database schema changes
- Data validation via Pydantic before saving

---

## Self-Documentation Principle

**IMPORTANT**: You not only read from Memory Bank, but also **update it**.

When performing tasks you MUST:
- Update status in `.memory_bank/current_tasks.md` (To Do → In Progress → Done)
- Create/update documentation in `.memory_bank/guides/` when implementing new subsystems
- Update `.memory_bank/tech_stack.md` when adding new dependencies
- Create new patterns in `.memory_bank/patterns/` when making architectural decisions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [o2alexanderfedin/ai-swe-template](https://github.com/o2alexanderfedin/ai-swe-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
