---
trigger: always_on
description: This is a suite of Python linters for validating and fixing [AutoPkg](https://github.com/autopkg/autopkg) recipes. AutoPkg recipes are XML plist or YAML files that automate software packaging for macOS. Each linter is a standalone script in its own directory that detects and fixes specific issues in recipe files.
---

# AutoPkg Recipe Linter Suite - AI Coding Agent Instructions

## Project Overview

This is a suite of Python linters for validating and fixing [AutoPkg](https://github.com/autopkg/autopkg) recipes. AutoPkg recipes are XML plist or YAML files that automate software packaging for macOS. Each linter is a standalone script in its own directory that detects and fixes specific issues in recipe files.

**Architecture**: Unified CLI runner (`autopkg-linter.py`) + 16 independent linter modules, each following the same pattern.

## Critical Environment Requirement

⚠️ **All scripts MUST run using AutoPkg's Python**: `/usr/local/autopkg/python`

Every script includes `verify_environment()` that checks `sys.executable.startswith('/usr/local/autopkg')`. This is not optional - AutoPkg recipes require AutoPkg's Python environment with its specific dependencies (plistlib, etc.).

## Linter Architecture Pattern

Each linter follows this structure:
```
LinterName/
├── LinterName.py          # Main script with main() function
├── README.md              # Detailed documentation
└── __pycache__/          # Python bytecode cache
```

### Standard Linter Structure

Every linter script contains:
1. **Shebang**: `#!/usr/local/autopkg/python`
2. **verify_environment()**: Validates Python path
3. **clean_path()**: Handles drag-and-drop paths with escaped spaces
4. **process_plist_recipe()**: Logic for XML plist recipes
5. **process_yaml_recipe()**: Logic for YAML recipes (if applicable)
6. **main()**: Interactive prompt for recipe directory, then scans/processes files

### Recipe Format Handling

- **Plist recipes** (`.recipe`): XML plist format using `plistlib`
- **YAML recipes** (`.yaml`): YAML format using `yaml` module
- Most linters support both formats with separate processing functions
- Some operations (e.g., XML escaping) only apply to plist format

## Suite Runner Integration

[autopkg-linter.py](autopkg-linter.py) dynamically loads and runs linters:

1. **get_available_linters()**: Returns list of (number, name, directory, script, description) tuples
2. **load_linter_module()**: Uses `importlib.util.spec_from_file_location()` to load scripts
3. **run_linter()**: Temporarily overrides `sys.argv` and `__builtins__.input` to inject recipe directory and auto-answer prompts in bulk mode
4. **Modes**: 
   - `bulk`: Auto-provides recipe directory + default answers to all prompts
   - `interactive`: Provides recipe directory, then allows user interaction

### Adding New Linters to Suite

Edit `get_available_linters()` in [autopkg-linter.py](autopkg-linter.py):
```python
linters = [
    (16, "NewLinterName", "NewLinterDirectory",
     "NewLinterScript.py",
     "Brief description of what it does"),
    # ... existing linters
]
```

Numbers should be sequential, directory must exist, script must have `main()` function.

## Common Patterns

### Path Handling
```python
def clean_path(path):
    """Handles drag-and-drop paths with backslash-escaped spaces"""
    path = path.strip().strip('"').strip("'")
    path = path.replace(r'\ ', ' ')
    path = path.rstrip(' ').rstrip('/')
    path = os.path.expanduser(path)
    return path
```

### Recipe Discovery
Standard pattern across all linters:
```python
recipe_files = []
for root, dirs, files in os.walk(recipe_dir):
    for file in files:
        if file.endswith(('.recipe', '.yaml')):
            recipe_files.append(Path(root) / file)
```

### Modification Tracking
Most linters return `(modified: bool, changes: list)` from processing functions to report what changed.

## Key Linter Examples

### DetabChecker Pattern (Whitespace Fixes)
- Uses string manipulation on raw file content
- Tracks line-by-line changes
- Processes both plist and YAML identically
- See [DetabChecker/DetabChecker.py](DetabChecker/DetabChecker.py)

### MinimumVersionChecker Pattern (Data Fetching)
- Fetches live data from GitHub (`PROCESSOR_VERSIONS_URL`)
- Maintains fallback data (`FALLBACK_PROCESSOR_VERSIONS`)
- Graceful degradation with try/except
- See [MinimumVersionChecker/MinimumVersionChecker.py](MinimumVersionChecker/MinimumVersionChecker.py)

### RecipeAlphabetiser Pattern (Structured Editing)
- Loads plist with `plistlib`, preserves structure
- Uses **dependency-aware topological sorting** for Input dict keys
- Critical: AutoPkg processes Input keys sequentially - if `KEY_A = %KEY_B%`, then `KEY_B` must be defined before `KEY_A`
- Uses `topological_sort_input_keys()` with Kahn's algorithm to respect dependencies
- Uses `alphabetize_dict_with_processor_last()` - special rule: `Processor` key always last in processor dicts
- Writes back with `plistlib.dump()`
- See [RecipeAlphabetiser/RecipeAlphabetiser.py](RecipeAlphabetiser/RecipeAlphabetiser.py)

### AutoPkgXMLEscapeChecker Pattern (Regex Surgery)
- Uses regex to find `<string>...</string>` blocks
- Escapes XML entities: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`
- Critical: Escape `&` first to avoid double-escaping
- Only for plist format (YAML doesn't need XML escaping)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autopkg/dataJAR-recipes](https://github.com/autopkg/dataJAR-recipes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
