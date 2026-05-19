---
trigger: always_on
description: This file provides guidance for agentic coding agents operating in the DML V4 repository.
---

# AGENTS.md

This file provides guidance for agentic coding agents operating in the DML V4 repository.

## Build/Lint/Test Commands

### Backend Commands

#### Development Server
```bash
cd backend
python -m app.main                    # Start FastAPI server (port 8000)
```

#### Database Initialization
```bash
cd backend
python app/init_mongodb.py             # Initialize MongoDB with workflow configs
python scripts/init_rbac.py            # Initialize RBAC (roles/permissions)
python scripts/create_user.py          # Create admin user
```

#### Testing (pytest)
```bash
cd backend

# All tests
pytest

# Specific test file
pytest tests/unit/workflow/test_workflow_service.py -v

# Integration tests
pytest tests/integration/ -v

# With coverage
pytest --cov=app

# Specific module
pytest tests/unit/workflow/

# Single test method
pytest tests/unit/workflow/test_workflow_service.py::test_create_item -v
```

#### Linting (flake8)
```bash
flake8                                  # Max line length: 110, max complexity: 12
flake8 app/modules/workflow/service/    # Lint specific directory
flake8 --select=E,W,F                   # Specific error codes
```

#### Dependencies
```bash
pip install -r requirements.txt         # Install Python dependencies
```

### Frontend Commands

#### Development Server
```bash
cd frontend
npm run dev                             # Start Vite dev server (port 3000)
```

#### Build & Preview
```bash
cd frontend
npm run build                           # Production build
npm run preview                         # Preview production build
npm run clean                           # Clean dist directory
```

#### Linting & Type Checking
```bash
cd frontend
npm run lint                            # TypeScript type check (tsc --noEmit)
```

#### Dependencies
```bash
cd frontend
npm install                             # Install Node.js dependencies
```

## Code Style Guidelines

### Backend (Python)

#### General Rules
- **Max line length**: 110 characters
- **Max complexity**: 12 per function/method
- **Python version**: 3.10+ (Python 3.13 compatible)
- **Async patterns**: Use async/await for I/O operations
- **Logging**: Use `from app.shared.core.logger import log` for structured logging

#### Architecture & Layering
- **Strong layering**: API → Service → Repository/Domain (no upward dependencies)
- **API Layer**: FastAPI route handlers in `app/modules/*/api/`
- **Service Layer**: Business logic in `app/modules/*/service/`
- **Repository Layer**: Data access in `app/modules/*/repository/`
- **Domain Layer**: Business rules in `app/modules/*/domain/`

#### Import Organization
```python
# Standard library imports
import re
from typing import Dict, Any, Optional, List

# Third-party imports
from beanie import PydanticObjectId
from pymongo import AsyncMongoClient

# Local application imports (absolute imports from app/)
from app.modules.workflow.repository.models import (
    SysWorkflowConfigDoc,
    BusWorkItemDoc,
)
from app.shared.core.logger import log as logger
```

#### Type Hints & Documentation
```python
def create_item(
    self,
    type_code: str,
    creator_id: str,
    form_data: Optional[Dict[str, Any]] = None
) -> BusWorkItemDoc:
    """
    Create new work item in DRAFT state.

    Args:
        type_code: Business item type (REQUIREMENT, TEST_CASE, etc.)
        creator_id: User ID of the item creator
        form_data: Optional form data for initial item state

    Returns:
        Created work item document

    Raises:
        WorkItemCreationError: If creation fails
    """
```

#### Error Handling
- Use domain-specific exceptions in `app/modules/*/domain/exceptions.py`
- Handle MongoDB-specific errors (DuplicateKeyError, OperationFailure)
- Use proper async exception handling with try/catch blocks
- Log errors with context using structured logger

#### Naming Conventions
- **Classes**: PascalCase (`AsyncWorkflowService`)
- **Functions/methods**: snake_case (`create_item`, `handle_transition`)
- **Variables**: snake_case (`current_state`, `type_code`)
- **Constants**: UPPER_SNAKE_CASE (`MONGO_URI`)
- **Private methods**: prefix with underscore (`_internal_method`)

#### MongoDB/Beanie Patterns
- Use Beanie ODM for MongoDB operations
- Document models inherit from `Document` class
- Always filter by `is_deleted: False` for soft delete pattern
- Use `PydanticObjectId` for document IDs
- Implement proper indexing in document models

### Frontend (React + TypeScript)

#### General Rules
- **Framework**: React 19 + TypeScript + Vite
- **Styling**: TailwindCSS 4
- **Single-file architecture**: All views in `src/App.tsx`
- **TypeScript**: Strict mode with comprehensive type definitions

#### Component Structure
```typescript
/**
 * Component description
 * Explains what this component does
 */

import React, { useState, useCallback } from 'react';
import { Component } from './Component';

// Types first
interface Props {
  data: DataType;
  onAction: (id: string) => void;
}

// Component implementation
const MyComponent: React.FC<Props> = ({ data, onAction }) => {
  const [state, setState] = useState<Type>();

  const handleClick = useCallback((id: string) => {
    onAction(id);
  }, [onAction]);

  return (
    <div className="...">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hankerbiao/omnistate](https://github.com/hankerbiao/omnistate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
