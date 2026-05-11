---
trigger: always_on
description: Tests
---

# Testing Guide 

## Directory Structure & Organization

```
tests/
├── unit/                    # Unit tests with heavy mocking
│   └── somedirectory/           # Mirror source code structure
│       ├── langchain_related/
│       ├── user_client/
│       └── ...
└── integration/            # Integration tests with mocking of external dependencies / APIs only
```

**File Naming**: Test files prefixed with `test_` and mirror source structure:
- Source: `somedirectory/langchain_related/tools/planer.py`
- Test: `tests/unit/somedirectory/langchain_related/tools/test_planer.py`

## 🚨 **CRITICAL: AVOID OVER-TESTING**

*Preventing excessive test code and maintaining focus on behavior*

### **The Over-Testing Problem**

**WARNING SIGNS:**
- Test code exceeds business code by 2:1 ratio
- Testing every possible internal error scenario
- Multiple tests for the same behavior
- Testing implementation details instead of user outcomes
- Complex test setup for simple functions

### **Test-to-Code Ratio Guidelines**

**✅ HEALTHY RATIOS:**
```
Simple Functions (< 50 lines):    1:1 to 1.5:1 test-to-code ratio
Complex Business Logic:           1.5:1 to 2:1 test-to-code ratio
Critical System Components:       2:1 to 2.5:1 test-to-code ratio
```

**❌ UNHEALTHY RATIOS:**
```
> 3:1 ratio = Over-testing likely
> 5:1 ratio = Definitely over-testing
```

### **Simple Function Testing Strategy**

**For a simple function like `display_image()`:**

**✅ GOOD - Behavior-Focused (5-6 tests max):**
```python
class TestDisplayImage:
    """Tests for display_image function - behavior focused."""

    def test_successfully_displays_image(self):
        """Test that image is displayed when called."""
        # Test the core behavior

    def test_clears_screen_when_requested(self):
        """Test clear_first=True behavior."""
        # Test parameter behavior

    def test_skips_clearing_when_not_requested(self):
        """Test clear_first=False behavior."""
        # Test parameter behavior

    def test_handles_errors_gracefully(self):
        """Test error handling behavior."""
        # Test error scenarios

    @pytest.mark.parametrize("image_name", [...])
    def test_works_with_different_images(self):
        """Test with various inputs."""
        # Test input variations
```

**❌ BAD - Implementation-Focused (10+ tests):**
```python
# DON'T DO THIS - Over-testing implementation details
def test_display_image_handles_clear_error_during_error_recovery(self):
    """Testing internal error recovery mechanisms."""

def test_display_image_calls_correct_internal_methods_in_order(self):
    """Testing internal method call patterns."""

def test_display_image_prints_exact_messages_in_sequence(self):
    """Testing internal print statement details."""
```

### **When to Stop Testing**

**STOP adding tests when:**
- You're testing the same behavior in different ways
- You're testing internal implementation details
- Your test setup is more complex than the function being tested
- You have more test code than business logic code for simple functions
- Tests are breaking when you refactor internals (not behavior)

**ASK YOURSELF:**
- "Does this test verify something the user cares about?"
- "Would this test catch a real bug that affects end users?"
- "Am I testing behavior or implementation?"

## Running Tests

### Direct UV Execution
```bash
uv run pytest tests/unit/ -v
uv run pytest tests/unit/path/to/test.py::TestClass::test_method -v
```

## Test Structure Patterns

### Test Organization
```python
class TestPlanModel:
    """Tests for PlanModel class."""
    
    def test_valid_model(self):
        """Test creating a valid PlanModel."""
        # given
        data = {"selected_tool_names": ["Tool1"], "main_goal": "Test"}
        
        # when
        model = PlanModel(**data)
        
        # then
        assert model.selected_tool_names == ["Tool1"]
        assert model.main_goal == "Test"

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

### Given-When-Then Pattern (Required)
```python
def test_feature_functionality(self):
    """Test description of what this validates."""
    # given: setup test conditions
    mock_data = {"key": "value"}
    component = ComponentUnderTest()
    
    # when: execute the action being tested
    result = component.method_under_test(mock_data)
    
    # then: verify expected outcomes
    assert result == "expected_result"
    assert component.state == "expected_state"
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

## Common Test Patterns

### Unit Test Template
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nantero1/ai-first-devops-toolkit](https://github.com/Nantero1/ai-first-devops-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
