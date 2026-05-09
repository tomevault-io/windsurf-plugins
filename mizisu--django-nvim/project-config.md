---
trigger: always_on
description: **Always response in Korean.**
---

# Agent Guidelines for django.nvim

**Always response in Korean.**

## Project Overview

Neovim plugin for Django development written in Lua (plugin) with Python scripts for Django introspection.

## Build/Test Commands

- No formal test suite or build process - this is a Neovim plugin
- Test manually: Open Neovim in project root with `:DjangoViews` or `:DjangoModels`
- Python scripts: Run with `DJANGO_SETTINGS_MODULE=project.settings uv run python scripts/<script_name>.py`
  - Example: `DJANGO_SETTINGS_MODULE=project.settings uv run python scripts/get_model_field_relations.py`
- Django test project included in `project/` directory for testing

### Unit Tests

Unit tests can run without Neovim:

```bash
# Test parse_filter_context() function
nvim --headless --noplugin -u NONE -l test_parse.lua
```

### Manual Integration Testing

Integration testing in a real Django project:

#### 1. Project Setup

```bash
# Activate Python environment
source .venv/bin/activate

# Verify Django server can run
python manage.py check
```

#### 2. Neovim Configuration

Verify plugin is configured in `init.lua` or lazy.nvim setup:

```lua
{
  "mizisu/django.nvim",
  dependencies = {
    "folke/snacks.nvim",
    {
      "saghen/blink.cmp",
      opts = {
        sources = {
          default = { "django" },
          providers = {
            django = {
              name = "Django",
              module = "django.features.completions.blink",
              async = true,
            },
          },
        },
      },
    },
  },
}
```

#### 3. Test Scenarios

Open a Python file in the project (e.g., `project/blog/views.py`):

**Scenario 1: Basic Field Completion**
```python
from blog.models import Post

def test_view(request):
    # Place cursor after ( and trigger autocomplete
    posts = Post.objects.filter(
    #                           ^ Press Ctrl+Space here
```

**Expected Results:**
- `title` (CharField)
- `title__` (for lookup operators)
- `author__` (ForeignKey traversal)
- `published_date` (DateTimeField)
- Lookup operators for each field (`title__exact`, `title__icontains`, etc.)

**Scenario 2: Relationship Traversal**
```python
posts = Post.objects.filter(author__
#                                   ^ Autocomplete here
```

**Expected Results:**
- `username` (User model field)
- `email` (User model field)
- Related lookup operators

**Scenario 3: Lookup Operators**
```python
posts = Post.objects.filter(title__
#                                  ^ Autocomplete here
```

**Expected Results:**
- `exact` (exact match)
- `iexact` (case-insensitive match)
- `contains` (contains)
- `icontains` (case-insensitive contains)
- Documentation shows SQL templates for each item

**Scenario 4: Partial Typing**
```python
posts = Post.objects.filter(tit
#                              ^ Autocomplete here
```

**Expected Results:**
- Only completion items starting with `title` shown

**Scenario 5: Multiple Filter Arguments**
```python
posts = Post.objects.filter(status='published', 
#                                              ^ Autocomplete here
```

**Expected Results:**
- New field completions (same as first scenario)

#### 4. Debugging

If autocomplete doesn't work:

1. **Check LSP Type Information:**
```python
# Place cursor on Post.objects and hover (K)
Post.objects.filter()
#    ^ Should show QuerySet[Post] type
```

2. **Run Python Script Directly:**
```bash
cd /path/to/django.nvim
source .venv/bin/activate
DJANGO_SETTINGS_MODULE=project.settings python scripts/get_model_field_relations.py | python -m json.tool | head -n 100
```

3. **Check Neovim Logs:**
```vim
:messages
```

4. **Clear Cache:**
```vim
:DjangoClearAllCache
```

### Expected Output Structure

Python scripts should output JSON in this structure:

```json
{
  "models": {
    "Post": {
      "app_label": "blog",
      "fields": {
        "title": {
          "type": "CharField",
          "definition": "models.CharField(max_length=200)",
          "null": false,
          "blank": false,
          "max_length": 200
        },
        "author": {
          "type": "ForeignKey",
          "definition": "models.ForeignKey(User, on_delete=models.CASCADE)",
          "null": false,
          "blank": false,
          "related_model": "User",
          "related_app": "auth",
          "traversable": true,
          "related_query_name": "post_set"
        }
      }
    }
  },
  "lookup_map": {
    "CharField": ["exact", "iexact", "contains", "icontains", ...],
    "DateTimeField": ["exact", "gt", "gte", "lt", "lte", ...]
  },
  "base_lookups": ["exact", "isnull", "in"],
  "lookup_metadata": {
    "exact": {
      "description": "Exact match",
      "sql_template": "WHERE column = %s"
    },
    ...
  }
}
```

## Code Style

### Lua (Plugin Code)

- Use tabs for indentation
- Local module pattern: `local M = {}` and `return M`
- No trailing newlines in files
- Function naming: snake_case (e.g., `get_python_path()`, `is_django_project()`)
- Variable naming: snake_case (e.g., `cache_dir`, `manage_py`)
- Private methods: prefix with `__` and declare on `M` table (e.g., `function M.__internal_helper()`)
- Method order: public methods at top, private methods at bottom based on dependency order
- Use `vim.fn`, `vim.api`, `vim.bo` namespaces - no deprecated `vim.fn.*` calls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mizisu/django.nvim](https://github.com/mizisu/django.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
