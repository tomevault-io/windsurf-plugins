---
trigger: always_on
description: 1. **LEARN FROM ERRORS - TOP PRIORITY** - When you encounter ANY error and fix it, IMMEDIATELY update the relevant documentation:
---

# Vibe Coding System - Document Usage Algorithm
**It is year 2025**

## 🚨 CRITICAL RULES (HIGHEST PRIORITY)

1. **LEARN FROM ERRORS - TOP PRIORITY** - When you encounter ANY error and fix it, IMMEDIATELY update the relevant documentation:
   - UPDATE vibe_docs/troubleshooting.md with the error and solution
   - UPDATE vibe_docs/environment_setup.md if it's setup-related
   - UPDATE this .cursorrules file if it's a process improvement
   - DOCUMENT the exact error message, cause, and solution steps
   - This prevents repeating the same mistakes and builds institutional knowledge

2. **ALWAYS start your response with "🎯VIBING..."** - So I know this context is being used
3. **Before you run any command, ALWAYS check your current dir with "pwd"** - In case you don't know where you are
4. **FOR NEW PROJECTS: ALWAYS ASK ABOUT DEVELOPMENT STACK FIRST** - Before any coding, determine tech stack and setup environment
5. **For new tasks, you ALWAYS use WEB SEARCH** - To make less errors and get the job done quickly
6. **NEVER ASSUME** - You have no memory from previous sessions
7. **WEB SEARCH** - Always rely on web search first to understand the issue or understand the task better
8. **ALWAYS READ** - Documents in order specified below
9. **ALWAYS UPDATE** - Documentation after every interaction
10. **ALWAYS ASK** - When information is unclear or missing
11. **NEVER SKIP** - The discovery phase
12. **ENVIRONMENT SETUP IS MANDATORY** - Create vibe_docs/environment_setup.md with clear run commands

## 🎯 SESSION START PROTOCOL (PRIORITY 1)

### STEP 1: Initial Context Loading

```
EVERY SESSION START:
1. READ this file (.cursorrules) FIRST
2. READ vibe_docs/task_on_hand.md SECOND
3. IF vibe_docs/project_context.md EXISTS:
   - READ vibe_docs/project_context.md
4. IF vibe_docs/technical_details.md EXISTS:
   - READ vibe_docs/technical_details.md
5. IF vibe_docs/environment_setup.md EXISTS:
   - READ vibe_docs/environment_setup.md
6. IF vibe_docs/development_log.md EXISTS:
   - READ last 20 entries
```

### STEP 2: Understand Current State

```
IF vibe_docs/task_on_hand.md has [DISCOVERY] status:
   - ASK questions from "Open Questions" section
   - UPDATE vibe_docs/task_on_hand.md with answers
   - CREATE/UPDATE other documents as needed
ELSE IF vibe_docs/task_on_hand.md has [DEVELOPMENT] status:
   - CHECK progress checkboxes
   - CONTINUE from last completed item
   - UPDATE progress after each step
```

## 🚀 NEW PROJECT SETUP PROTOCOL (PRIORITY 2)

### STEP 0: Development Environment Discovery (BEFORE ANY CODING)

```
FOR NEW PROJECTS ONLY:

1. ASK USER about their preferred development stack:
   - Programming language (Python, Node.js, Java, etc.)
   - Framework preferences (React, Vue, Django, Flask, etc.)
   - Database preferences (PostgreSQL, MySQL, MongoDB, etc.)
   - Deployment preferences (Docker, cloud platforms, etc.)

2. WEB SEARCH for current best practices:
   - "[Language/Framework] development environment setup 2024"
   - "[Language] virtual environment best practices"
   - "[Framework] project structure recommendations"
   - "How to run [Framework] development server"

3. CREATE vibe_docs/environment_setup.md with:
   - Chosen tech stack
   - Installation commands
   - Environment setup steps
   - Clear "How to Run" instructions
   - Development vs Production configurations

4. VERIFY setup works by testing basic commands
```

### ENVIRONMENT SETUP CHECKLIST:

```
□ Language Runtime Installation
  - Python: Use pyenv/conda + virtual environments
  - Node.js: Use nvm + npm/yarn/pnpm
  - Java: Use SDKMAN + Maven/Gradle
  - Go: Use official installer + go mod
  - Rust: Use rustup + cargo

□ Package Manager Setup
  - Python: pip + requirements.txt OR poetry + pyproject.toml
  - Node.js: npm + package.json OR yarn + package.json
  - Java: Maven (pom.xml) OR Gradle (build.gradle)

□ Virtual Environment Creation
  - Python: python -m venv venv OR conda create -n project_name
  - Node.js: Project-specific node_modules (automatic)
  - Docker: Containerized environment setup

□ Development Tools Installation
  - Code editor/IDE setup
  - Linting and formatting tools
  - Debugging tools
  - Testing frameworks

□ Clear Run Commands Documentation
  - How to start development server
  - How to run tests
  - How to build for production
  - How to deploy
```

### COMMON STACK SETUPS (WEB SEARCH FIRST):

```
PYTHON STACK:
1. Install Python via pyenv or conda
2. Create virtual environment: python -m venv venv
3. Activate: source venv/bin/activate (Linux/Mac) OR venv\Scripts\activate (Windows)
4. Install dependencies: pip install -r requirements.txt
5. Run: python app.py OR flask run OR python manage.py runserver

NODE.JS STACK:
1. Install Node.js via nvm
2. Install dependencies: npm install OR yarn install
3. Run development: npm run dev OR yarn dev
4. Run production: npm start OR yarn start

DOCKER STACK:
1. Create Dockerfile and docker-compose.yml
2. Build: docker-compose build
3. Run: docker-compose up
4. Development: docker-compose up --watch (for hot reload)

ALWAYS SEARCH WEB for latest best practices before setup!
```

## 📋 DOCUMENT UPDATE PROTOCOL (PRIORITY 3)

### STEP 3: Document Update Rules

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WhoJoshi69) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
