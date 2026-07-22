---
trigger: always_on
description: The jesse repository is the **core open-source framework** of the Jesse trading system. It contains the main Python codebase for backtesting trading strategies, importing historical data from crypto exchanges, running optimizations, and providing the API backend for the dashboard. It glues together the other repositories and makes them work together.
---

# Jesse Repository Guide for AI Agents

## Overview
The jesse repository is the **core open-source framework** of the Jesse trading system. It contains the main Python codebase for backtesting trading strategies, importing historical data from crypto exchanges, running optimizations, and providing the API backend for the dashboard. It glues together the other repositories and makes them work together.

## Skills

This repo stores shared **Agent Skills** in `.claude/skills/` — focused, task-specific playbooks used by Claude Code and Codex. Codex also discovers them through the repository-relative `.agents/skills` symlink.

Before starting a task, scan the YAML frontmatter of every `.claude/skills/*/SKILL.md`. When a skill's `description` matches the task, read that `SKILL.md` completely before acting. A matching skill is the authoritative reference for what it covers, and its details are deliberately kept out of this file. This scan is the cross-platform fallback if native skill discovery or symlink support is unavailable.

- **`jesse-strategy-tests`** — conventions for writing Jesse strategy and engine tests.

## Key Characteristics

### Central Framework
- **jesse-live depends on this** - Changes here affect live trading
- **jesse-rust integrates here** - Rust functions are called from this codebase
- **dashboard consumes this API** - Frontend uses the FastAPI routes and controller files. 

### Technology Stack
- **Python** - Primary language
- **FastAPI** - API framework for all routes
- **NumPy** - Array operations and calculations
- **keewee** - ORM for the database

## Development Workflow

### Making Changes
When implementing features or fixing bugs:

1. **Understand the scope** - Determine if other repositories such as the dashboard need updates
2. **Implement the code** in the appropriate module
3. **Write/update tests** - Maintain test coverage
4. **Run tests** to verify changes:
   ```bash
   cd-jesse && pytest
   ```
5. **Consider jesse-live** - Does this affect live trading?
6. **Update API routes** if needed - Follow FastAPI patterns
7. **Don't restart server** unless specifically asked

### Python Environment
Use the Jesse Python interpreter:
```
/Users/salehmir/miniconda3/envs/jesse3.12/bin/python
```

### Running Jesse Backend
The API server provides routes for the dashboard:
```bash
# Stop any running process
pkill -f "jesse run"

# Start Jesse from bot directory (not jesse/)
cd /Users/salehmir/codes/jesse/dev-jesse/bot
/Users/salehmir/miniconda3/envs/jesse3.12/bin/jesse run > /tmp/jesse-output.log 2>&1 &

# Server runs at http://localhost:9001

# Check logs
tail -f /tmp/jesse-output.log
```

**Important**: Don't restart Jesse after code changes unless explicitly requested.

### Running Tests
Run the test suite after changes if asked.
```bash
cd-jesse && pytest
```

If you've updated jesse-rust, run tests after building:
```bash
cd /Users/salehmir/Codes/jesse/dev-jesse/jesse-rust
./build-local.sh

cd /Users/salehmir/Codes/jesse/dev-jesse/jesse
pytest
```

## Publishing the Docker Image

When the user asks to **"push a docker build for Jesse"** (or to "release"/"publish"
Jesse), publish by pushing a version git tag. The build runs on GitHub Actions
(`.github/workflows/docker-publish.yml`) and, on a `v*` tag push, publishes to **PyPI** and
**Docker Hub in parallel**: it uploads the package to PyPI and (independently) builds the
multi-arch `linux/amd64` + `linux/arm64` Docker image, publishing `salehmir/jesse:<version>`
and `salehmir/jesse:latest`. The two are independent — if one fails the other still
publishes; just cut a new version to retry the failed half.

Steps (run from inside `jesse/`):

1. Read the current version from `setup.py` (the `VERSION = "x.y.z"` line). Do **not**
   hardcode it — always read it fresh.
2. Tell the user which version you're about to tag and push (e.g. "Pushing docker build
   for v2.2.0").
3. Confirm the tag doesn't already exist (`git tag -l v<version>` and
   `git ls-remote --tags origin v<version>`). If it already exists, stop and ask the user
   whether to bump the version in `setup.py` (and `version.py`) first.
4. Create and push the tag:
   ```bash
   cd /Users/salehmir/Codes/jesse/dev-jesse/jesse
   git tag v<version>
   git push origin v<version>
   ```
5. The `push: tags: ['v*']` trigger starts the workflow automatically. Optionally watch it:
   ```bash
   gh run watch --repo jesse-ai/jesse
   ```
6. When done, verify both architectures are present:
   ```bash
   docker buildx imagetools inspect salehmir/jesse:latest
   ```

Notes:
- Pushing a `v*` tag publishes **both PyPI and Docker** via this one workflow — PyPI first,
  Docker second. Do **not** also publish to PyPI manually (e.g. `twine upload`), or the tag
  push will fail on a duplicate-version upload.
- Required `jesse-ai/jesse` GitHub repo secrets: `PYPI_API_TOKEN` (PyPI),
  `DOCKERHUB_USERNAME` + `DOCKERHUB_TOKEN` (Docker Hub), and optionally

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jesse-ai/jesse](https://github.com/jesse-ai/jesse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
