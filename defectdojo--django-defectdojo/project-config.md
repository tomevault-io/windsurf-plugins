---
trigger: always_on
description: DefectDojo is a Django application (`dojo` app) for vulnerability management. The codebase is undergoing a modular reorganization to move from monolithic files toward self-contained domain modules.
---

# DefectDojo Development Guide

## Project Overview

DefectDojo is a Django application (`dojo` app) for vulnerability management. The codebase is undergoing a modular reorganization to move from monolithic files toward self-contained domain modules.

## Module Reorganization

### Reference Pattern: `dojo/url/`

All domain modules should match the structure of `dojo/url/`. This is the canonical example of a fully reorganized module.

```
dojo/{module}/
├── __init__.py       # import dojo.{module}.admin  # noqa: F401
├── models.py         # Domain models, constants, factory methods
├── admin.py          # @admin.register() for the module's models
├── services.py       # Business logic (no HTTP concerns)
├── queries.py        # Complex DB aggregations/annotations
├── signals.py        # Django signal handlers
├── [manager.py]      # Custom QuerySet/Manager if needed
├── [validators.py]   # Field-level validators if needed
├── [helpers.py]      # Async task wrappers, tag propagation, etc.
├── ui/
│   ├── __init__.py   # Empty
│   ├── forms.py      # Django ModelForms
│   ├── filters.py    # UI-layer django-filter classes
│   ├── views.py      # Thin view functions — delegates to services.py
│   └── urls.py       # URL routing
└── api/
    ├── __init__.py   # path = "{module}"
    ├── serializer.py # DRF serializers
    ├── views.py      # API ViewSets — delegates to services.py
    ├── filters.py    # API-layer filters
    └── urls.py       # add_{module}_urls(router) registration
```

### Architecture Principles


**services.py is the critical layer**: Both `ui/views.py` and `api/views.py` call `services.py` for business logic. Services accept domain objects and primitives — never request/response objects, forms, or serializers.

**Backward-compatible re-exports**: When moving code out of monolithic files (`dojo/models.py`, `dojo/forms.py`, `dojo/filters.py`, `dojo/api_v2/serializers.py`, `dojo/api_v2/views.py`), always leave a re-export at the original location:
```python
from dojo.{module}.models import {Model}  # noqa: F401 -- backward compat
```
Never remove re-exports until all consumers are updated in a dedicated cleanup pass.

### Current State

Modules in various stages of reorganization:

| Module | models.py | services.py | ui/ | api/ | Status |
|--------|-----------|-------------|-----|------|--------|
| **url** | In module | N/A | Done | Done | **Complete** |
| **location** | In module | N/A | N/A | Done | **Complete** |
| **product_type** | In module | N/A | Done | Done | **Complete** (#14970) |
| **test** | In module | N/A | Done | Done | **Complete** (#14971) |
| **engagement** | In module | In module | Done | Done | **Complete** (#14972) |
| **product** | In module | N/A | Done | Done | **Complete** (#14973) |
| **finding** | In module | N/A (helper.py) | Done | Done | **Complete** (#14974, incl. CWE + BurpRawRequestResponse) |
| **user / system_settings** | In module | N/A | Done | Done | **Complete** (#14981) |
| **endpoint / tool_type / tool_config / tool_product** | In module | N/A | Done | Done | **Complete** (#14982) |
| **survey / benchmark** | In module | N/A | Done | N/A (no API) | **Complete** (#14983) |
| **notes / note_type / file_uploads / reports / risk_acceptance** | In module | N/A | Done | Done | **Complete** (#14986) |
| **regulations / banner / announcement / development_environment / object** | In module | N/A | Done | Partial (API where one exists) | **Complete** (#14987) |

### Monolithic Files Being Decomposed

These files still contain code for multiple modules. Extract code to the target module's subdirectory and leave a re-export stub. (Counts reflect the completed Phase 10 stack; they shrink as branches merge to `dev`.)

- `dojo/models.py` (~645 lines) — re-export hub + the few models intentionally left here (`DojoMeta`, `Network_Locations`, `Sonarqube_Issue`/`Sonarqube_Issue_Transition`, `Check_List`, `Testing_Guide_Category`/`Testing_Guide`, `Language_Type`/`Languages`, `App_Analysis`, `SLA_Configuration`) plus shared utilities (`copy_model_util`, `get_current_date`, `tomorrow`, `UniqueUploadNameProvider`)
- `dojo/forms.py` (~914 lines) — remaining/shared Django forms
- `dojo/filters.py` (~1,376 lines) — remaining/shared filter classes + shared bases
- `dojo/api_v2/serializers.py` (~1,101 lines) — remaining serializers + re-exports for prefetcher discovery
- `dojo/api_v2/views.py` (~901 lines) — remaining viewsets + shared base classes/helpers

---

## Reorganization Playbook

When asked to reorganize a module, follow these phases in order. Each phase should be independently verifiable.

### Phase 0: Pre-Flight (Read-Only)

Before any changes, identify all code to extract:

```bash
# 1. Model classes and line ranges in dojo/models.py
grep -n "class {Model}" dojo/models.py

# 2. Form classes in dojo/forms.py
grep -n "class.*{Module}" dojo/forms.py
grep -n "model = {Model}" dojo/forms.py

# 3. Filter classes in dojo/filters.py
grep -n "class.*{Module}\|class.*{Model}" dojo/filters.py

# 4. Serializer classes
grep -n "class.*{Model}" dojo/api_v2/serializers.py

# 5. ViewSet classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DefectDojo/django-DefectDojo](https://github.com/DefectDojo/django-DefectDojo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
