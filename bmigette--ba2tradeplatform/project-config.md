---
trigger: always_on
description: BA2 Trade Platform is a Python-based algorithmic trading platform built around a plugin architecture for accounts and market experts. It features a SQLModel-based ORM, NiceGUI web interface, and extensible settings system for AI-driven trading strategies.
---

# BA2 Trade Platform - AI Assistant Instructions

## Project Overview
BA2 Trade Platform is a Python-based algorithmic trading platform built around a plugin architecture for accounts and market experts. It features a SQLModel-based ORM, NiceGUI web interface, and extensible settings system for AI-driven trading strategies.

## Core Architecture

### Plugin System
- **Account Interfaces**: Implement `AccountInterface` for different brokers (e.g., `AlpacaAccount`)
- **Market Experts**: Implement `MarketExpertInterface` for AI trading strategies (e.g., `TradingAgents`)
- **Extensible Settings**: Both interfaces extend `ExtendableSettingsInterface` for flexible configuration

### Database Layer
- **SQLModel ORM**: All models in `ba2_trade_platform/core/models.py`
- **SQLite Database**: Located at `~/Documents/ba2_trade_platform/db.sqlite`
- **Database Functions**: Use `ba2_trade_platform/core/db.py` helpers (`get_instance`, `add_instance`, etc.)

### Directory Structure
```
ba2_trade_platform/
├── core/                    # Core interfaces and data models
│   ├── AccountInterface.py  # Abstract base for trading accounts
│   ├── MarketExpertInterface.py  # Abstract base for AI experts
│   ├── ExtendableSettingsInterface.py  # Settings management
│   ├── models.py           # SQLModel database models
│   ├── types.py            # Enums (OrderStatus, ExpertActionType, etc.)
│   ├── db.py              # Database utilities
│   └── utils.py           # Shared utility functions (close_transaction_with_logging, etc.)
├── modules/
│   ├── accounts/          # Account implementations (AlpacaAccount)
│   └── experts/           # Expert implementations (TradingAgents)
├── ui/                    # NiceGUI web interface
│   ├── main.py           # Route definitions
│   ├── pages/            # Page components
│   └── components/       # Reusable UI components
├── config.py             # Global configuration and environment variables
└── logger.py             # Centralized logging setup
```

## Key Patterns

### 1. **CRITICAL: Avoid Code Duplication**
**MOST IMPORTANT RULE**: Before writing any code, check if similar functionality already exists. If it does, refactor to use/extend the existing code instead of duplicating.

**Common Anti-Patterns to Avoid**:
- ❌ Copying try-except blocks with log_activity() calls
- ❌ Duplicating transaction closing logic in multiple places
- ❌ Repeating activity logging patterns with slight variations
- ❌ Copy-pasting order submission, cancellation, or validation logic

**Proper Approach**:
- ✅ **ALWAYS check `core/utils.py` first** for existing helper functions
- ✅ **Create helper functions** in `core/utils.py` for repeated patterns
- ✅ **Extract common logic** into centralized functions with clear parameters
- ✅ **Reuse existing functions** even if they need small modifications - extend them!

**Example of Good Practice**:
```python
# ❌ BAD: Duplicated logging code
try:
    from ..db import log_activity
    from ..types import ActivityLogSeverity, ActivityLogType
    log_activity(
        severity=ActivityLogSeverity.SUCCESS,
        activity_type=ActivityLogType.TRANSACTION_CLOSED,
        description=f"Submitted closing order for {symbol}",
        data={...},
        source_account_id=self.id,
        source_expert_id=expert_id
    )
except Exception as e:
    logger.warning(f"Failed to log: {e}")

# ✅ GOOD: Use centralized helper function
from ..utils import log_close_order_activity
log_close_order_activity(
    transaction=transaction,
    account_id=self.id,
    success=True,
    close_order_id=order_id
)
```

**Existing Helper Functions in `core/utils.py`**:
- `close_transaction_with_logging()` - Close transactions with P&L calculation and activity logging
- `log_close_order_activity()` - Log close order submission (success/failure/retry)
- `get_account_instance_from_id()` - Get cached account instances

**When to Create New Helper Functions**:
- When you find yourself writing similar code in 2+ places
- When error handling + logging patterns repeat
- When business logic (like order validation, price calculations) is duplicated
- When database operations follow the same pattern

**Refactoring Checklist**:
1. Search for similar code patterns in the codebase (`grep_search`, `semantic_search`)
2. Identify what varies and what stays the same
3. Extract common logic into a function with parameters for variations
4. Add the function to `core/utils.py` with clear docstring
5. Replace all instances with calls to the new function
6. Test to ensure behavior is preserved

### 2. **Settings Management**
All plugins use the ExtendableSettingsInterface pattern:
```python
class MyAccount(AccountInterface):
    @classmethod
    def get_settings_definitions(cls) -> Dict[str, Any]:
        return {
            "api_key": {"type": "str", "required": True, "description": "API Key"},
            "paper_account": {"type": "bool", "required": True, "description": "Paper trading?"}
        }
    
    def __init__(self, id: int):
        super().__init__(id)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bmigette/BA2TradePlatform](https://github.com/bmigette/BA2TradePlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
