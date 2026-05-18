---
trigger: always_on
description: My GitHub 2025 is a web application that generates beautiful statistics and visualizations of user activities on GitHub for any given year (2008-2025). The application uses Flask (Python 3.12+), GitHub OAuth for authentication, and GitHub's GraphQL/REST APIs to fetch and analyze user activity data.
---

# GitHub Copilot Instructions for My GitHub 2025

## Project Overview

My GitHub 2025 is a web application that generates beautiful statistics and visualizations of user activities on GitHub for any given year (2008-2025). The application uses Flask (Python 3.12+), GitHub OAuth for authentication, and GitHub's GraphQL/REST APIs to fetch and analyze user activity data.

## Project Structure

```
/config       - Configuration files (app settings, constants)
/models       - SQLAlchemy database models
/routes       - Flask blueprints (auth, main, api)
/services     - Business logic (GitHub API, data processing, database)
/utils        - Helper functions (context builders, error handlers, logging)
/templates    - Jinja2 HTML templates
/static       - Static assets (CSS, JavaScript, images)
```

## Technology Stack

### Backend

- **Framework**: Flask (Python 3.12+)
- **Database**: SQLAlchemy with SQLite
- **Authentication**: GitHub OAuth 2.0
- **APIs**: GitHub REST API and GraphQL API
- **HTTP Client**: requests library with tenacity for retries
- **Time Handling**: pytz for timezone support
- **Environment**: python-dotenv for configuration

### Frontend

- **Templating**: Jinja2
- **JavaScript**: Vanilla JS
- **Charts**: Chart.js
- **Styling**: CSS (no framework)

## Development Commands

```bash
# Run the application
python main.py

# Run with specific environment
export FLASK_ENV=development
python app.py

# Lint code (using pylint)
pylint app.py routes/ services/ utils/ models/ config/

# Format code (if using black)
black .

# Type checking (if using mypy)
mypy .
```

## Commit Message Guidelines

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for semantic commit messages:

### Format

```plaintext
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that don't affect code meaning (formatting, whitespace)
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Performance improvement
- **test**: Adding or updating tests
- **build**: Changes to build system or dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files

### Examples

```
feat(github): add support for organization statistics
fix(auth): resolve OAuth callback redirect issue
docs(readme): update Python version requirement
refactor(data): extract commit analysis to separate function
perf(api): optimize GraphQL query to reduce API calls
chore(deps): update Flask to version 3.0
```

### Best Practices

- Use present tense (`add` not `added`)
- Use imperative mood (`move` not `moves`)
- Keep first line under 72 characters
- Reference issues/PRs in footer: `Fixes #123`
- Break long descriptions into body paragraphs

## Code Style Guidelines

### General Python

- Follow PEP 8 style guidelines
- Maximum line length: 100 characters (configured in `.pylintrc`)
- Use type hints for function parameters and return values
- Write docstrings for all modules, classes, and functions
- Use meaningful variable and function names

### Import Organization

```python
# Standard library imports
import os
from datetime import datetime

# Third-party imports
from flask import Flask, request
import requests

# Local application imports
from config.config import Config
from services.github_service import GitHubService
```

### Flask Code Style

- Use blueprints for route organization
- Implement proper error handling
- Use Flask's application factory pattern
- Leverage context variables (`current_app`, `g`)
- Keep route handlers thin - delegate to services

### Type Hints

```python
from typing import Dict, List, Optional

def fetch_user_data(username: str, year: int) -> Dict[str, any]:
    """Fetch GitHub user data for a specific year."""
    pass

def calculate_statistics(data: List[Dict]) -> Optional[Dict[str, int]]:
    """Calculate statistics from raw data."""
    pass
```

### Docstrings

Use Google-style docstrings:

```python
def process_commits(commits: List[Dict], year: int) -> Dict[str, int]:
    """Process commit data to extract statistics.
    
    Args:
        commits: List of commit dictionaries from GitHub API
        year: The year to filter commits by
        
    Returns:
        Dictionary containing commit statistics
        
    Raises:
        ValueError: If year is outside valid range (2008-2025)
    """
    pass
```

## Key Features to Understand

### GitHub OAuth Flow

1. User clicks "Sign in with GitHub"
2. Redirect to GitHub authorization page
3. User authorizes the application
4. GitHub redirects back with authorization code
5. Exchange code for access token
6. Store token securely in database
7. Use token for GitHub API requests

### Data Fetching Strategy

- **GraphQL API**: Primary method for fetching user activity data
- **REST API**: Fallback for certain endpoints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WCY-dt/my-github-2025](https://github.com/WCY-dt/my-github-2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
