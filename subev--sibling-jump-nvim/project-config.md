---
trigger: always_on
description: **sibling-jump.nvim** is a Neovim plugin for context-aware code navigation using Tree-sitter.
---

# AI Agent Development Instructions

## Overview

**sibling-jump.nvim** is a Neovim plugin for context-aware code navigation using Tree-sitter.

**Primary languages**: TypeScript, JavaScript, JSX, TSX, Lua  
**Partial support**: Java, C, C++, C#, Python

**Read `ARCHITECTURE.md` first** to understand the two-mode navigation system (sibling navigation vs block-loop).

## Critical Rules

### 1. Testing is Non-Negotiable

```bash
# ALWAYS run tests after ANY change
bash tests/test_runner.sh

# All tests MUST pass - no exceptions
```

**Why**: Complex Tree-sitter AST interactions mean small changes can break navigation in unexpected ways.

### 2. Never Break Backward Compatibility

Public API in `init.lua` must remain stable:
- `M.setup(opts)` - Same signature
- `M.jump_to_sibling(opts)` - Same signature  
- User commands: `SiblingJumpBuffer*` - Same names
- Configuration options: Same names and types

**Internal modules can change freely**, but public API is sacred.

### 3. Don't Use Plenary for Tests

❌ **DO NOT** add plenary.nvim as a test dependency  
✅ **DO** use the existing direct test runner (`tests/run_tests.lua`)

**Why**: Plenary's test isolation breaks Tree-sitter parser access.

### 4. Module Naming Convention

- **Repository**: `sibling-jump.nvim` (hyphen)
- **Lua modules**: `sibling_jump` (underscore)
- **Always use**: `require("sibling_jump.module")`

### 5. Always Ask Before Committing

**NEVER auto-commit changes.** Always:
1. Show summary of changes
2. Explain what was fixed/changed and why
3. Wait for explicit user approval
4. Only commit after receiving approval

## Development Workflow

### Making Changes

```bash
# 1. Create feature branch (if not on one)
git checkout -b feature/your-feature

# 2. Make changes & run tests continuously
bash tests/test_runner.sh

# 3. Ask user before committing (ALWAYS!)

# 4. Commit with user approval
git add -A
git commit -m "feat: description"

# 5. Push when tests pass
git push origin feature/your-feature
```

### Test-Driven Development

**IMPORTANT**: Write failing test BEFORE fixing bugs or adding features.

1. **Add fixture** (if needed): `tests/fixtures/your_test.ts`
2. **Write failing test**: In `tests/run_tests.lua` or `tests/run_block_loop_tests.lua`
   ```lua
   test("Description", function()
     vim.cmd("edit " .. fixtures_dir .. "your_test.ts")
     vim.api.nvim_win_set_cursor(0, {5, 0})
     sibling_jump.jump_to_sibling({ forward = true })
     local pos = vim.api.nvim_win_get_cursor(0)
     assert_eq(7, pos[1], "Should jump to line 7")
   end)
   ```
3. **Run test** - Verify it fails: `bash tests/test_runner.sh`
4. **Implement fix/feature** - Make test pass
5. **Run all tests** - Ensure no regressions

**Why test-first?**
- Proves the bug exists
- Specifies correct behavior
- Prevents future regressions
- Validates the fix works

## Common Tasks

### Debugging Navigation Issues

1. **Check Tree-sitter structure**: Use `:InspectTree` (Neovim 0.9+) or `:TSPlaygroundToggle`
2. **Add temporary debug prints**:
   ```lua
   print("Node type:", node:type())
   print("Parent:", parent and parent:type() or "nil")
   ```
3. **Check if node is meaningful**:
   ```lua
   :lua local utils = require("sibling_jump.utils")
   :lua print("Meaningful?", utils.is_meaningful_node(vim.treesitter.get_node()))
   ```
4. **Remove debug prints after fixing**

### Adding Block-Loop Handler

See `ARCHITECTURE.md` → Extension Points. Basic structure:

1. Create `lua/sibling_jump/block_loop/your_handler.lua`:
   ```lua
   local M = {}
   
   function M.detect(node, cursor_pos)
     -- Walk AST to detect if cursor is in this construct
     -- Return: detected (bool), context (table with positions)
   end
   
   function M.navigate(context, cursor_pos, mode)
     -- Find next/prev position based on current cursor
     -- Return: {row, col} or nil at boundaries
   end
   
   return M
   ```

2. Add to `block_loop.lua` handlers list in priority order (most specific first)

3. Add tests to `tests/run_block_loop_tests.lua`

4. Run tests: `bash tests/test_runner.sh`

### Adding Language Support

1. Use `:InspectTree` to find node type names
2. Add to `config.lua` MEANINGFUL_TYPES:
   ```lua
   -- Python
   "function_definition",
   "class_definition",
   "for_statement",
   ```
3. Test thoroughly with fixtures

## Code Style

- **Indentation**: 2 spaces
- **Line length**: ~100 chars (no hard limit)
- **Comments**: Explain *why*, not *what*
- **Functions**: `snake_case` for local, `M.snake_case` for exports
- **Early returns**: Prefer guard clauses over deep nesting
- **Nil checks**: Always check nodes before calling methods

### Avoid Deep Nesting

❌ **Bad**:
```lua
if condition1 then
  if condition2 then
    if condition3 then
      -- work
    end
  end
end
```

✅ **Good**:
```lua
if not condition1 then return end
if not condition2 then return end
if not condition3 then return end
-- work
```

### Tree-sitter Common Patterns

**Walking up the tree:**
```lua
local current = node
local depth = 0
while current and depth < MAX_DEPTH do
  if current:type() == "target_type" then
    return current
  end
  current = current:parent()
  depth = depth + 1
end
```

**Getting position:**
```lua

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subev/sibling-jump.nvim](https://github.com/subev/sibling-jump.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
