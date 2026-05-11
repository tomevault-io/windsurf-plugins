---
trigger: always_on
description: **CRITICAL: Never use emojis or special characters.**
---

# Claude Code Guidelines

**CRITICAL: Never use emojis or special characters.**

---

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Add changelog fragments** (if user-facing changes) - Use towncrier format:
   - Feature: `changelog.d/feature/<short-name>.md`
   - Bug fix: `changelog.d/fix/<short-name>.md`
   - Breaking: `changelog.d/breaking/<short-name>.md`
   - Content: Brief description of the change (1-2 sentences)
4. **Update issue status** - Close finished work, update in-progress items
5. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
6. **Clean up** - Clear stashes, prune remote branches
7. **Verify** - All changes committed AND pushed
8. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---

## ISSUE DISCOVERY (MANDATORY)

While coding, you MUST document any issues you encounter using `bd create`. This includes:

**Code Quality Issues:**
- SOLID principle violations (god classes, missing protocols, hard-coded dependencies)
- Missing dependency injection
- Concrete classes without protocols
- Tests that violate guidelines (over-mocked, structure-only, no behavior testing)
- Missing edge case handling

**Bugs and Problems:**
- Runtime errors or unexpected behavior
- Logic errors discovered during implementation
- Integration issues between components
- Performance problems

**Technical Debt:**
- TODO/FIXME comments in code
- Incomplete implementations
- Missing tests for existing functionality
- Documentation gaps

---

## ARCHITECTURAL PRINCIPLES (READ THIS FIRST)

### You Are an Architect, Not a Code Monkey

Before writing ANY code, you must:
1. **Design the abstraction** - What protocol/interface is needed?
2. **Consider SOLID principles** - Is this following Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion?
3. **Plan dependency injection** - How will this be tested? What needs to be injected?
4. **Think about edge cases** - What can go wrong? What are the boundaries?
5. **Design before coding** - No coding until the design is clear

### MANDATORY: Protocol-First Design

**NEVER write a concrete class without defining its protocol first.**

```python
# WRONG - Concrete class first
class ResponseCache:
    def get(self, key: str) -> Optional[str]:
        return self._cache.get(key)

# RIGHT - Protocol first, then implementation
class CacheProtocol(Protocol):
    """Defines the contract for caching behavior."""
    def get(self, key: str) -> Optional[str]: ...
    def put(self, key: str, value: str) -> None: ...
    def clear(self) -> None: ...

class ResponseCache:  # Implements CacheProtocol
    def get(self, key: str) -> Optional[str]:
        return self._cache.get(key)
```

**Why?** Protocols enable:
- Testing with test doubles
- Swapping implementations
- Dependency inversion
- Clear contracts

---

## SOLID PRINCIPLES (NON-NEGOTIABLE)

### Single Responsibility Principle
**Each class should have ONE reason to change.**

**BAD - God Class:**
```python
class AgentOrchestrator:
    def __init__(self):
        self.cache = ResponseCache()
        self.rate_tracker = RateLimitTracker()
        self.session_manager = SessionManager()
        # ... does caching, rate limiting, sessions, delegation, context, etc.
```

**GOOD - Focused Classes:**
```python
class Orchestrator:
    def __init__(
        self,
        cache: CacheProtocol,
        rate_tracker: RateLimitProtocol,
        session: SessionProtocol,
        delegator: DelegationProtocol,
    ):
        # Each dependency is a focused, single-purpose component
```

### Open/Closed Principle
**Open for extension, closed for modification.**

Use strategy pattern, not if/else chains.

 **BAD:**
```python
def execute(self, task_type: str):
    if task_type == "research":
        # research logic
    elif task_type == "coding":
        # coding logic
    # Adding new type = modifying this function
```

**GOOD:**
```python
class ExecutionStrategy(Protocol):
    def execute(self, task: Task) -> Result: ...

# Add new strategies without modifying existing code
strategies = {
    TaskType.RESEARCH: ResearchStrategy(),
    TaskType.CODING: CodingStrategy(),
}
```

### Liskov Substitution Principle
**Subtypes must be substitutable for their base types.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HakAl/scrappy](https://github.com/HakAl/scrappy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
