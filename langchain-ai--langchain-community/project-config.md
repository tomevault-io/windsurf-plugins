---
trigger: always_on
description: **Always attempt to preserve function signatures, argument positions, and names for exported/public methods.**
---

# Global Development Guidelines for LangChain Projects

## Core Development Principles

### 1. Maintain Stable Public Interfaces ⚠️ CRITICAL

**Always attempt to preserve function signatures, argument positions, and names for exported/public methods.**

❌ **Bad - Breaking Change:**

```python
def get_user(id, verbose=False):  # Changed from `user_id`
    pass
```

✅ **Good - Stable Interface:**

```python
def get_user(user_id: str, verbose: bool = False) -> User:
    """Retrieve user by ID with optional verbose output."""
    pass
```

**Before making ANY changes to public APIs:**

- Check if the function/class is exported in `__init__.py`
- Look for existing usage patterns in tests and examples
- Use keyword-only arguments for new parameters: `*, new_param: str = "default"`
- Mark experimental features clearly with docstring admonitions (using MkDocs Material, like `!!! warning`)

🧠 _Ask yourself:_ "Would this change break someone's code if they used it last week?"

### 2. Code Quality Standards

**All Python code MUST include type hints and return types.**

❌ **Bad:**

```python
def p(u, d):
    return [x for x in u if x not in d]
```

✅ **Good:**

```python
def filter_unknown_users(users: list[str], known_users: set[str]) -> list[str]:
    """Filter out users that are not in the known users set.

    Args:
        users: List of user identifiers to filter.
        known_users: Set of known/valid user identifiers.

    Returns:
        List of users that are not in the known_users set.
    """
    return [user for user in users if user not in known_users]
```

**Style Requirements:**

- Use descriptive, **self-explanatory variable names**. Avoid overly short or cryptic identifiers.
- Attempt to break up complex functions (>20 lines) into smaller, focused functions where it makes sense
- Avoid unnecessary abstraction or premature optimization
- Follow existing patterns in the codebase you're modifying

### 3. Testing Requirements

**Every new feature or bugfix MUST be covered by unit tests.**

**Test Organization:**

- Unit tests: `tests/unit_tests/` (no network calls allowed)
- Integration tests: `tests/integration_tests/` (network calls permitted)
- Use `pytest` as the testing framework

**Test Quality Checklist:**

- [ ] Tests fail when your new logic is broken
- [ ] Happy path is covered
- [ ] Edge cases and error conditions are tested
- [ ] Use fixtures/mocks for external dependencies
- [ ] Tests are deterministic (no flaky tests)

Checklist questions:

- [ ] Does the test suite fail if your new logic is broken?
- [ ] Are all expected behaviors exercised (happy path, invalid input, etc)?
- [ ] Do tests use fixtures or mocks where needed?

```python
def test_filter_unknown_users():
    """Test filtering unknown users from a list."""
    users = ["alice", "bob", "charlie"]
    known_users = {"alice", "bob"}

    result = filter_unknown_users(users, known_users)

    assert result == ["charlie"]
    assert len(result) == 1
```

### 4. Security and Risk Assessment

**Security Checklist:**

- No `eval()`, `exec()`, or `pickle` on user-controlled input
- Proper exception handling (no bare `except:`) and use a `msg` variable for error messages
- Remove unreachable/commented code before committing
- Race conditions or resource leaks (file handles, sockets, threads).
- Ensure proper resource cleanup (file handles, connections)

❌ **Bad:**

```python
def load_config(path):
    with open(path) as f:
        return eval(f.read())  # ⚠️ Never eval config
```

✅ **Good:**

```python
import json

def load_config(path: str) -> dict:
    with open(path) as f:
        return json.load(f)
```

### 5. Documentation Standards

**Use Google-style docstrings with Args and Returns sections for all public functions.**

❌ **Insufficient Documentation:**

```python
def send_email(to, msg):
    """Send an email to a recipient."""
```

✅ **Complete Documentation:**

```python
def send_email(to: str, msg: str, *, priority: str = "normal") -> bool:
    """
    Send an email to a recipient with specified priority.

    Args:
        to: The email address of the recipient.
        msg: The message body to send.
        priority: Email priority level.

    Returns:
        True if email was sent successfully, False otherwise.

    Raises:
        InvalidEmailError: If the email address format is invalid.
        SMTPConnectionError: If unable to connect to email server.
    """
```

**Documentation Guidelines:**

- Types go in function signatures, NOT in docstrings
- Focus on "why" rather than "what" in descriptions
- Document all parameters, return values, and exceptions
- Keep descriptions concise but clear

📌 _Tip:_ Keep descriptions concise but clear. Only document return values if non-obvious.

### 6. Architectural Improvements

**When you encounter code that could be improved, suggest better designs:**

❌ **Poor Design:**

```python
def process_data(data, db_conn, email_client, logger):
    # Function doing too many things
    validated = validate_data(data)
    result = db_conn.save(validated)
    email_client.send_notification(result)
    logger.log(f"Processed {len(data)} items")
    return result
```

✅ **Better Design:**

```python
@dataclass
class ProcessingResult:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain-ai/langchain-community](https://github.com/langchain-ai/langchain-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
