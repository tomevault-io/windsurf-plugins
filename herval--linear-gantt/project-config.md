---
trigger: always_on
description: This document outlines the development practices and methodology for maintaining this project.
---

# Development Methodology for Linear Gantt Chart Visualizer

This document outlines the development practices and methodology for maintaining this project.

## Core Principles

1. **Test-Driven Development**: Write tests before or alongside code changes
2. **Clean Commits**: Commit working, tested code at stable milestones
3. **Documentation**: Keep README, TODO, and PROJECT_SPEC in sync
4. **Code Reuse**: Follow DRY principles and maintain modular architecture
5. **Scope Discipline**: Never modify files outside this working directory

---

## Testing Requirements

### MANDATORY: Tests for Every Change

**Before writing code:**
1. Understand what you're changing and what could break
2. Identify which test file(s) need updates
3. Write failing tests first (TDD approach when possible)

**After writing code:**
1. Run relevant test suite: `python -m pytest tests/test_<module>.py -v`
2. Run ALL tests before committing: `python -m pytest tests/ -v`
3. Ensure ALL tests pass (no exceptions)
4. Add new tests for new functionality

### Test Coverage Areas

- **`tests/test_models.py`**: Project and Issue data models
- **`tests/test_date_logic.py`**: Smart date calculation, velocity-based estimates
- **`tests/test_visualization.py`**: Gantt chart rendering logic
- **`tests/test_api.py`**: Linear API client and authentication
- **`tests/test_app.py`**: Streamlit application logic

### Test Writing Guidelines

```python
# Good: Descriptive test names
def test_velocity_calculation_with_3_completed_tasks_estimates_60_days():
    """Test basic velocity calculation: 3 tasks in 30 days, 6 remaining"""
    # Arrange
    data = {...}
    # Act
    result = project.get_effective_end_date()
    # Assert
    assert result == expected_date

# Good: Test edge cases
def test_velocity_calculation_falls_back_when_no_completed_tasks():
    """Test velocity calculation falls back to start + 6 months"""
    ...
```

---

## Development Workflow

### 1. Planning Phase

**Use TodoWrite tool to plan tasks:**
```python
TodoWrite({
    "todos": [
        {"content": "Add feature X", "status": "in_progress", "activeForm": "Adding feature X"},
        {"content": "Write tests for X", "status": "pending", "activeForm": "Writing tests for X"},
        {"content": "Update docs", "status": "pending", "activeForm": "Updating docs"},
        {"content": "Commit changes", "status": "pending", "activeForm": "Committing changes"}
    ]
})
```

**Break down complex tasks:**
- If a task has 3+ steps, break it into smaller tasks
- Mark tasks as in_progress → completed as you go
- Keep TODO list in sync with actual progress

### 2. Implementation Phase

**Order of operations:**
1. Read relevant source files
2. Write/update tests (if TDD approach)
3. Implement the feature/fix
4. Run tests and iterate until passing
5. Update TodoWrite to mark task completed

**Code organization:**
- **Models** (`src/models/`): Data structures, business logic
- **API** (`src/api/`): Linear API client, GraphQL queries
- **Visualization** (`src/visualization/`): Gantt chart rendering
- **Utils** (`src/utils/`): Caching, authentication, helpers
- **Config** (`config/`): Settings, constants, color schemes

### 3. Testing Phase

**Run tests progressively:**
```bash
# Test the specific module you changed
python -m pytest tests/test_models.py -v

# Run all tests before committing
python -m pytest tests/ -v

# Check test coverage (optional but encouraged)
python -m pytest tests/ --cov=src --cov-report=term-missing
```

**What to test:**
- Happy path (normal usage)
- Edge cases (empty data, None values, missing fields)
- Error handling (invalid input, API errors)
- Backward compatibility (cached data, old project structures)

### 4. Commit Phase

**Only commit when:**
- ✅ All tests pass
- ✅ Code is working and tested
- ✅ TODO list is updated
- ✅ Changes are at a stable milestone

**Commit message format:**
```
Brief summary of changes (imperative mood)

Detailed description of what changed and why:
- Added X feature to handle Y scenario
- Updated Z to support new functionality
- Fixed bug where A would cause B

Technical details:
- Modified file1.py: added method_x()
- Updated file2.py: changed logic in method_y()
- Added tests in test_file.py

All N tests passing.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Git workflow:**
```bash
# Check status
git status

# Review changes
git diff <files>

# Stage files
git add <files>

# Commit with detailed message
git commit -m "$(cat <<'EOF'
Your detailed commit message here
EOF
)"

# Verify commit
git log --oneline -1
```

---

## Streamlit-Specific Considerations

### Caching

**Important:** Streamlit caches function results. When adding new fields to models:

1. **Update the model** (e.g., add `team_names` field)
2. **Add defensive checks** for cached objects:
   ```python
   # Good: Backward compatible with cached data
   team_names = getattr(project, 'team_names', [])

   # Bad: Will break with cached objects
   team_names = project.team_names
   ```
3. **Test with cache clear**: User can click "Refresh Data" to clear cache

### Function Signatures

**Cache-friendly function signatures:**
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herval/linear-gantt](https://github.com/herval/linear-gantt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
