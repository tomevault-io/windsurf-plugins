---
trigger: always_on
description: This is a Python web application that provides AI-powered image upscaling using Real-ESRGAN. The application is built with FastAPI and uses the uv package manager for dependency management.
---

# GitHub Copilot Instructions for Image Upscaler

## Project Overview
This is a Python web application that provides AI-powered image upscaling using Real-ESRGAN. The application is built with FastAPI and uses the uv package manager for dependency management.

## Architecture & Structure
- **Package Structure**: Follow Python packaging standards with source code in `src/upscaler/`
- **Framework**: FastAPI for web API with uvicorn server
- **AI Model**: Real-ESRGAN x4plus for image upscaling (lazy loaded)
- **Templates**: HTML templates stored in `src/upscaler/templates/`
- **Static Files**: CSS and JavaScript files in `src/upscaler/static/`
  - `static/css/style.css`: All UI styles
  - `static/js/app.js`: All UI JavaScript logic
- **Tests**: Pytest-based tests in `tests/` directory

## Self-Maintenance
**IMPORTANT**: When making changes to the codebase structure, architecture, or key patterns, **always update this copilot-instructions.md file** to keep it synchronized with the current state of the project. This ensures consistency and helps future development.

## Development Guidelines

### Design Patterns & Software Architecture
- **DRY Principle (Don't Repeat Yourself)**: Extract common functionality into shared utilities
  - Shared logic should be moved to `src/upscaler/utils.py`
  - Both API and CLI should use the same core upscaling logic
- **Single Responsibility**: Each module should have a clear, focused purpose
  - `upscaler.py`: Core upscaling functions and model management
  - `utils.py`: Shared utility functions used across multiple modules
  - `app.py`: API routes and web server
  - `cli.py`: Command-line interface
- **Separation of Concerns**: Keep business logic separate from presentation/interface logic
- **Code Reusability**: Write functions that can be used in multiple contexts
- **Modularity**: Keep modules loosely coupled and easily testable

### Code Style
- **Linting**: Use ruff for Python linting (configured in `pyproject.toml`)
- **Formatting**: Use black for code formatting (line length: 100)
- **HTML/CSS/JS**: Use prettier for formatting (configured in `.prettierrc`)
- **Imports**: Keep imports organized (stdlib → third-party → local)
- **Mobile-First**: Always ensure UI is user-friendly on smartphone devices
  - Use responsive design patterns
  - Test touch interactions and tap targets (minimum 44×44px)
  - Use appropriate font sizes (minimum 16px for inputs to prevent zoom on iOS)
  - Stack layouts vertically on small screens

### Commit Messages
Follow the [Conventional Commits](https://www.conventionalcommits.org/) standard for all commit messages:
- **Format**: `<type>(<scope>): <description>`
- **Types**:
  - `feat`: New feature
  - `fix`: Bug fix
  - `docs`: Documentation changes
  - `style`: Code style changes (formatting, missing semicolons, etc.)
  - `refactor`: Code refactoring without changing functionality
  - `perf`: Performance improvements
  - `test`: Adding or updating tests
  - `build`: Changes to build system or dependencies
  - `ci`: CI/CD configuration changes
  - `chore`: Other changes that don't modify src or test files
- **Scope**: Optional, indicates the area affected (e.g., `api`, `upscaler`, `tests`, `makefile`)
- **Examples**:
  - `feat(api): add batch upload endpoint`
  - `fix(upscaler): resolve memory leak in model loading`
  - `docs: update installation instructions`
  - `test(api): add tests for error handling`
  - `chore(deps): update torch to 2.4.1`

### Testing
- Write unit tests for core logic functions
- Write integration tests for API endpoints using FastAPI TestClient
- Mark slow tests (requiring AI model) with `@pytest.mark.slow`
- Maintain test coverage above 80%
- Run tests with: `make test` or `PYTHONPATH=src pytest tests/ -v`
- **CRITICAL**: When making changes to `src/upscaler/upscaler.py` or `src/upscaler/app.py`, ALWAYS run an end-to-end test to verify that image upscaling works correctly. This ensures the AI model integration is not broken.

### API Design
- Use FastAPI's dependency injection and validation
- Return appropriate HTTP status codes
- Include proper error messages in HTTPException
- Document endpoints with docstrings
- Validate input parameters (e.g., image dimensions: 1-10000 pixels)

### Configuration
- Use `pyproject.toml` for project configuration
- Pin critical dependencies (torch, torchvision) for compatibility
- Use uv for dependency management and lock files

### Performance
- Lazy load the Real-ESRGAN model (only on first upscale request)
- Use CPU by default (GPU support configurable via `gpu_id` parameter)
- Preserve aspect ratio when resizing images

### Makefile Commands
- `make install`: Install dependencies with uv
- `make test`: Run test suite
- `make lint`: Check code with linters
- `make format`: Auto-format code
- `make run`: Start the application
- `make clean`: Remove cache and temporary files

## Key Files
- `src/upscaler/app.py`: FastAPI application and routes
- `src/upscaler/upscaler.py`: Core upscaling logic with Real-ESRGAN
- `src/upscaler/utils.py`: Shared utility functions (e.g., upscale_image)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertsLando/upscaler](https://github.com/robertsLando/upscaler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
