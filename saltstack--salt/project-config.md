---
trigger: always_on
description: You are assisting with development of Salt, a powerful infrastructure automation and configuration management system written in Python. This guide provides essential quick reference and links to detailed documentation.
---

# Salt Development Instructions for Gemini

You are assisting with development of Salt, a powerful infrastructure automation and configuration management system written in Python. This guide provides essential quick reference and links to detailed documentation.

## Coding Philosophy

**CRITICAL**: Write code as a CPython core developer:
- Understand Python internals, memory management, GC
- Know performance implications
- Be aware of implementation details

**Zen of Python**: Readability counts. Explicit > implicit. Simple > complex. Errors never pass silently.

**TDD for Bug Fixes**:
1. Write failing test first → 2. Verify it fails → 3. Fix bug (minimal changes) → 4. Verify test passes → 5. Run related tests

---

## Architecture Quick Reference

Salt is a Python-based configuration management system:
- **Master-Minion Architecture**: Central master controls distributed minions
- **Event-Driven**: Real-time communication via event bus
- **Plugin-Based**: Extensible through loader system

**Key Module Types:**
- **Execution Modules** (`salt/modules/`): CLI commands on minions (264+)
- **State Modules** (`salt/states/`): Declarative configuration (126+)
- **Utils** (`salt/utils/`): Shared utility functions (170+)

**See [agents/docs/architecture.md](agents/docs/architecture.md) for complete architecture.**

---

## Dunder Dictionaries

Salt injects special variables into module scope:

**Always Available:**
- `__opts__` - Config options: `__opts__.get("id")`, `__opts__["test"]`
- `__grains__` - System data: `__grains__.get("os_family")`
- `__pillar__` - Secure data: `__pillar__.get("password")`
- `__context__` - Per-run cache for expensive operations

**Module-Specific:**
- `__salt__` - Other execution modules: `__salt__["pkg.install"]("nginx")`
- `__utils__` - Utility functions: `__utils__["files.is_text"](path)`
- `__states__` - State modules (in state modules only)

**Context Caching Pattern:**
```python
if "pkg_list" not in __context__:
    __context__["pkg_list"] = expensive_fetch()  # Once per run
return __context__["pkg_list"]
```

**See [agents/docs/architecture.md](agents/docs/architecture.md) for loader system details.**

---

## Execution Module Structure

```python
import logging
import salt.exceptions
import salt.utils.platform

log = logging.getLogger(__name__)

def __virtual__():
    """Returns True to load, False or (False, reason) to skip"""
    if not salt.utils.path.which("required_binary"):
        return False, "required_binary not found"
    return True

def my_function(name, param=None):
    """
    CLI Example:
        salt '*' mymodule.my_function foo param=bar
    """
    if not name:
        raise salt.exceptions.SaltInvocationError("name required")
    return __salt__["cmd.run"](f"command {name}")
```

**See [agents/docs/module-templates.md](agents/docs/module-templates.md) for complete templates.**

---

## State Module Structure

**Required Return:**
```python
{
    "name": name,              # Resource identifier
    "result": True/False/None, # Success/Failure/Test mode
    "changes": {},             # {"old": ..., "new": ...}
    "comment": ""              # What happened
}
```

**State Flow:**
1. Validate input → 2. Check current state → 3. If correct: return success → 4. If test mode: return `result=None` with proposed changes → 5. Make changes → 6. Return result

**See [agents/docs/module-templates.md](agents/docs/module-templates.md) for complete state templates.**

---

## Common Patterns

### Logging
```python
log = logging.getLogger(__name__)
# Use lazy formatting (NOT f-strings)
log.debug("Processing: %s", filename)  # Good
log.debug(f"Processing: {filename}")   # Bad
```
**Never log sensitive data.**

### Error Handling
```python
from salt.exceptions import (
    CommandExecutionError,   # Operation failed
    SaltInvocationError,     # Invalid arguments
    CommandNotFoundError,    # Binary not found
)
```

### Platform Detection
```python
import salt.utils.platform
if salt.utils.platform.is_windows():
    # Windows-specific
if __grains__.get("os_family") == "Debian":
    # Debian-based
```

---

## Git Workflow

### CRITICAL RULES

1. **NO attribution lines** - **NEVER** add "Generated with Claude Code", "Co-Authored-By: Claude", or any AI attribution
2. **Imperative mood** - "Fix bug", not "Fixed bug"
3. **Reference issues** - Use `#NNNN`
4. **Small commits** - One logical change per commit
5. **Rebase before push** - Linear history

**Good Commit:**
```
Fix loader cache invalidation on module reload

Clear cache dict on reload to ensure fresh module imports.

Fixes #12345
```

**Bad (NEVER):**
```
Fixed bugs
Co-Authored-By: Claude <noreply@anthropic.com>
```

**See [agents/docs/git-and-ci.md](agents/docs/git-and-ci.md) for complete workflow.**

---

## Checklists

### Execution Module
- [ ] `log = logging.getLogger(__name__)`
- [ ] Docstrings with CLI examples
- [ ] `__virtual__()` if platform-specific
- [ ] Input validation (`SaltInvocationError`)
- [ ] Error handling (`CommandExecutionError`)
- [ ] Context caching for expensive ops
- [ ] Unit tests

### State Module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saltstack/salt](https://github.com/saltstack/salt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
