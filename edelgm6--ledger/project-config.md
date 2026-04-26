---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run development server
uv run python manage.py runserver

# Run all tests
uv run python manage.py test

# Run specific test module
uv run python manage.py test api.tests.models.test_transaction
uv run python manage.py test api.tests.statement.test_balance_sheet

# Run with coverage
uv run coverage run --source='.' manage.py test
uv run coverage report

# Run Celery worker (for async Textract processing)
uv run celery -A api worker --loglevel=info

# Create migrations after model changes
uv run python manage.py makemigrations
uv run python manage.py migrate
```

## Architecture

Ledger is a Django personal accounting application that generates three accounting statements (income statement, balance sheet, statement of cash flows) using double-entry bookkeeping.

### Core Structure

- **ledger/** - Django project configuration (settings, urls, wsgi)
- **api/** - Main Django app containing all business logic
  - `models.py` - 19 model classes for accounting entities
  - `statement.py` - Statement generation (IncomeStatement, BalanceSheet, CashFlowStatement, Trend)
  - `views/` - HTMX-based views returning HTML fragments
  - `services/` - Business logic (journal_entry_services.py)
  - `forms.py` - Django forms with custom CommaDecimalField for currency
  - `aws_services.py` - S3 uploads and Textract OCR integration
  - `tasks.py` - Celery tasks for async document processing

### Key Models and Relationships

**Account** → has many **Transactions** → each creates a **JournalEntry** → with multiple **JournalEntryItems** (debits/credits)

- **Entity**: Business entities (companies, individuals) which can be linked to accounts as a default (e.g., every time my Checking Account has a Transaction, auto-link to Ally Bank)
- **Account**: Chart of accounts with types (ASSET, LIABILITY, INCOME, EXPENSE, EQUITY) and sub_types
- **Transaction**: Financial transactions that auto-generate journal entries
- **TaxCharge**: Tax liability linked to transactions
- **Amortization**: Expense amortization over periods
- **Reconciliation**: Account balance verification with plug transactions

### Double-Entry Bookkeeping Logic

Every Transaction creates a JournalEntry with balanced JournalEntryItems:
- Debits increase: ASSET, EXPENSE accounts
- Debits decrease: LIABILITY, INCOME, EQUITY accounts
- `Account.get_balance(end_date, start_date)` sums JournalEntryItems respecting debit/credit rules

### Frontend Pattern

HTMX-based - views return HTML fragments via `render_to_string()`. No separate API endpoints; views render templates directly for AJAX updates.

### Local Development Setup

Create `ledger/local_settings.py` with SECRET_KEY, AWS credentials, and database config (see README.md for template).

### Testing

Tests use Factory Boy (`api/tests/testing_factories.py`) for test data. Test structure:
- `api/tests/models/` - Model unit tests
- `api/tests/statement/` - Statement generation tests

## Code Organization Patterns

This codebase follows a strict separation of concerns for maintainability and testability. The refactoring of `journal_entry_views.py` serves as the reference implementation of this pattern.

### Architecture Layers

```
Views (HTTP orchestration)
  ↓
Helpers (pure HTML rendering functions)
  ↓
Forms (field validation)
  ↓
Services (business logic + atomic DB writes)
  ↓
Models (domain entities)
```

### Service Layer Guidelines (`api/services/`)

Services contain all business logic and database writes:

- **All database writes must be in services** - No `.save()`, `.delete()`, or `.create()` in views
- **Use `@db_transaction.atomic`** for multi-step operations to ensure atomicity
- **Return dataclass result objects** (e.g., `SaveResult`, `ValidationResult`) with:
  - `success: bool` - Whether operation succeeded
  - `data: Optional[Model]` - The created/updated object
  - `error: Optional[str]` - Error message if failed
- **Use bulk operations** - `bulk_create()`, `bulk_update()` for efficiency
- **Type hints required** on all function parameters and return types
- **Pure functions** - No HTTP dependencies (no `request` objects)
- **Single responsibility** - Each function does one thing well

**Example:**
```python
@dataclass
class SaveResult:
    success: bool
    journal_entry: Optional[JournalEntry] = None
    error: Optional[str] = None

@db_transaction.atomic
def save_journal_entry(
    date: datetime.date,
    memo: str,
    items: List[Dict[str, Any]]
) -> SaveResult:
    """Creates a journal entry with balanced items."""
    # Validation
    if not validate_items_balance(items):
        return SaveResult(success=False, error="Debits must equal credits")

    # Create entry
    entry = JournalEntry.objects.create(date=date, memo=memo)

    # Bulk create items
    item_objects = [JournalEntryItem(**item, journal_entry=entry) for item in items]
    JournalEntryItem.objects.bulk_create(item_objects)

    return SaveResult(success=True, journal_entry=entry)
```

### Helper Function Guidelines (`api/views/*_helpers.py`)

Helpers are pure functions for HTML rendering:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edelgm6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
