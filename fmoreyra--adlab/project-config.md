---
trigger: always_on
description: - **ALWAYS place imports at the top of the file** (module level)
---

# Cursor Rules - Django/Python Best Practices
# AdLab Laboratory Management System

## 🐍 Python General Best Practices

### Imports
- **ALWAYS place imports at the top of the file** (module level)
- **NEVER use local imports inside functions** unless absolutely necessary for:
  - Avoiding circular imports (rare, document why)
  - Optional dependencies
  - Performance-critical edge cases
- **Group imports** in this order (PEP 8):
  1. Standard library imports
  2. Related third-party imports (Django, etc.)
  3. Local application imports
- **Sort imports alphabetically** within each group
- **Use absolute imports** over relative imports when possible

### Code Style (PEP 8)
- Use **4 spaces** for indentation (never tabs)
- **Maximum line length**: 88 characters (Black formatter standard)
- **Two blank lines** before top-level classes and functions
- **One blank line** between methods in a class
- Use **snake_case** for functions and variables
- Use **PascalCase** for classes
- Use **UPPER_CASE** for constants
- Add **trailing commas** in multi-line collections

### Naming Conventions
- Use **descriptive names**: `calculate_total_price()` not `calc()`
- **Avoid single-letter variables** except for: i, j, k (loops), e (exceptions), f (files)
- **Boolean variables**: prefix with `is_`, `has_`, `can_`, `should_`
- **Private methods**: prefix with single underscore `_internal_method()`

### Documentation
- **Every module** should have a module-level docstring
- **Every function/method** should have a docstring explaining:
  - What it does
  - Parameters (with types)
  - Return value (with type)
  - Exceptions raised
- Use **Google-style** or **NumPy-style** docstrings
- Keep docstrings **up-to-date** with code changes

---

## 🎯 Django Best Practices

### Models
- **Always use verbose_name** and **verbose_name_plural** for models
- **Add help_text** to fields that need clarification
- **Use choices** for fields with limited options (TextChoices, IntegerChoices)
- **Override save()** with super().save() calls, never skip the super call
- **Add __str__()** method to every model (for admin and debugging)
- **Use related_name** in ForeignKey/ManyToMany relationships
- **Add indexes** for fields used in filters/queries
- **Use db_index=True** for fields frequently used in WHERE clauses
- **Avoid nullable CharField/TextField** - use blank=True with default="" instead

### Views
- **Use class-based views** for common patterns (ListView, DetailView, etc.)
- **Keep views thin** - business logic belongs in models or services
- **Always use @login_required** or LoginRequiredMixin for protected views
- **Use get_object_or_404** instead of try/except for single object retrieval
- **Add docstrings** to views explaining their purpose
- **Use proper HTTP status codes** (200, 201, 400, 404, etc.)

### Forms
- **Always use Django forms** - never process raw POST data
- **Use ModelForm** when working with models
- **Add clean_*() methods** for field-level validation
- **Add clean()** for cross-field validation
- **Customize widgets** for better UX (add CSS classes)
- **Use help_text** for user guidance

### Templates
- **Always use {% csrf_token %}** in forms
- **Escape user input** by default (Django does this, don't use |safe unless needed)
- **Use {% static %}** tag for static files
- **Use {% url %}** tag for URLs (never hardcode URLs)
- **Keep logic out of templates** - move to views or template tags

### Admin
- **Register all models** with admin.site.register() or @admin.register()
- **Customize list_display** for better overview
- **Add list_filter** for common filters
- **Add search_fields** for searchable fields
- **Use readonly_fields** for fields that shouldn't be edited
- **Add actions** for bulk operations
- **Customize fieldsets** for better organization

### URLs
- **Use path()** over re_path() when possible
- **Always name your URLs** with name= parameter
- **Use app namespaces** with app_name
- **Group related URLs** using include()
- **Keep URL patterns RESTful** when appropriate

### Settings
- **NEVER commit secrets** to version control
- **Use environment variables** for secrets (.env file)
- **Separate settings** for dev/staging/production (if needed)
- **Use django-environ** or python-decouple for env management
- **Document all custom settings** with comments

### Security
- **Always use HTTPS** in production (set SECURE_SSL_REDIRECT=True)
- **Use CSRF protection** (enabled by default, keep it)
- **Validate and sanitize** all user input
- **Use Django's password hashers** (never store plain passwords)
- **Set secure cookie flags** (HTTPONLY, SECURE, SAMESITE)
- **Use secrets module** for token generation (not random)
- **Add audit logging** for sensitive operations
- **Implement rate limiting** for authentication endpoints
- **Use permissions and groups** for access control
- **Keep dependencies updated** (security patches)

---

## 🎨 Clean Code Practices for Views & Complex Logic

**CRITICAL**: Apply these patterns from the start. Write clean code, not code that needs refactoring.

### Early Returns (Guard Clauses) ✅

**ALWAYS use early returns to avoid deep nesting.**

**Bad Example** ❌:
```python
def login_view(request):
    if request.user.is_authenticated:
        return redirect('/')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fmoreyra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
