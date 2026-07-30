---
trigger: always_on
description: ComfyUI custom node providing utilities for model management, prompting, metadata handling, and workflow enhancement.
---

# Sage Utils ComfyUI Custom Node Development Guide

## Project Overview
ComfyUI custom node providing utilities for model management, prompting, metadata handling, and workflow enhancement.

Refer to `okf/developer/project_overview.md` for the project directory layout and key asset locations.

## Python Standards
**Naming:** `snake_case` functions/variables, `PascalCase` classes, `ALL_CAPS` constants  
**Formatting:** 4-space indent, f-strings, type hints where possible  
**Structure:** Group nodes by function, utils for shared code  
**Error Handling:** Try/except for I/O, tuple unpacking for multiple returns  
**Best Practices:** List comprehensions, context managers, single responsibility

## JavaScript Standards  
**Naming:** `camelCase` variables/functions, `PascalCase` classes, `ALL_CAPS` constants  
**Formatting:** 2-space indent, semicolons, single quotes, ES6 modules  
**Best Practices:** Arrow functions, array methods, strict equality, template literals

**File Organization:**
Refer to `okf/architecture/backend_js_architecture.md` for frontend JavaScript structure and sidebar organization.

**Code Structure Guidelines:**
- Prefer multiple shorter files (refactor when approaching 1000 lines)
- Maximize code reuse through modular design
- Create generic UI components for reusability across different contexts
- Maintain clear separation between frontend and backend code
- Split long functions into smaller, focused functions
- Use composition over inheritance for component design

**JavaScript Validation:**
Always validate JavaScript files after making large changes using Node.js syntax checking:
- Single file: `node -c path/to/file.js`
- All JS files: `find js -name "*.js" -exec node -c {} \; && echo "All files valid"`
- Specific directory: `find js/sidebar -name "*.js" -exec node -c {} \;`
This catches syntax errors, missing imports, and basic structural issues before testing in ComfyUI.

## Node Development
- Place nodes in appropriate `nodes/*.py` module
- Each node class needs docstring with purpose/inputs/outputs
- Register in `__init__.py` CLASS_MAPPINGS and NODE_DISPLAY_NAME_MAPPINGS
- Custom types defined as strings in input/output tuples
- Use `comfyui_sageutils.utils` for shared functionality

**Type Errors to Ignore:**
- `INPUT_TYPES` will have type errors when using custom types as strings (e.g., `"IMAGE"`, `"MODEL"`)
- Lists and tuples in type definitions may not match IO.* types - this is expected ComfyUI behavior
- Do not attempt to "fix" these type errors as they are intentional for ComfyUI's dynamic type system

**Plugin Import Limitations:**
- This is a ComfyUI plugin/custom node, not a standalone project
- Test code may fail with import errors when run independently (outside ComfyUI context)
- Imports like `from comfy.comfy_types.node_typing import ComfyNodeABC` only work when loaded by ComfyUI
- Don't attempt to fix import errors in test files - they require ComfyUI's runtime environment

## Critical Environment Requirements

### Virtual Environment Isolation
- **ALWAYS activate ComfyUI's venv before running Python tests**: `source /home/ai/programs/comfyui/venv/bin/activate` (no dot — named `venv`, not `.venv`)
- Never run Python scripts/tests directly from `comfyui_sageutils/` directory without activating parent comfyui venv
- Missing dependencies will cause test failures even if code is correct
- ComfyUI must be running in same directory as its venv to avoid library conflicts

### Cache File Safety
- User data at `/home/ai/programs/comfyui/user/default/SageUtils/` contains cache files (`sage_cache_info.json`, `sage_cache_hash.json`) that store model information
- **Do NOT modify or delete these cache files during development work on the code**
- If cache gets corrupted/wiped, restore from backup at `/home/ai/programs/comfyui/user/default/SageUtils/backup/`

### JavaScript Error Debugging
- Sidebar errors often lack clear indication of which file caused the problem
- When sidebar fails to load or shows errors:
  1. Check recent modifications (git diff, filesystem timestamps)
  2. Look for syntax errors in modified JS files: `node -c path/to/file.js`
  3. Trace import chains — broken imports often originate in parent files that cascade
  4. Common culprits: missing semicolons, incorrect module paths, unbalanced brackets/braces
- Use browser DevTools to identify error origins when possible

### Python Syntax Validation
- Before running tests or loading into ComfyUI, compile Python files to catch syntax errors early:
  ```bash
  python -m py_compile path/to/script.py
  # Or for all .py files in a directory:
  find . -name "*.py" -exec python -m py_compile {} \;
  ```
- This catches syntax errors without needing imports or dependencies
- Run after making changes before testing in ComfyUI

## Documentation Updates
- Update `README.md` for new features
- Update `pyproject.toml` version for releases
- Create workflow examples with JSON + JPG pairs

**OKF Documentation Guidance:**
- Treat `okf/` as the primary structured documentation bundle for this project.
- When adding or changing architecture, developer, UI, node, example, docs, or tool content, create or update the corresponding `okf/*/index.md` and concept files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcum42/ComfyUI_SageUtils](https://github.com/arcum42/ComfyUI_SageUtils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
