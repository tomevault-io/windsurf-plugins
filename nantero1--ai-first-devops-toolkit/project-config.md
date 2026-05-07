---
trigger: always_on
description: Provides unified interface for Stripe, PayPal, and Square payments.
---

# GitHub Copilot Instructions

This file provides comprehensive guidance to AI to assist with development tasks in this Python project. Follow these instructions for all code generation, documentation, and testing activities.

## 🐍 Python Development Standards

### Code Style & Structure
- **Base Standard**: Google Python Style Guide with PEP 8 compliance
- **Line Length**: 120 characters maximum
- **Purpose Documentation**: Every file and class must explain its purpose with detailed docstrings
- **Readability First**: Code is read more often than written - prioritize clarity
- **Linting**: Use `ruff check` and `ruff format` for code quality and formatting
- **Type Checking**: Use `mypy` for static type checking to catch type-related issues early

### Import Organization
```python
from __future__ import annotations

# Standard library
import os
from typing import Dict, List, Optional

# Third-party
import requests
from pydantic import BaseModel, Field

# Project imports
from mixins.mixin_secrets import SecretsMixin
from our_utils.our_logger import get_formatted_logger
```

### Class Documentation Pattern
```python
class PaymentProcessor:
    """Processes payments through multiple payment gateways.
    
    Provides unified interface for Stripe, PayPal, and Square payments.
    Handles validation, error handling, and transaction logging with
    automatic retry logic for failed transactions.
    
    Attributes:
        retry_attempts: Number of retries for failed transactions.
        supported_currencies: List of supported currency codes.
    """
```

### Error Handling Patterns
- Use specific exception types with descriptive messages
- Implement fallback mechanisms - avoid crashes when possible
- Log errors but continue with sane defaults
- Use `ToolException` for external service errors

```python
from langchain_core.tools import ToolException

def process_data(data: str) -> dict:
    """Process user data with proper error handling."""
    try:
        result = external_service.process(data)
        LOGGER.info("Data processed successfully", data_size=len(data))
        return result
    except ValidationError as e:
        LOGGER.error("Validation failed", error=str(e), data=data)
        raise ToolException(f"Invalid data format: {e}")
    except ExternalServiceError as e:
        LOGGER.error("External service failed", service="processor", error=str(e))
        raise ToolException("Service temporarily unavailable. Please try again.")
```

### Domain Models with Pydantic
```python
from pydantic import BaseModel, Field
from datetime import datetime

class User(BaseModel):
    """Domain aggregate for user data and business logic."""
    
    id: str = Field(..., description="Unique user identifier")
    preferences: Dict[str, Any] = Field(
        default_factory=dict, 
        description="User preferences and settings"
    )
    created_at: datetime = Field(
        default_factory=datetime.now,
        description="User creation timestamp"
    )
    
    def has_active_subscription(self) -> bool:
        """Check if user has an active subscription."""
        return self.subscription is not None and self.subscription.is_active
```

## 🧪 Testing Standards

### Test Structure & Organization
- **Directory Structure**: Mirror source code structure in `tests/unit/` and `tests/integration/`
- **File Naming**: Prefix with `test_` and mirror source structure
- **Given-When-Then Pattern**: Mandatory for all tests

### Test Template
```python
class TestMyComponent:
    """Tests for MyComponent class."""
    
    def test_basic_functionality(self, component):
        """Test basic component functionality."""
        # given
        input_data = "test input"
        
        # when
        result = component.process(input_data)
        
        # then
        assert result == "expected output"
    
    @pytest.mark.parametrize("input_data, expected", [
        pytest.param("input1", "output1", id="scenario1"),
        pytest.param("input2", "output2", id="scenario2"),
    ])
    def test_multiple_scenarios(self, input_data, expected):
        """Test multiple scenarios with different inputs."""
        # given
        component = ComponentUnderTest()
        
        # when
        result = component.process(input_data)
        
        # then
        assert result == expected
```

### Async Testing
```python
@pytest.mark.asyncio
async def test_async_functionality(self):
    """Test async operations."""
    # given
    mock_service = AsyncMock()
    mock_service.process.return_value = "async_result"
    
    # when
    result = await component.async_method()
    
    # then
    assert result == "async_result"
    mock_service.process.assert_called_once()
```

### Testing Best Practices
- Use Given-When-Then structure for all tests
- Group related tests in classes
- Use descriptive test names explaining what is being tested
- Leverage autouse fixtures - don't manually mock what's auto-mocked
- Test both success and error paths
- Use parametrization for multiple scenarios
- Keep tests focused with one assertion per test concept

## 📝 Documentation Standards

### Inline Code Documentation
- **Keep all existing comments** - only remove if wrong, not helpful, or outdated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nantero1/ai-first-devops-toolkit](https://github.com/Nantero1/ai-first-devops-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
