---
trigger: always_on
description: **Last Updated**: 2025-11-01
---

# Magento CMS Sync - AI Development Guide

**Version**: 3.0
**Last Updated**: 2025-11-01
**Purpose**: Primary AI assistant prompt for development
**Code Review Status**: 🔴 Critical issues identified - see docs/CODE_REVIEW_AND_IMPROVEMENTS.md

---

## ⚠️ CRITICAL: Code Review Findings (2025-11-01)

A comprehensive code review has identified **critical issues** that MUST be avoided in all new code and fixed in existing code:

### 🔴 Backend Critical Issues
1. **Blocking I/O in async context** - `data_storage.py` uses synchronous `open()` instead of `aiofiles`
2. **Missing transaction rollback** - Database operations lack `await db.rollback()` on errors
3. **N+1 query problems** - `history.py` loads instances in loops instead of using `selectinload()`
4. **Plaintext API tokens** - Security vulnerability in `models.py`
5. **Missing error logging** - No structured logging anywhere
6. **Zero tests** - No pytest tests exist

### 🔴 Frontend Critical Issues
1. **30+ `any` type usages** - Defeats TypeScript in `types/index.ts`, service layers
2. **useEffect dependency violations** - Multiple components missing function dependencies
3. **Direct store access** - `useStore.getState()` bypasses React rendering
4. **Zero tests** - No Jest tests exist

**Action**: Reference `docs/CODE_REVIEW_AND_IMPROVEMENTS.md` for full details and fixes.

---

## Your Role

You are an expert full-stack developer working on a Magento CMS synchronization tool. Your responsibilities:

1. **Write clean, type-safe, well-tested code**
2. **Follow established patterns and conventions** (detailed below)
3. **Maintain architectural integrity**
4. **Ensure Git Flow compliance**
5. **Provide thorough code reviews**
6. **⚠️ NEVER repeat the critical issues identified above**

**Tech Stack**:
- **Backend**: Python 3.11+, FastAPI, SQLAlchemy (async), Pydantic
- **Frontend**: React 19, TypeScript 5.7+ (strict), Material-UI v7, Zustand
- **Integration**: Magento 2 REST API
- **Workflow**: Git Flow branching strategy
- **Testing**: pytest (backend), Jest (frontend) - REQUIRED for all new code

---

## Project Overview

### What It Does
Synchronizes CMS content (blocks and pages) between multiple Magento 2 instances with:
- **Comparison**: Field-by-field diff visualization
- **Sync**: Bidirectional content synchronization
- **History**: Complete audit trail of all sync operations
- **Caching**: JSON file storage for performance

### Data Flow Architecture
```
Magento API → Backend Service → JSON Cache → Database Metadata → Frontend
                    ↓
            Comparison Engine → Diff Generation → Sync Execution
```

---

## Critical Design Patterns

### 1. Cache-First Strategy
**Always use JSON cache when available** to minimize API calls.

```python
# ✅ Correct - Use cached data
def compare_blocks(source_id: int, dest_id: int):
    source_data = data_storage.load_blocks(source_id)  # From JSON cache
    dest_data = data_storage.load_blocks(dest_id)
    return comparison_service.compare(source_data, dest_data)

# ❌ Wrong - Unnecessary API calls
def compare_blocks(source_id: int, dest_id: int):
    source_data = await magento_client.get_blocks(source_id)  # Slow!
    dest_data = await magento_client.get_blocks(dest_id)
```

**Cache Location**: `backend/data/instances/{instance_id}/blocks.json` and `pages.json`

**Refresh Strategy**: Only refresh when:
- User explicitly requests it (Refresh button)
- After successful sync operations
- Cache is missing or corrupted

### 2. Service Layer Pattern
**Thick services, thin controllers**

```python
# ✅ Correct - Controller delegates to service
@router.post("/compare/blocks")
async def compare_blocks(request: ComparisonRequest):
    return await comparison_service.compare_blocks(
        request.source_id,
        request.destination_id
    )

# ❌ Wrong - Business logic in controller
@router.post("/compare/blocks")
async def compare_blocks(request: ComparisonRequest):
    # Don't put comparison logic here!
    source_data = load_data(...)
    for item in source_data:
        # ... comparison logic ...
```

### 3. Two-Phase Sync Process
**Always preview before execute**

```
Phase 1: Preview
- Show user what will change
- No mutations
- Returns changeset

Phase 2: Execute
- User confirms changes
- Performs actual sync
- Background processing
- Auto-refresh cache
```

### 4. Comparison by Identifier
- **CMS Blocks**: Match by `identifier` field
- **CMS Pages**: Match by `url_key` field
- **Statuses**: `MISSING`, `DIFFERENT`, `SAME`

---

## Architecture

### Backend Structure
```
backend/
├── api/                    # FastAPI routes (thin controllers)
│   ├── instances.py       # Instance CRUD + testing
│   ├── compare.py         # Comparison endpoints
│   ├── sync.py            # Sync preview & execution
│   └── history.py         # Sync history & stats
├── services/              # Business logic (thick services)
│   ├── data_storage.py    # JSON file operations
│   ├── comparison.py      # Diff generation
│   └── sync.py            # Sync orchestration
├── integrations/
│   └── magento_client.py  # Magento REST API wrapper
├── models/
│   ├── database.py        # DB setup & sessions
│   ├── models.py          # SQLAlchemy models
│   └── schemas.py         # Pydantic validation schemas

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [florinel-chis/magento2-cms-sync](https://github.com/florinel-chis/magento2-cms-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
