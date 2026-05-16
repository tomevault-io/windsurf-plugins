---
trigger: always_on
description: Second Brain v5.0 - A **100% local AI-powered** knowledge management system with multimodal search, Google Drive integration, and knowledge graph visualization. NO API KEYS REQUIRED for core functionality.
---

# Second Brain Project Context

## Project Overview
Second Brain v5.0 - A **100% local AI-powered** knowledge management system with multimodal search, Google Drive integration, and knowledge graph visualization. NO API KEYS REQUIRED for core functionality.

## Development Environment

### Quick Setup
1. **Docker Required**: All development uses Docker - no Python installation needed
2. **Default Port**: 8000 for all services
3. **Database**: PostgreSQL with pgvector (managed by Docker)
4. **GPU Services**: Run on Windows host (CLIP:8002, LLaVA:8003, LM Studio:1234)

### Network Architecture (Windows + Docker)
```
Windows Host (your machine)
├── Browser → http://localhost:8000 (accesses Docker container)
├── CLIP Service (port 8002) - GPU image embeddings
├── LM Studio (port 1234) - Text generation & embeddings
└── Docker Containers
    ├── secondbrain-app (8000) → uses host.docker.internal:8002 for CLIP
    ├── secondbrain-postgres (5432) - Vector database
    └── secondbrain-adminer (8080) - Database UI
```

**Key Point**: Containers use `host.docker.internal` to reach Windows services

### Key Commands
```bash
# Start all services (port 8000)
docker-compose up -d                # Start full stack
docker-compose logs -f app          # View logs
docker-compose down                 # Stop services
docker-compose restart app          # Restart after code changes

# Run commands inside container
docker exec -it secondbrain-app python -m pytest tests/unit/ -v
docker exec -it secondbrain-app ruff check .
docker exec -it secondbrain-app black .

# Database access
docker exec -it secondbrain-postgres psql -U secondbrain
```

### Platform Optimizations

#### Windows 11 with WSL2
- **Docker Management**: Always use `wsl docker-compose` for better performance
- **File Operations**: Consider `wsl` prefix for grep, find, and other Unix tools
- **PostgreSQL**: Run `wsl psql` for database access
- **Performance**: WSL2 provides near-native Linux performance for containers

Example Windows workflow:
```bash
# Use WSL2 for Docker (faster, more reliable)
wsl docker-compose up -d
wsl docker ps
wsl docker-compose logs -f app

# But use native Windows for Python (better IDE integration)
.venv\Scripts\python.exe scripts\run_dev.py
```

#### All Platforms
- **Direct Python paths** work best (`.venv/Scripts/python.exe` on Windows, `.venv/bin/python` on Unix)
- **Port Conflicts**: Check with `netstat` (Windows) or `lsof` (Unix)

## Common Platform Pitfalls & Solutions

### Issue: "command not found" errors
- **Windows**: You're in Git Bash, use `cmd.exe /c` for Windows commands or `wsl` for Linux commands
- **Solution**: `.venv\Scripts\python.exe` or `wsl python3`

### Issue: Docker commands fail on Windows
- **Cause**: Docker Desktop runs in WSL2
- **Solution**: ALWAYS prefix with `wsl`: `wsl docker-compose up -d`

### Issue: Path separator confusion
- **Windows**: Use backslashes `\` or raw strings
- **Mac/Linux**: Use forward slashes `/`
- **Solution**: Let Claude detect platform and use correct separators

### Issue: .env file encoding problems
- **Windows**: Often saves as UTF-16 or with BOM
- **Solution**: Save as UTF-8 without BOM in VS Code/Notepad++

### Issue: Port already in use
- **Windows**: `cmd.exe /c "netstat -ano | findstr :8000"` then `taskkill /PID <pid> /F`
- **Mac/Linux**: `lsof -i :8000` then `kill -9 <pid>`

### Issue: venv activation doesn't persist
- **All platforms**: Just use direct paths: `.venv\Scripts\python.exe` (Windows) or `.venv/bin/python` (Unix)

## Platform-Specific Quick Reference

### Windows 11 (Docker Desktop Required)
```bash
# All operations through Docker - no Python installation needed
docker-compose up -d                # Start services
docker ps                           # Check status
docker-compose logs -f app          # View logs
docker exec -it secondbrain-postgres psql -U secondbrain

# Running tests and tools
docker exec -it secondbrain-app python -m pytest tests/unit/ -v
docker exec -it secondbrain-app ruff check .
docker exec -it secondbrain-app black .

# Port management (if needed)
netstat -ano | findstr :8000
taskkill /PID <pid> /F
```

### macOS / Linux
```bash
# All operations through Docker
docker-compose up -d
docker ps
docker exec -it secondbrain-app python -m pytest tests/unit/ -v
lsof -i :8000                      # macOS
ss -tlnp | grep 8000               # Linux
```

### Why WSL2 on Windows?
- **10x faster** Docker operations than Docker Desktop alone
- **Native Linux tooling** (grep, awk, sed, find)
- **Consistent behavior** with CI/CD pipelines
- **Better PostgreSQL** integration with pgvector

## Project Architecture

### Core Services
- **FastAPI Backend** (port 8000) - Main application server
- **PostgreSQL + pgvector** (port 5432) - Vector database for embeddings
- **LM Studio** (port 1234) - Local text generation (LLaVA 1.6 Mistral 7B)
- **CLIP Service** (port 8002) - Image embeddings and similarity
- **LLaVA Service** (port 8003) - Vision understanding and OCR

### Project Structure
```
/app                 - Main application code
  /routes           - API endpoints (v2 API, WebSocket, Google Drive)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raold/second-brain](https://github.com/raold/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
