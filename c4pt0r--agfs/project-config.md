---
trigger: always_on
description: Use when: Need backward compat with dict operations but want rich internal representation.
---

# AGENTS.md - agfs-shell (Python Shell & AST Engine)

## OVERVIEW
`agfs-shell` is a pure-Python shell implementation featuring a full Lexer/Parser/AST pipeline. Unlike simple wrappers, it manages its own state, control flow, and job control, using the `pyagfs` SDK for filesystem operations.

## STRUCTURE
```
agfs_shell/
├── shell.py            # Central state (CWD, env, functions, aliases, jobs)
├── executor.py         # AST Executor; handles control flow via exceptions
├── parser.py           # Lexer and Parser (Shlex-based + custom logic)
├── ast_nodes.py        # AST node definitions (For, If, Command, etc.)
├── pipeline.py         # Process chaining and threading logic
├── process.py          # Command execution context (stdin/out/err, args)
├── commands/           # Built-in command implementations
└── command_decorators.py # @command and @register_command decorators
```

## WHERE TO LOOK
- **Control Flow**: `executor.py` and `control_parser.py`. Uses `BreakException`, `ContinueException`, and `ReturnException` for flow control.
- **Variable Expansion**: `Shell._expand_variables` in `shell.py`.
- **Job Management**: `JobManager` in `job_manager.py`.
- **Path Resolution**: Handled by `@command(needs_path_resolution=True)` which resolves AGFS paths before the command runs.

## ADDING COMMANDS
Commands are Python functions decorated for registration. They receive a `Process` object containing the execution context.

```python
from ..process import Process
from ..command_decorators import command
from . import register_command

@command(needs_path_resolution=True)
@register_command('mycmd')
def cmd_mycmd(process: Process) -> int:
    """
    My custom command description
    """
    # Access arguments: process.args
    # Write output: process.stdout.write(b"hello\n")
    # Access AGFS: process.filesystem.list_directory("/")
    return 0  # Exit code
```

## CONTROL FLOW
The shell uses a recursive descent parser to build an AST.
1. `parser.py` splits tokens and identifies `if`, `for`, `while`, and `|` (pipelines).
2. `ShellExecutor` (in `executor.py`) traverses the AST.
3. Nested blocks (e.g., functions, loops) manage `_function_depth` and `_local_` variables in the `Shell.env` to maintain scope isolation.

## ANTI-PATTERNS
- **NEVER** use `os.path` or `open()` for AGFS paths. Always use `process.filesystem`.
- **NEVER** print directly to `sys.stdout`. Use `process.stdout.write()`.
- **DO NOT** use `subprocess.run()` for built-ins; use the `Process` abstraction to keep everything within the interpreter.
- **AVOID** blocking operations in the main thread during pipeline execution.

---

## REFACTORING LESSONS LEARNED

### Phase 1: Testing Infrastructure & Documentation (2026-01-17)

**Objective:** Establish comprehensive test suite and document current architecture before refactoring.

#### Key Accomplishments
1. ✅ **Created MockFileSystem** - In-memory filesystem simulation for testing
2. ✅ **Added 75+ new tests** - Increased from 16 to 92 total tests (80 passing)
3. ✅ **Achieved 20% code coverage** - Baseline established for future refactoring
4. ✅ **Documented architecture** - Complete ARCHITECTURE.md and REFACTORING.md

#### Lessons Learned

##### L1.1: Mock Filesystem Design Pattern
**What We Built:**
- Dictionary-based in-memory filesystem (`MockFileSystem` in `tests/conftest.py`)
- Supports files, directories, and metadata
- Simulates errors (FileNotFoundError, PermissionDenied, etc.)

**Why It Works:**
- Tests run ~10x faster (no network I/O to AGFS server)
- Reproducible test environments
- Easy to set up complex filesystem states

**Pattern to Reuse:**
```python
@pytest.fixture
def mock_filesystem():
    fs = MockFileSystem()
    fs.write_file('/test.txt', b'content')
    fs.create_directory('/testdir')
    return fs
```

**Key Insight:** Focus on error *types* (FileNotFoundError) rather than exact error *messages* when matching AGFS behavior. Messages may vary, but error types are consistent.

##### L1.2: Stream Output Access Pattern
**What We Learned:**
- Initially tried `process.stdout.buffer.getvalue()` ❌
- Correct approach: `process.get_stdout()` ✅
- Process class provides helper methods for common operations

**Why This Matters:**
- The API is designed to abstract stream internals
- Direct buffer access couples tests to implementation details
- Helper methods provide consistent interface

**Action:** Always check existing tests (`test_builtins.py`) for established patterns before writing new tests.

##### L1.3: Test Organization Strategy
**What Worked:**
- Organized tests by component (shell core, process, expression, etc.)
- Grouped related tests in classes (TestEnvironmentVariables, TestPipelines)
- Descriptive test names that explain intent

**Example:**
```python
class TestEnvironmentVariables:
    def test_get_environment_variable(self, mock_filesystem):
        """Test retrieving environment variables."""
        # Test implementation

    def test_set_environment_variable(self, mock_filesystem):
        """Test setting environment variables."""
        # Test implementation
```

**Benefits:**
- Easy to run specific test groups
- Self-documenting test suite
- Clear coverage gaps visible at a glance

##### L1.4: Coverage vs Quality Tradeoff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c4pt0r/agfs](https://github.com/c4pt0r/agfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
