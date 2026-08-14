---
trigger: always_on
description: RinhaDasLendas is an internal League of Legends platform used to organize matches, players, drafts, statistics and future Discord/Riot integrations.
---

# AGENTS.md

## Project Overview

RinhaDasLendas is an internal League of Legends platform used to organize matches, players, drafts, statistics and future Discord/Riot integrations.

The project follows a Specification Driven Development (SDD) workflow using Spec Kit.

All implementation decisions must respect:

* Constitution
* Specifications
* Plans
* Tasks
* Architecture documentation

---

# Spec Kit Workflow

Always follow this sequence:

1. Constitution
2. Specify
3. Plan
4. Tasks
5. Implement

Do not skip phases.

Do not start implementation before tasks are approved.

---

# Git Workflow

Before starting any feature:

1. Check the current branch.

2. If the current branch is `main`, create a feature branch.

Branch naming:

```text
feature/<feature-id>-<slug>
```

Example:

```text
feature/003-cadastro-jogadores-rotas
```

3. Never implement features directly on `main`.

4. Commit after each phase:

* specify
* plan
* tasks
* implement

Examples:

```bash
git commit -m "docs: complete feature specification"
git commit -m "docs: complete implementation plan"
git commit -m "docs: generate tasks"
git commit -m "feat: implement player registration"
```

---

# Commit Messages

All commit messages must be written in Brazilian Portuguese.

Preferred patterns:

```text
docs: adicionar diretrizes de arquitetura
docs: atualizar plano da feature cadastro de jogadores

feat: implementar cadastro de jogadores
feat: adicionar atualização de preferências de rotas

fix: corrigir validação de prioridades de rota
fix: corrigir consulta de jogadores inativos

refactor: reorganizar camada de aplicação
refactor: simplificar validações de jogador

test: adicionar testes de cadastro de jogador
test: adicionar testes de preferências de rotas

chore: atualizar dependências
chore: ajustar configuração do devcontainer
```

Do not create commit messages in English.

Avoid messages such as:

```text
feat: implement player registration
fix: update route validation
docs: update architecture docs
```

Always prefer Portuguese commit messages.

---

# Repository Structure

The repository already contains:

```text
BackEnd/
FrontEnd/
.devcontainer/
docs/
specs/
```

All backend code must be created inside:

```text
BackEnd/
```

All frontend code must be created inside:

```text
FrontEnd/
```

Do not create alternative root folders such as:

```text
backend/
frontend/
api/
web/
application/
server/
client/
```

Use the existing project structure.

---

# Architecture Documentation

Always follow the documents located at:

```text
docs/architecture/
```

Current documents:

```text
docs/architecture/DESIGN_PATTERNS.md
docs/architecture/DDD_GUIDELINES.md
docs/architecture/API_STANDARDS.md
docs/architecture/DATABASE_GUIDELINES.md
```

These documents are considered the source of truth for architectural decisions.

---

# Backend Standards

Mandatory technologies:

* .NET 10
* ASP.NET Core Web API
* Entity Framework Core
* PostgreSQL
* FluentValidation
* MediatR
* Dependency Injection

Preferred architecture:

```text
Api
Application
Domain
Infrastructure
Tests
```

Business rules must not be implemented inside Controllers.

Controllers should only:

* receive requests;
* validate input;
* execute use cases;
* return responses.

---

# Development Environment

The .NET 10 SDK and backend tooling are provided by the `app` service in `.devcontainer/docker-compose.yml`. Do not assume `dotnet` is installed in the WSL host.

Before running backend commands:

1. If `dotnet --version` succeeds, the agent is already inside the devcontainer and should run commands directly.
2. Otherwise, run backend commands through the active devcontainer:

```bash
docker compose -f .devcontainer/docker-compose.yml exec -T app dotnet test /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
docker compose -f .devcontainer/docker-compose.yml exec -T app dotnet build /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
```

3. If the Linux `docker` command is unavailable, try Docker Desktop's Windows CLI with `docker.exe` before reporting a blocker. The VS Code devcontainer project uses these stable names:

```bash
docker.exe start rinhadaslendas_devcontainer-app-1
docker.exe exec rinhadaslendas_devcontainer-app-1 dotnet test /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
docker.exe exec rinhadaslendas_devcontainer-app-1 dotnet build /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
```

4. Before creating containers, inspect existing devcontainer containers with `docker.exe ps -a --filter "label=com.docker.compose.project=rinhadaslendas_devcontainer"`. Reuse/start them to avoid port conflicts and duplicate PostgreSQL volumes.
5. Run EF Core commands through the same `app` container and use paths under `/workspaces/RinhaDasLendas`.
6. Only report the backend environment as blocked after `dotnet`, `docker`, and `docker.exe` have all been checked. If none is available, request Docker Desktop WSL integration or an active devcontainer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Igorkaiqq/RinhaDasLendas](https://github.com/Igorkaiqq/RinhaDasLendas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
