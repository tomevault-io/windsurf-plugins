---
trigger: always_on
description: This guide defines how Claude Code assists with developing the circuit-synth Python library.
---

# CLAUDE.md - circuit-synth Library Development Guide

This guide defines how Claude Code assists with developing the circuit-synth Python library.

## Project Context

- **Test-first mentality** - never commit untested code
- **Log-driven investigation** - observe behavior, don't assume
- **Small batch releases** - frequent, incremental PyPI releases
- **Professional quality** - 80%+ test coverage, type hints, CI/CD enforcement

---

## Professional Quality Standards

All work must meet professional software engineering standards:

- **Testing**: 80%+ coverage (enforced in settings), test-first development
- **Type Safety**: Type hints required for all functions
- **Code Quality**: black, isort, ruff (automated linting)
- **Security**: bandit, safety (automated scanning)
- **CI/CD**: Automated enforcement on every PR
- **Documentation**: Complete docstrings for all public APIs
- **Writing**: Technical claims only (no marketing language)

See [PROFESSIONAL_QUALITY_STANDARDS.md](https://github.com/circuit-synth/claude-files-inventory/blob/main/PROFESSIONAL_QUALITY_STANDARDS.md) and [SHARED_WRITING_STANDARDS.md](https://github.com/circuit-synth/claude-files-inventory/blob/main/SHARED_WRITING_STANDARDS.md) for complete requirements.

---

## 🎯 Core Development Philosophy

### 1. GitHub Issue-Driven Development

All work starts with a GitHub issue:

1. **Create/select issue** - Check existing issues or create new one
2. **Break down** - Issue describes the goal; we break into small subtasks
3. **Work on subtask** - One focused task at a time
4. **Reference in commits** - `fix: Correct Text class parameters (#238)`
5. **Close with verification** - Issue closes when task is done and tested

**Why:** Keeps work visible, context survives across sessions, enables focused problem-solving.

### 2. Test-First Mentality (MANDATORY)

Never commit untested code.

```python
# Pattern for every task:

# Step 1: Write failing test
def test_potentiometer_component():
    pot = Potentiometer("R1", value="10k")
    assert pot.reference == "R1"
    assert pot.footprint  # Should auto-select

# Step 2: Implement minimal code to pass test
class Potentiometer(Component):
    def __init__(self, reference, value):
        super().__init__(reference)
        self.value = value
        self._select_footprint()

# Step 3: Run test - verify it passes
# Step 4: Add regression tests for edge cases
# Step 5: Verify test coverage >80%
```

**Why:** Test documents expected behavior, provides safety net for changes, catches regressions.

### 3. Iterative Cycle Pattern (CORE)

**Do NOT write large amounts of code, then test once.**
**DO work in tight iterative cycles: Add Logs → Run → Observe → Repeat**

```
Cycle: Add Logs → Run → Observe → Repeat
Target: 10-20 cycles per task, each 2-3 minutes
Result: Deep understanding + confident implementation
```

#### The Cycle Process:

1. **Add strategic logging** to code area you're studying
2. **Run the code/test** immediately
3. **Observe log output** to understand behavior
4. **Refine understanding** based on what logs revealed
5. **Make small change** (1-5 lines) based on observation
6. **Run again immediately** - repeat cycle

#### Example: 8-Cycle Bug Investigation

```
Issue #238: Text class parameters incorrect

Cycle 1: Add logs to see what's being called
  ↓ Run test, observe logs
  ↓ Logs show: Text(position, text) is being called

Cycle 2: Add logs to constructor
  ↓ Run test, observe
  ↓ Logs show: Parameters received in wrong order

Cycle 3: Check Text.__init__ signature
  ↓ Run code
  ↓ Logs show: Signature expects (text, at=position) but caller uses (position, text)

Cycle 4: Find all callers of Text
  ↓ Run grep
  ↓ Found: 3 places calling with wrong parameter order

Cycle 5: Write test with correct parameter order
  ↓ Run test - FAILS (parameters still swapped)

Cycle 6: Fix first caller
  ↓ Run test - PASSES
  ↓ Logs show: Parameters now in correct order

Cycle 7: Fix remaining callers
  ↓ Run all tests - PASS
  ↓ All logs show correct order

Cycle 8: Clean up debug logs, verify production logs
  ↓ Remove 🔍 temporary logs
  ↓ Run test - still passing, output clean
```

**Total time:** ~30 minutes in 8 tight cycles.

---

## 🔬 Log-Driven Investigation

Logs are your development tool. Use them to understand behavior, not to guess.

### Strategic Logging Pattern

Follow standard Python logging best practices. **No emojis in logs.**

```python
# Temporary investigation logs (remove after understanding)
logger.debug(f"CYCLE {n}: Investigating {variable_name} = {value}")
logger.debug(f"CYCLE {n}: Function entry with args: {args}")
logger.debug(f"CYCLE {n}: Branch taken: {branch_info}")

# Permanent operational logs (keep)
# These provide production insight and debugging capability
logger.info(f"Generated netlist for {circuit.name}")
logger.debug(f"Selected footprint {footprint} for {component.reference}")
logger.warning(f"Component {ref} missing footprint, using default")
logger.error(f"Failed to validate {ref}: {error}")
```

**Log Levels:**
- **DEBUG:** Development insights, detailed state inspection
- **INFO:** Important state transitions, user-visible operations
- **WARNING:** Recoverable issues, deprecated API usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [circuit-synth/circuit-synth](https://github.com/circuit-synth/circuit-synth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
