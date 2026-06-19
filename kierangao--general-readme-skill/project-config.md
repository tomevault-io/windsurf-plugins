---
trigger: always_on
description: >
---


# General README Skill
Generate professional, human-feeling README files for any project.
Strictly follow real project content, reject filler text and placeholder content.

## Overview
Full workflow: **Configure → Scan → Generate → Beautify → Output**
**Zero external dependencies.** All rules, templates and assets are embedded in the skill.
No third-party CLI, runtime or network service required for core functions.

## Trigger Rules
Trigger this skill when user inputs match any of the following:
- Exact command: `/readme`
- Text commands: generate readme, write readme, create project documentation, update readme
- Chinese commands: 帮我写 README, 生成项目文档, 更新README
- Scenario: User requests to create/rewrite/refresh project README

---

## Phase 1: Configuration
Collect configuration options from developer before generation.
All options have fixed default values for quick use.

Defaults (used when user supplies no configuration):
- Tone Profile = Professional
- Badge Style = Flat
- Primary language = English
- Secondary languages = none
If the user provides no configuration, use these defaults.

### 1. Tone Profile (Default: Professional)
Select README writing style:
- **Energetic** — Direct, confident, allow emojis in features. Reference: FastAPI style.
- **Minimal** — Terse, code-first, no redundant description. Reference: Tailwind CSS style.
- **Professional** — Neutral, structured, formal layout. Reference: Kubernetes style.

### 2. Badge Style (Default: Flat)
Select shields.io badge appearance:
- **Flat** — Clean, compact, standard style
- **Flat-square** — Sharp edges, modern compact style
- **for-the-badge** — Tall, bold, highly visual style

### 3. Multi-Language Setting
- Primary language (Default: English)
- Secondary languages (Optional: Chinese, Japanese, Korean, Spanish, French etc.)
File naming follows rules in `language-guide.md`.

---

## Phase 2: Project Scan
Use built-in `Read`, `Grep`, `Glob` tools to scan local project directory.
> Rules: Only read static files. **Never execute, modify or delete any project files.**
> Ignore directories: node_modules, .git, build, dist, __pycache__, .venv, .next, coverage, logs

Detector priority and tie-break rules (deterministic order):
1. Manifest parsing: inspect recognized manifests (package.json, pyproject.toml, go.mod, Cargo.toml, etc.). If a manifest clearly declares language/framework, mark that as primary.
2. Explicit dependency declarations: prefer declared dependencies/devDependencies when identifying frameworks.
3. File-extension majority: use only if manifests are absent or ambiguous.
4. Heuristic filename patterns: use as a last resort.
If multiple sources conflict, prefer the earlier item in this list. If manifests conflict (multiple package roots), prompt the user to choose which package root to document or follow the manifest precedence rules below.

### 2.1 Language Detection
1. Parse recognized manifests: package.json, pyproject.toml, Cargo.toml, go.mod, Gemfile, pom.xml, build.gradle. If a single manifest declares the project language unambiguously, that is the `Primary Language`.
2. If no manifest is present or manifests are ambiguous, use majority file-extension count across the selected package root.
3. If multiple manifests declare different languages for different package roots, prompt the user to select which package root to document, or default to manifest precedence: `package.json > pyproject.toml > go.mod > Cargo.toml`.

### 2.2 Framework Detection
Parse dependency fields to identify used frameworks:
- package.json → dependencies / devDependencies
- requirements.txt / pyproject.toml → Python frameworks
- Gemfile → Ruby frameworks
- go.mod → Go frameworks
- Cargo.toml → Rust frameworks

### 2.3 Build & CI/CD Detection
- Makefile → Make commands
- CMakeLists.txt → CMake build
- package.json → Scripts field
- Dockerfile / docker-compose.yml → Docker deployment
- .github/workflows / .gitlab-ci.yml / Jenkinsfile → CI/CD pipelines

### 2.4 Database & ORM Detection
- Database connection config: DATABASE_URL, DB_HOST and related fields
- ORM frameworks: Sequelize, TypeORM, Prisma, SQLAlchemy, ActiveRecord, GORM
- Database drivers in dependency list

### 2.5 Architecture Type Detection
Classify overall project architecture:
- **Microservice**: Multiple independent service folders + .proto / RPC files
- **Frontend-Backend Split**: client/server or frontend/backend directories
- **Monolithic**: Standard MVC / layered structure
- **Event-Driven**: Message queue dependencies (Kafka, RabbitMQ, etc.)

### 2.6 API Style Detection
Identify interface type:
- **REST**: Route / endpoint definition files
- **gRPC**: .proto definition files
- **GraphQL**: .graphql schema files
- **WebSocket**: WebSocket server implementation code

### 2.7 License Detection
- Read root `LICENSE` / `LICENSE.md`
- Recognize common licenses: MIT, Apache-2.0, GPL-3.0, BSD-2-Clause, BSD-3-Clause, ISC, MPL-2.0, Unlicense

If no LICENSE file is present, do not add a license section; instead add a one-line recommendation: `No LICENSE file detected. Add a LICENSE to clarify project licensing.`
If multiple conflicting license files are present, list them and prompt the user to confirm which to use (for example: `Detected license files: LICENSE, LICENSE.md. Please confirm which to use.`).

### 2.8 Project Type Detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KieranGao/general-readme-skill](https://github.com/KieranGao/general-readme-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
